<template>
  <div class="settings-page">
    <div class="container page">
      <div class="row">

        <div class="col-md-6 offset-md-3 col-xs-12">
          <h1 class="text-xs-center">Your Settings</h1>

          <form>
            <fieldset>
                <fieldset class="form-group">
                  <input 
                    class="form-control" 
                    type="text" 
                    placeholder="URL of profile picture"
                    v-model="userInfo.image"
                  >
                </fieldset>
                <fieldset class="form-group">
                  <input 
                    class="form-control form-control-lg" type="text" 
                    placeholder="Your Name"
                    v-model="userInfo.username"
                  >
                </fieldset>
                <fieldset class="form-group">
                  <textarea 
                    class="form-control form-control-lg" rows="8" 
                    placeholder="Short bio about you"
                    v-model="userInfo.bio"
                  ></textarea>
                </fieldset>
                <fieldset class="form-group">
                  <input 
                    class="form-control form-control-lg" type="text" 
                    placeholder="Email"
                    v-model="userInfo.email"
                  >
                </fieldset>
                <fieldset class="form-group">
                  <input 
                    class="form-control form-control-lg" type="password" 
                    placeholder="Password"
                    v-model="userInfo.password"
                  >
                </fieldset>
                <button class="btn btn-lg btn-primary pull-xs-right" @click.prevent="UpdateSettings" :disabled="updateDisabled">Update Settings</button>
            </fieldset>
          </form>
          <hr>
          <button class="btn btn-outline-danger" @click.prevent="logout">
            Or click here to logout.
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const Cookie = process.client ? require('js-cookie') : undefined;
import { mapState, mapMutations } from 'vuex'
import { updateUser } from '@/api/user'

export default {
  middleware: 'authenticated',
  name: 'SettingsIndex',
  data() {
    return {
      updateDisabled: false,
      userInfo: { 
        bio: '',
        email: '',
        image: '',
        password: '',
        username: ''
      }
    }
  },
  created() {
    const { bio,email,image,password,username} = this.user;
    this.userInfo = { bio, email, image, password, username };
  },
  computed: {
    ...mapState(['user'])
  },
  methods: {
    ...mapMutations(['setUser']),
    // 更新用户信息
    async UpdateSettings(){
      this.updateDisabled = true;
      const { data } = await updateUser({
        user: this.userInfo
      });
      // 更新客户端缓存数据
      this.setUser(data.user);
      // 更新 cookie 缓存
      Cookie.set('user', data.user)
      this.$router.push(`/profile/${data.user.username}`)
      this.updateDisabled = false;
    },
    // 退出登录
    logout() {
      // 删除客户端缓存数据
      this.setUser(null);
      // 删除 cookie 缓存
      Cookie.remove('user');
      this.$router.replace('/');
    }
  }
}
</script>

<style>

</style>
