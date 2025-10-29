

<script>
// Note: this import needs some symlinks... see readme.
// Directions for extending a core component without importing it : https://github.com/rasteiner/oh-hi-mark/blob/main/src/index.js
// import Tabs from "@KirbyPanel/components/Layout/Tabs.vue";
// import KirbyContentHackModule from "~/mixins/kirbyContentHackModule.js";

// Fixme : localstorage item should not be only the field name, but also the page uid/path/model ?

import { usePanel } from "kirbyuse";

export default {
  name: 'k-tabs-field',
  extends: 'k-tabs',

  inheritAttrs: false,

  // Blueprint values
  props: {
    // Important: Needed for parent's value to be available in here, and to listen to it
    value: String, 
    tab: String,
  },

  //beforeCreate: function(){
    // So we can create a computed tab() to trick KTabs, without throwing warnings about replacing a prop.
    //window.panel.$vue.$delete(this.$options.props, 'tab'); // K3 only !!!
  //},

  // Useful : https://github.com/getkirby/ideas/issues/219#issuecomment-491178900
  created: function(){

    // Set value from storage
    this.$emit('input', this.getSavedTab());

    return;
  },

  mounted(){
    // Inject own css classlist into html dom
    this.$el?.classList.add('k-field');
    this.$el?.classList.add('k-tabs-field');
  },

  watch: {
    // Any time the value changes, cancel the changes object that probably got populated
    'value'(){
      
      // Note: When user discards, the tab value is nulled, hiding all tabs.
      // Never allow empty values and unexisting tab values
      if(this.value==null || !this.tabs.find((tab) => tab.name === this.value)){
        let savedTab = this.getSavedTab();
        if(savedTab) this.$emit('input', savedTab);
        return;
      }

      // Remember
      this.writeTabToLocalStorage(this.value); // Save tab for when page reloads
      //this.$set(this.$props, 'tab', this.value); // Modify the tab prop
      //this.$set(this.$props, 'value', this.value); // Modify the value prop
      this.cancelChanges(); // Ignore value change
    },
  },

  computed: {
    // Overwrite current function which lacks reactivity using native (not in sync with value)
    current() {
			const tab =
				this.tabs.find((tab) => tab.name === this.value) ?? this.tabs[0];
			return tab?.name;
		},

    // Tricks the parent to use another ref to the current tab
    // tab() {
    //   return this.current;
    // },

    // Try to override the prop ? (checkme: Has no effect !?)
    saveable(){
      return false;
    },

    // Changed : inject click method (works on hidden tabs too !)
    // (This version uses less copied code than `methods.button()` below)
    buttons() {
      let btns = this.visible.map(this.button);
      let self = this;
      return btns.map(function(btn){btn.click=function(e){self.btnClick(btn.name,e);}; return btn;});
    },
  },

  methods: {
    // Changed : inject click method (works on hidden tabs too !)
    // Works but I found a better method.
    // button(tab) {
    //   let self = this; // Added line !
    //   const button = {
    //     ...tab,
    //     current: tab.name === this.current,
    //     title: tab.label,
    //     text: tab.label ?? tab.text ?? tab.name,
    //     click(e){self.btnClick(tab.name,e);}, // Added line !
    //   };

    //   if (tab.badge) {
    //     button.badge = {
    //       theme: this.theme,
    //       text: tab.badge
    //     };
    //   } else {
    //     delete button.badge;
    //   }

    //   return button;
    // },

    // Returns the last saved tab or the default one, always valid
    getSavedTab(){
      // Grab from local storage, fall back to cur value (which can still be null)
      let valueToSanitize = this.getTabFromLocalStorage() ?? this.value;
      // Filter with fallback to 1st tab
      return ((valueToSanitize ? (this.tabs.find((tab) => tab.name === valueToSanitize)):null) ?? this.tabs[0]).name;
    },
    
    // Trick notification system by overwriting the original value with the new value
    cancelChanges(){
      if(!this.$attrs.name || this.$attrs.name=='') return; // ignore when name isn't set

      // If it breaks: See kirby/panel/src/panel/content.js @ change()
      //panel.view.props.originals[this.$attrs.name]=panel.view.props.content[this.$attrs.name]; // K5, way simpler ! :)

      const panel = usePanel();
      // K5-beta6 way ?
      if(panel.view.props.originals){
        panel.view.props.originals[[this.$attrs.name]] = panel.view.props.content[this.$attrs.name]; 
      }
      else { // k5 way
        panel.content.version("latest")[this.$attrs.name] = panel.content.version("changes")[this.$attrs.name];
      }
    },
    getTabFromLocalStorage(){
      try {
        //console.log("getTabFromLocalStorage()=", window.localStorage.getItem(`tabsfield-${this.$attrs.name}`));
        return window.localStorage.getItem(`tabsfield-${this.$attrs.name}`);
      } catch (error) {

      }
      return null;
    },
    writeTabToLocalStorage(tabName){
      if(!this.$attrs.name || this.$attrs.name=='' || tabName == undefined || tabName == null) return false; // ignore when name isn't set
      try {
        window.localStorage.setItem(`tabsfield-${this.$attrs.name}`, tabName);
        return true;
      } catch (error) {

      }
      return false;
    },
    btnClick(newTab, $event){
      // this.current = newTab;
      this.$emit('input', newTab); // Changes value, which is then triggered by our listener.
      // this.cancelChanges();// <-- useless here... $emit is delayed. So listen for the change to happen
    },
  },
}

</script>

<style lang="less">
.k-tabs-field {
  &.k-tabs {
    margin-bottom: 1px;
  }
}
</style>
