<template>
  <div>
    <transition>
      <div v-show="visible" class="header-wrapper">
        <div class="header" style="position:relative">
          <div class="logo">
            <router-link tag="h2" to="/">
              前端
              <sup>Blog</sup>
            </router-link>
          </div>
          <div class="nav">
            <ul class="navmenu">
              <li v-for="(item,index) in navlist" :key="index" class="tcss">
                <router-link :to="item.path" tag="span">
                  <span :class="item.icon"></span>
                  <span style="font-size:18px">{{item.title}}</span>
                </router-link>
              </li>
            </ul>
            <div class="navmenu1">
              <el-dropdown trigger="click" @command="handleCommand">
                <div class="el-dropdown-link" style="color:#53a8ff">
                  {{title}}
                  <i class="el-icon-arrow-down el-icon--right"></i>
                </div>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item
                    v-for="(item,index) in navlist"
                    :key="index"
                    :command="item.title"
                  >
                    <router-link tag="span" :to="item.path">{{item.title}}</router-link>
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </div>
          </div>
          <div class="search">
            <div>
              <el-autocomplete
                v-model="state"
                value-key="title"
                suffix-icon="el-icon-search"
                :fetch-suggestions="querySearchAsync"
                size="small"
                placeholder="搜索本站"
                @select="handleSelect"
              ></el-autocomplete>
            </div>
          </div>
          <login />
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import { findarticle } from "@/api/article";
import login from "../../components/login";
export default {
  components: {
    login
  },
  data() {
    return {
      uname: "",
      upwd: "",
      state: "",
      visible: true,
      title: "首页",
      dialogVisible: false,
      dialogtwo: false,
      navlist: [
        {
          title: "首页",
          icon: "el-icon-house",
          path: "/"
        },
        {
          title: "归档",
          icon: "el-icon-cloudy",
          path: "/history"
        },
        {
          title: "友链",
          icon: "el-icon-link",
          path: "/link"
        },
        {
          title: "留言板",
          icon: "el-icon-chat-line-round",
          path: "/message"
        },
        {
          title: "关于",
          icon: "el-icon-s-operation",
          path: "/about"
        }
      ]
    };
  },
  methods: {
    register() {
      if (this.uname == "" || this.upwd == "") {
        this.$newmessage("请输入完整！", "error");
      } else {
        findone(this.uname)
          .then(res => {
            if (res.data.length == 1) {
              this.$newmessage("该用户名已经被注册了哦~", "warning");
            } else {
              add({ username: this.uname, pwd: this.upwd })
                .then(res => {
                  this.$newmessage(this.uname + "注册成功，请登录~", "success");
                  setTimeout(() => {
                    this.dialogtwo = false;
                  }, 500);
                })
                .catch(err => {
                  console.log(err);
                });
            }
          })
          .catch(err => {
            console.log(err);
          });
      }
    },
    async login() {
      if (this.uname == "" || this.upwd == "") {
        this.$newmessage("请输入完整！", "error");
      } else {
        await this.$store.dispatch("login", {
          username: this.uname,
          pwd: this.upwd
        });
        await this.$store.dispatch("getInfo");
      }
      if (this.$store.state.user.token) {
        setTimeout(() => {
          this.dialogVisible = false;
        }, 500);
      }
    },
    logout() {
      this.$store.commit("settoken", null);
    },
    handleCommand(x) {
      this.title = x;
    },
    handleSelect(res) {
      var id = res.id;
      if (this.$route.path != "/article/" + id) {
        this.$router.push("/article/" + id);
      }
    },
    querySearchAsync(keyword, cb) {
      if (keyword) {
        findarticle(null, keyword).then(res => {
          setTimeout(() => {
            cb(res.list.rows);
          }, 300);
        });
      } else {
        setTimeout(() => {
          cb([]);
        }, 300);
      }
    },
    headershow() {
      // 头部高度为70px
      const height = 70;
      const offsetTop =
        window.pageYOffset || document.documentElement.scrollTop;
      this.visible = offsetTop < height;
    }
  },
  computed: {
    inout() {
      if (!this.$store.state.user.token) {
        return true;
      } else {
        return false;
      }
    }
  },
  mounted() {
    window.addEventListener("scroll", this.headershow);
  },
  destroyed() {
    window.removeEventListener("scroll", this.headershow);
  }
};
</script>

<style scoped>

ul {
  margin: 0;
  padding: 0;
}
.header {
  display: flex;
  height: 70px;
  line-height: 70px;
  margin: 0 auto;
  justify-content: space-between;
  transition: all 0.2s;
}
.logo {
  padding-left: 10px;
}
.login {
  padding-right: 10px;
}
.logo h2 {
  font-size: 24px;
  padding: 0px;
  margin: 0px;
  transition: all 1s;
}
.logo h2 sup {
  font-size: 12px;
  position: relative;
  top: -5px;
  transition: all 1s;
}
@media screen and (max-width: 540px) {
  .header {
    height: 50px;
    line-height: 50px;
  }
  .search {
    width: 30%;
  }
  .logo h2 {
    font-size: 18px;
  }
  .logo h2 sup {
    font-size: 10px;
  }
}
.v-enter-active {
  transition: all 0.5s ease;
}
.v-leave-active {
  transition: all 0.5s ease;
}
.v-enter,
.v-leave-to {
  transform: translateY(-75px);
}
</style>
