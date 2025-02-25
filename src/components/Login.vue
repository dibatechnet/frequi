<template>
  <div class="body">
    <form ref="form" novalidate @submit.stop.prevent="handleSubmit" @reset="handleReset">
      <b-form-group
        :state="urlState"
        label="ادرس کلید"
        label-for="url-input"
        invalid-feedback="API Url required"
      >
        <b-form-input
          id="url-input"
          v-model="auth.url"
          :state="urlState"
          required
          @keydown.enter.native="handleOk"
        ></b-form-input>
      </b-form-group>
      <b-form-group
        :state="nameState"
        label="نام کاربری"
        label-for="username-input"
        invalid-feedback="Name is required"
      >
        <b-form-input
          id="username-input"
          v-model="auth.username"
          :state="nameState"
          required
          @keydown.enter.native="handleOk"
        ></b-form-input>
      </b-form-group>
      <b-form-group label="رمزعبور" label-for="password-input" invalid-feedback="Invalid Password">
        <b-form-input
          id="password-input"
          v-model="auth.password"
          required
          type="password"
          @keydown.enter.native="handleOk"
        ></b-form-input>
      </b-form-group>
      <div>
        <b-alert v-if="errorMessage" class="alert-wrap" show variant="warning">
          {{ errorMessage }}
          <br />
          <span v-if="errorMessageCORS">
            لطفا CORS هم چک کنید:
            <a href="https://www.freqtrade.io/en/latest/rest-api/#cors">قوانین CORS</a></span
          >
        </b-alert>
      </div>
      <div v-if="inModal === false" class="float-right">
        <b-button class="mr-2" type="reset" variant="danger">شروع مجدد</b-button>
        <b-button type="submit" variant="primary">تایید</b-button>
      </div>
    </form>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Emit, Prop } from 'vue-property-decorator';
import { Action } from 'vuex-class';
import userService from '@/shared/userService';
import { setBaseUrl } from '@/shared/apiService';

import { AuthPayload } from '@/types';

const defaultURL = window.location.origin || 'http://localhost:8080';

@Component({})
export default class Login extends Vue {
  @Action setLoggedIn;

  @Prop({ default: false }) inModal!: boolean;

  $refs!: {
    form: HTMLFormElement;
  };

  auth: AuthPayload = {
    url: defaultURL,
    username: '',
    password: '',
  };

  @Emit('loginResult')
  emitLoginResult(value: boolean) {
    return value;
  }

  nameState: boolean | null = null;

  urlState: boolean | null = null;

  errorMessage = '';

  errorMessageCORS = false;

  checkFormValidity() {
    const valid = this.$refs.form.checkValidity();
    this.nameState = valid;
    return valid;
  }

  resetLogin() {
    this.auth.url = defaultURL;
    this.auth.username = '';
    this.auth.password = '';
    this.nameState = null;
    this.errorMessage = '';
  }

  handleReset(evt) {
    evt.preventDefault();
    this.resetLogin();
  }

  handleOk(evt) {
    evt.preventDefault();
    this.handleSubmit();
  }

  handleSubmit() {
    // Exit when the form isn't valid
    if (!this.checkFormValidity()) {
      return;
    }
    this.errorMessage = '';
    // Push the name to submitted names
    userService
      .login(this.auth)
      .then(() => {
        this.setLoggedIn(true);
        setBaseUrl(userService.getAPIUrl());
        this.emitLoginResult(true);
        if (this.inModal === false) {
          if (typeof this.$route.query.redirect === 'string') {
            const resolved = this.$router.resolve({ path: this.$route.query.redirect });
            if (resolved.route.name !== '404') {
              this.$router.push(resolved.route.path);
            } else {
              this.$router.push('/');
            }
          } else {
            this.$router.push('/');
          }
        }
      })
      .catch((error) => {
        this.errorMessageCORS = false;
        // this.nameState = false;
        console.error(error.response);
        if (error.response && error.response.status === 401) {
          this.nameState = false;
          this.errorMessage =
            'به ربات وصل شدید, اما ورود شکست خورد, نام کاربری و رمز عبور اشتباه است.';
        } else {
          this.urlState = false;
          this.errorMessage = `ورود ناموفق.
لطفا مطمئن شوید ربات فعال است, کلید ربات فعال باشد و ادرس قابل دسترسی باشد.
شما با کلیک بر روی  این لینک ${this.auth.url}/api/v1/ping میتوانید از وضعیت ربات مطلع شوید`;
          if (this.auth.url !== window.location.origin) {
            this.errorMessageCORS = true;
          }
        }
        console.error(this.errorMessage);
        this.emitLoginResult(false);
      });
  }
}
</script>

<style scoped lang="scss">
.body {
  font-family: 'iransans';
}
.alert-wrap {
  white-space: pre-wrap;
}
</style>
