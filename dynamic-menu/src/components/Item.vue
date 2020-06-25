<template>
  <li :class="[isParentMenu ? 'Parent' : 'Child']">
    <a :href="[menu.url ? menu.url : '#']">
      <label :class="{ open: open }" @click="toggle">
        {{ menu ? (menu.name ? menu.name : menu.url) : "" }}
      </label>
    </a>
    <ul v-show="open" v-if="isParentMenu" :class="{ open: open }">
      <item
        v-for="(menuItem, index) in menu.children"
        :key="index"
        :menu="menuItem"
      >
      </item>
    </ul>
  </li>
</template>

<script>
import Vue from "vue";
import VueLodash from "vue-lodash";
import lodash from "lodash";

Vue.use(VueLodash, { lodash: lodash });

export default {
  name: "item",
  props: {
    menu: Object
  },
  data() {
    return {
      open: false
    };
  },
  computed: {
    isParentMenu: function() {
      let childrenSize =
        this.menu !== undefined ? Vue._.size(this.menu.children) : 0;
      return childrenSize > 0;
    }
  },
  methods: {
    toggle: function() {
      if (this.isParentMenu) {
        this.open = !this.open;
      }
    }
  }
};
</script>
