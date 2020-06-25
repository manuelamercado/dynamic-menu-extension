<template>
  <section class="container left">
    <div>
      <h2 class="title">
        Dynamic Menu
      </h2>
      <p v-if="loading">Loading...</p>
      <ul class="cd-accordion-menu" v-for="item in menuItems" :key="item.name">
        <item :menu="item"></item>
      </ul>
    </div>
  </section>
</template>

<script>
import axios from "axios";
import Vue from "vue";
import VueLodash from "vue-lodash";
import lodash from "lodash";
import Item from "../components/Item.vue";

Vue.use(VueLodash, { lodash: lodash });

export default {
  name: "App",
  components: {
    Item
  },
  data () {
    return {
      loading: true,
      menuItems: {}
    }
  },
  mounted() {
  // async asyncData() {
    const groupBy = (items, key) =>
      items.reduce(
        (result, item) => ({
          ...result,
          [item[key]]: [...(result[item[key]] || []), item]
        }),
        {}
      );

    const AuthStr = `Bearer ${process.env.AIRTABLE_API_KEY}`;
    const URL = `https://api.airtable.com/v0/${process.env.AIRTABLE_DB_ID}/Config`;
    let response = axios
      .get(URL, {
        headers: { Authorization: AuthStr }
      })
      .then(response => {
        let menuItems = response
          ? response.data.records
            .filter(item => !Vue._.isEmpty(item.fields) && item.fields.Live)
            .map(item => {
              return {
                url: item.fields.URL || "",
                mainMenu: item.fields.MainMenu || "",
                actions: item.fields.Actions || "",
                name: item.fields["Sub-menu"] || ""
              };
            })
          : [];
        let menuGrouped = groupBy(menuItems, "mainMenu");
        let subMenuGrouped = [];
        for (let menu in menuGrouped) {
          subMenuGrouped.push({
            name: menu,
            children: groupBy(menuGrouped[menu], "name")
          });
        }
        let itemMenuGrouped = [];
        for (let index in subMenuGrouped) {
          let tempItem = { name: subMenuGrouped[index].name };
          let children = [];
          for (let key in subMenuGrouped[index].children) {
            let childrenTemp = {
              name: key,
              children: subMenuGrouped[index].children[key]
              .filter(item => item.url !== "")
              .map(item => {
                return {
                  name: item.actions,
                  url: item.url
                };
              })
          };
          children.push(childrenTemp);
        }
        tempItem = { ...tempItem, children: children };
        itemMenuGrouped.push(tempItem);
        }
        this.loading = false;
        this.menuItems = itemMenuGrouped;
      })
      .catch(error => console.log(error));
  }
};
</script>

<style>
*,
*::after,
*::before {
  box-sizing: border-box;
}

body {
  color: #fff;
  background: #15181d;
  background: linear-gradient(
    115deg,
    rgba(164, 189, 232, 1) 10%,
    rgba(232, 207, 164, 1) 90%
  );
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", Helvetica, Arial,
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  overflow: hidden;
}

html,
body,
.container {
  min-height: 100vh;
}

.center {
  display: flex;
  justify-content: left;
  align-items: left;
}

ol,
ul,
li {
  list-style: none;
  padding: 0px;
}

a {
  color: #15181d;
  text-decoration: none;
}

.cd-accordion-menu a:hover {
  color: yellow;
  text-decoration: underline;
  transition: color 0.375s cubic-bezier(0.4, 0, 0.2, 1);
}

h1 {
  text-align: center;
  width: 90%;
  margin: 2em auto 0;
  color: #a4bde8;
  font-weight: bold;
}

.cd-accordion-menu {
  width: 50%;
  max-width: 600px;
  margin: 1em;
  background: rgba(0, 0, 0, 0.8);
  box-shadow: 0 4px 6px 0 rgba(0, 0, 0, 0.3);
}

.cd-accordion-menu li {
  user-select: none;
}

.cd-accordion-menu li span {
  float: left;
}

.cd-accordion-menu label,
.cd-accordion-menu a {
  position: relative;
  display: block;
  padding: 10px 10px 10px 45px;
  box-shadow: inset 0 -1px #000;
  color: #ffffff;
}
.no-touch .cd-accordion-menu label:hover,
.no-touch .cd-accordion-menu a:hover {
  background: #52565d;
}
.cd-accordion-menu label::before,
.cd-accordion-menu label::after,
.cd-accordion-menu a::after {
  /* icons */
  content: "";
  display: inline-block;
  width: 16px;
  height: 16px;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
.cd-accordion-menu label {
  cursor: pointer;
}

.cd-accordion-menu label span {
  float: right;
  color: #828282;
}

.cd-accordion-menu li.Parent > a > label::before {
  content: "\f054";
}

.cd-accordion-menu label::before {
  /* arrow icon */
  font: normal normal normal 14px/1 FontAwesome;
  left: 18px;
  transform: translateY(-50%);
  transition: transform 0.3s;
}
.cd-accordion-menu label.open::before {
  transform: translateY(-25%) rotate(90deg);
}

.cd-accordion-menu ul label,
.cd-accordion-menu ul a {
  background: #000;
  box-shadow: inset 0 -1px #141617;
  padding-left: 82px;
}
.no-touch .cd-accordion-menu ul label:hover,
.no-touch .cd-accordion-menu ul a:hover {
  background: #3c3f45;
}
.cd-accordion-menu > li:last-of-type > label,
.cd-accordion-menu > li:last-of-type > a,
.cd-accordion-menu > li > ul > li:last-of-type label,
.cd-accordion-menu > li > ul > li:last-of-type a {
  box-shadow: none;
}
.cd-accordion-menu ul label::before {
  left: 36px;
}
.cd-accordion-menu ul label::after,
.cd-accordion-menu ul a::after {
  left: 59px;
}
.cd-accordion-menu ul ul label,
.cd-accordion-menu ul ul a {
  padding-left: 100px;
}
.cd-accordion-menu ul ul label::before {
  left: 54px;
}
.cd-accordion-menu ul ul label::after,
.cd-accordion-menu ul ul a::after {
  left: 77px;
}
.cd-accordion-menu ul ul ul label,
.cd-accordion-menu ul ul ul a {
  padding-left: 118px;
}
.cd-accordion-menu ul ul ul label::before {
  left: 72px;
}
.cd-accordion-menu ul ul ul label::after,
.cd-accordion-menu ul ul ul a::after {
  left: 95px;
}
</style>
