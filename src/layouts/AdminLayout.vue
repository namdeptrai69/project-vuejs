<template>
  <el-container>
    <el-header style="height: auto">
      <el-row>
        <el-col :span="12" class="logo">
          <img @click="$router.push({ path: '/home' })" src="../assets/images/LogoMakr-2rdE5H.png" alt="" style="cursor: pointer;"/>
        </el-col>
        <el-col :span="12" class="user">
          <div style="margin-left: auto">
            <el-dropdown trigger="click">
              <el-avatar
                v-if="users.avatar"
                :src="'http://vuecourse.zent.edu.vn/storage/users/'+users.avatar"
              ></el-avatar>
              <el-avatar v-else icon="el-icon-user-solid"></el-avatar>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item @click="user()" icon="el-icon-setting">
                  <el-button @click="user()" type="text">Cài đặt</el-button>
                </el-dropdown-item>
                <el-dropdown-item divided icon="el-icon-close">
                  <el-button @click="handleLogout()" type="text">Đăng xuất</el-button>
                </el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </div>
        </el-col>
      </el-row>
    </el-header>
    <el-main>
      <router-view />
    </el-main>
  </el-container>
</template>

<script>
import {mapState, mapMutations } from 'vuex'
import axios from 'axios'

export default {
  computed: {
      ...mapState('auth', ['isAuthenticated']),
      ...mapState('user', ['users']),
    },
  methods: {
    ...mapMutations('auth', ['changeLoginStatus']),
      async handleLogout() {
        localStorage.removeItem('access_token')
        localStorage.removeItem('vuex')
        this.changeLoginStatus({
          isAuthenticated: false,
          authUser: {},
        })
        if (this.$router.currentRoute.name !== 'Login') {
          await this.$router.push({ name: 'Login' })
        }
      },
    user() {
      this.$router.push({ name: "User" });
    }
  },
  mounted() {
      axios({
        method: 'get',
        url: 'http://vuecourse.zent.edu.vn/api/auth/me',
        headers: {
          Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
      }).then((response) => {
        this.changeLoginStatus({
          isAuthenticated: true,
          authUser: response.data,
        })
      }).catch((error) => {
        if (error.response.status === 401) {
          this.changeLoginStatus({
            isAuthenticated: false,
            authUser: {},
          })
          localStorage.removeItem('access_token')
          if (this.$router.currentRoute.name !== 'Login') {
            this.$router.push({ name: 'Login' })
          }
        }
      })
    }
};
</script>

<style scoped lang="scss">
.el-container {
  background-image: url("../assets/images/bg-home.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  overflow: auto;
  .el-header {
    background-color: rgba(0, 0, 0, 0.32);
    padding: 5px;
    .logo {
      img {
        width: 99px;
        filter: invert(1);
      }
    }
    .user {
      display: flex;
    }
  }
  .el-main {
    position: relative;
    padding: 0;
  }
}
</style>
