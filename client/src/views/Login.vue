<template>
  <div>
    <div class="msgs">{{errMsg}}</div>
    <div class="inputs">
      <b-field :type="usernameErr ? 'is-danger' : ''" :message="usernameErr">
        <b-input placeholder="Username" size="is-medium" icon="user" v-model="username" @keyup.native.enter="login"></b-input>
      </b-field>
      <b-field :type="passwordErr ? 'is-danger' : ''" :message="passwordErr">
        <b-input placeholder="Password" type="password" password-reveal size="is-medium" icon="key" v-model="password" @keyup.native.enter="login"></b-input>
      </b-field>
      <b-button type="is-primary" size="is-medium" expanded class="mt-20" @click="login" :loading="isLoading">Login</b-button>
      <h1 class="mt-20 alt-button" @click="signup" v-if="!hideSignup">Sign Up</h1>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import Component from 'vue-class-component';

import {Requests} from '../services/requests';
import {setToken} from '../services/user';

declare var process: any;

@Component({
  metaInfo: {
    title: 'Login'
  }
})
export default class Login extends Vue {
  public username: string = '';
  public usernameErr: string = '';

  public password: string = '';
  public passwordErr: string = '';

  public errMsg: string = '';

  public isLoading: boolean = false;

  public hideSignup = process.env.VUE_APP_PREVENT_SIGNUPS ? true : false;

  public signup() {
    this.$router.push({name: 'Sign Up'});
  }

  public async login() {
    if (this.isLoading) {
      return;
    }

    this.usernameErr = '';
    this.passwordErr = '';
    this.errMsg = '';

    if (!this.username || !this.username.length) {
      this.usernameErr = 'Username must be filled';
      return;
    }

    if (!this.password || !this.password.length) {
      this.passwordErr = 'Password must be filled.';
      return;
    }

    this.isLoading = true;

    try {
      const res = await Requests.post('/login', {username: this.username, password: this.password});
      if (res.data && res.data.access_token) {
        setToken(res.data.access_token);

        if (this.$route.query && this.$route.query.from) {
          this.$router.push({path: (this.$route.query as any).from});
        } else {
          this.$router.push({name: 'Home Redirect'});
        }
      } else {
        throw Error('Data isn\'t right');
      }
    } catch (e) {
      console.log(e);

      this.errMsg = 'There was an error logging in. Please try again.';
      this.$buefy.toast.open({
        duration: 5000,
        message: this.errMsg,
        position: 'is-top',
        type: 'is-danger'
      });
    }

    this.isLoading = false;
  }
}
</script>
