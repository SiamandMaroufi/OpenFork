<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import Ribbon from "@/components/Ribbon.vue";
import ControlBox from "@/components/ControlBox.vue";
import { RibbonMenuItem } from "@/components/RibbonMenu";
import Editor from "@/Editor.vue";
import Viewer from "@/Viewer.vue";
import ActionGroup from "@/components/ActionGroup.vue";
import Action from "./components/Action";
import ActionButton from "@/components/ActionButton.vue";

@Component({
  components: {
    ActionButton,
    ActionGroup,
    Viewer,
    Editor,
    ControlBox,
    Ribbon
  }
})
export default class App extends Vue {
  active = "editor";
  menu: RibbonMenuItem[] = [
    { name: "editor", label: "Text Editor" },
    { name: "viewer", label: "Object Viewer" },
    { name: "about", label: "About" }
  ];

  clipboard: Action[] = [
    {
      name: "copy",
      label: "Copy"
    },
    {
      name: "paste",
      label: "Paste"
    }
  ];
  history: Action[] = [
    { name: "redo", disabled: true },
    { name: "undo", disabled: true }
  ];
  data: string = `{
    "name":"Open Fork",
    "version":0.1,
    "author": {
      "name":"Siamand",
      "web":"https://siamand.css"
    }
  }`;

  historyChanging = false;

  historyItems: string[] = [];
  historyIndex = -1;

  async onClipboard(action: string) {
    if (action === "paste") {
      this.data = await navigator.clipboard.readText();
    } else if (action === "copy") {
      navigator.clipboard.writeText(this.data);
    }
  }

  onHistory(action: string) {
    this.historyChanging = true;
    if (action === "undo" && this.historyItems[this.historyIndex - 1]) {
      const item = this.historyItems[--this.historyIndex];
      if (item) this.data = item;
    } else if (action === "redo" && this.historyItems[this.historyIndex + 1]) {
      const item = this.historyItems[++this.historyIndex];
      if (item) this.data = item;
    }
    this.historyChanging = false;
  }

  format() {
    try {
      this.data = JSON.parse(this.data);
      this.data = JSON.stringify(this.data, null, 2);
    } catch (e) {
      console.error(e);
    }
  }

  minify() {
    try {
      this.data = JSON.parse(this.data);
      this.data = JSON.stringify(this.data);
    } catch (e) {
      console.error(e);
    }
  }

  async loadFile(event: any) {
    this.clearHistory();
    this.data = await event.target.files[0].text();
  }

  selectFile() {
    const el = document.getElementById("file_input");
    el && el.click();
  }

  clearHistory() {
    this.historyItems = [];
    this.historyIndex = -1;
  }

  @Watch("data", { deep: true })
  onChange(newData: string, oldData: string) {
    if (!this.historyChanging) {
      this.historyIndex++;
      this.historyItems = [...this.historyItems, oldData];
    }
  }
}
</script>

<template>
  <div id="app">
    <ribbon :menu="menu" v-model="active" class="header-menu">
      <div slot="editor">
        <control-box label="Data" class="last">
          <action-button @click="selectFile" label="load from file">
            <i slot="prefix" class="fa fa-file" />
          </action-button>
          <action-button disabled label="load from url">
            <i slot="prefix" class="fa fa-link" />
          </action-button>
        </control-box>

        <control-box label="Clipboard" class="independent">
          <action-group
            :value="clipboard"
            class="clipboard"
            @action="onClipboard"
          >
            <i slot="copy" class="fa fa-copy" />
            <i slot="paste" class="fa fa-paste" />
          </action-group>
        </control-box>

        <control-box label="Formatting">
          <action-button label="Clear whitespace" @click="minify" />
          <action-button label="Format Text" @click="format" />
        </control-box>
        <control-box label="Content">
          <action-button label="Clear" @click="data = '{}'">
            <i slot="postfix" class="fa fa-eraser" />
          </action-button>
          <action-group :value="history" class="history" @action="onHistory">
            <span class="action" slot="undo">
              <span>Undo</span>
              <i class="fa fa-undo" />
            </span>
            <span class="action" slot="redo">
              <span>Redo</span>
              <i class="fa fa-redo" />
            </span>
          </action-group>
        </control-box>
      </div>
      <div slot="viewer"></div>
      <div slot="about" style="flex:1">
        <control-box label="OSI">
          <span style="font-size:xx-large;color:#00454d;font-weight:bold">
            <i class="fab fa-osi"></i>
          </span>
        </control-box>
        <control-box
          label="About Open Web Sheet Project"
          style="flex:1;"
          :grid="false"
        >
          <span style="flex: 1">
            Designed and developed by
            <strong>Siamand Maroufi</strong>
            <br />3rd Parties: <strong>FontAwesome Icons</strong> &
            <strong>Google noto web font</strong>
            <br />Licence:
            <strong>MIT Licence</strong>
          </span>
          <span style="align-self: flex-start">
            <a href="https://siamand.cc">
              <i class="fa fa-link"></i> Developer's site
            </a>
            |
            <a href="https://github.com/SiamandMaroufi/OpenFork">
              <i class="fab fa-github"></i> Source Code
            </a>
          </span>
        </control-box>
      </div>
    </ribbon>
    <editor v-if="active === 'editor'" v-model="data" style="flex:1" />
    <viewer v-model="data" v-else-if="active === 'viewer'" style="flex:1" />
    <div v-else>nothing here yet ...</div>
    <input
      id="file_input"
      type="file"
      @change="x => loadFile(x)"
      style="position:absolute;visibility:hidden"
    />
  </div>
</template>

<style lang="scss" scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  font-size: 12px;

  .header-menu {
    position: sticky;
    top: 0;
    z-index: 999;
  }
}

.clipboard i {
  font-size: 1.5em;
  padding: 0.5em;
}

.history {
  & .action {
    display: flex;

    i {
      margin: 0 5px;
    }
  }
}
</style>
