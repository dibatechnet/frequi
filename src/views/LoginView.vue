<template>
  <div class="container">
    <b-card header="ورود به ربات موتور سامانه انتخاب دیبا">
      <Login id="loginForm" ref="loginForm" />
    </b-card>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

import Login from '@/components/Login.vue';

@Component({
  components: { Login },
})
export default class LoginView extends Vue {}
</script>

<style scoped>
.container {
  max-width: 520px;
  font-family: 'iransans';
}
</style>
