<template>
  <!-- App.vue -->
  <v-app>
    <v-app-bar dark color="pink" app>
      <v-app-bar-nav-icon @click="drawer = true"></v-app-bar-nav-icon>
      <v-toolbar-title>心愿单✨</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn icon>
        <v-icon @click="add">mdi-plus</v-icon>
      </v-btn>
    </v-app-bar>
    <v-navigation-drawer v-model="drawer" absolute temporary>
      <v-list nav dense>
        <v-list-item-group
          v-model="group"
          active-class="deep-purple--text text--accent-4"
        >
          <v-list-item>
            <v-list-item-icon>
              <v-icon>mdi-home</v-icon>
            </v-list-item-icon>
            <v-list-item-title>家</v-list-item-title>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>
    <v-main>
      <v-card class="mx-auto">
        <v-list rounded>
          <v-subheader>我们的小心愿</v-subheader>
          <v-list-item-group v-model="selectedItem" color="pink">
            <v-list-item v-for="(item, i) in list" :key="i" @click="info(item)">
              <v-list-item-icon>
                <v-icon>mdi-heart-multiple</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title
                  v-if="item.status === '已实现'"
                  class="text-decoration-line-through"
                  v-text="item.name"
                >
                </v-list-item-title>
                <v-list-item-title
                  v-else
                  v-text="item.name"
                ></v-list-item-title>
              </v-list-item-content>
              <v-spacer></v-spacer>
              <!-- <v-list-item-action>
                <v-btn icon>
                  <v-icon @click="del(item)" color="grey lighten-1"
                    >mdi-delete</v-icon
                  >
                </v-btn>
              </v-list-item-action> -->
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-card>
      <!-- 详情 -->
      <v-dialog
        v-model="dialog"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      >
        <v-card>
          <v-toolbar dark color="pink">
            <v-btn icon dark @click="dialog = false">
              <v-icon>mdi-close</v-icon>
            </v-btn>
            <v-toolbar-title>{{ dialogTitle }}</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-toolbar-items>
              <v-btn dark text @click="dialogDel = true"> 删除 </v-btn>
              <v-btn dark text @click="save()"> 保存 </v-btn>
            </v-toolbar-items>
          </v-toolbar>
          <v-list three-line subheader>
            <v-subheader>我已见过银河，但我只爱一颗星</v-subheader>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title></v-list-item-title>
                <v-text-field
                  prepend-icon="mdi-message-bulleted"
                  v-model="form.name"
                  label="心愿内容"
                ></v-text-field>
                <v-menu
                  ref="menu"
                  v-model="menu"
                  :close-on-content-click="false"
                  :return-value.sync="form.createDate"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="form.createDate"
                      label="许愿日期"
                      prepend-icon="mdi-calendar-today"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker v-model="form.createDate" no-title scrollable>
                    <v-spacer></v-spacer>
                    <v-btn text color="primary" @click="menu = false">
                      取消
                    </v-btn>
                    <v-btn
                      text
                      color="primary"
                      @click="$refs.menu.save(form.createDate)"
                    >
                      确定
                    </v-btn>
                  </v-date-picker>
                </v-menu>
                <v-select
                  prepend-icon="mdi-star-crescent"
                  v-model="form.status"
                  :items="status"
                  label="状态"
                ></v-select>
                <v-menu
                  v-if="form.status === '已实现'"
                  ref="menu2"
                  v-model="menu2"
                  :close-on-content-click="false"
                  :return-value.sync="form.achieveDate"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="form.achieveDate"
                      label="实现日期"
                      prepend-icon="mdi-calendar"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker v-model="form.achieveDate" no-title scrollable>
                    <v-spacer></v-spacer>
                    <v-btn text color="primary" @click="menu2 = false">
                      取消
                    </v-btn>
                    <v-btn
                      text
                      color="primary"
                      @click="$refs.menu2.save(form.achieveDate)"
                    >
                      确定
                    </v-btn>
                  </v-date-picker>
                </v-menu>
              </v-list-item-content>
            </v-list-item>
          </v-list>
          <v-divider></v-divider>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogDel" persistent max-width="290">
        <v-card>
          <v-card-title class="text-h5"> 是否删除此愿望？ </v-card-title>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green darken-1" text @click="dialogDel = false">
              取消
            </v-btn>
            <v-btn color="green darken-1" text @click="del()"> 确定 </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-main>
    <v-footer app>
      <!-- -->
    </v-footer>
  </v-app>
</template>

<script>
import { getAPI } from "./plugins/config";
var api = getAPI() + "/deziro";
export default {
  name: "App",

  components: {
  },

  data: () => ({
    dialogDel: false,
    dialogTitle: "心愿详情",
    list: [],
    menu: false,
    menu2: false,
    status: ["已实现", "未实现"],
    form: {
      createDate: "2022-01-06",
      name: "zaiyiqi",
      status: "未实现",
      achieveDate: undefined,
    },
    dialog: false,
    notifications: false,
    sound: true,
    widgets: false,
    drawer: false,
    group: null,
    selectedItem: 1,
    items: [
      { text: "Real-Time", icon: "mdi-clock" },
      { text: "Audience", icon: "mdi-account" },
      { text: "Conversions", icon: "mdi-flag" },
    ],
  }),
  created() {
    this.init();
  },

  methods: {
    init() {
      // 计算实现日期
      if (this.form.achieveDate === undefined) {
        this.form.achieveDate = new Date(
          Date.now() - new Date().getTimezoneOffset() * 60000
        )
          .toISOString()
          .substr(0, 10);
      }
      // 列表数据
      this.axios
        .get(api)
        .then((response) => {
          this.list = response.data.data;
        })
        .catch((response) => {
          console.log(response);
        });
    },
    // 详情展示
    info(item) {
      this.dialogTitle = "心愿详情";
      this.dialog = true;
      this.form = item;
    },
    // 添加心愿唤起
    add() {
      this.dialogTitle = "添加心愿";
      this.dialog = true;
      this.form = {};
    },
    // 保存心愿
    save() {
      if (this.form.status === "未实现") {
        this.form.achieveDate = undefined;
      }
      console.log(this.form);
      this.axios
        .post(api, this.form)
        .then((response) => {
          this.dialog = false;
          this.init();
          console.log("添加成功");
        })
        .catch((response) => {
          console.log(response);
        });
    },
    // 删除
    del() {
      this.axios
        .delete(api + "/" + this.form.id)
        .then((response) => {
          this.init();
          this.dialog = false;
          this.dialogDel = false;
          console.log("删除成功");
        })
        .catch((response) => {
          console.log(response);
        });
    },
  },
};
</script>
