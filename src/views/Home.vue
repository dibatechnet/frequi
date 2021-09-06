<template>
  <div class="home">
    <!-- <img alt="Freqtrade logo" src="../assets/freqtrade-logo.png" width="450px" class="my-5" /> -->
    <div alt="Freqtrade logo" class="logo-svg my-5 mx-auto" />
    <div>
      <h1>به سامانه مدیریت رمزارز سامانه انتخاب دیبا خوش آمدید</h1>
    </div>
    <div>این صفحه به شما دسترسی کنترل بات ترید مارا میدهد</div>
    <br />
    <!-- <p>
      If you need any help, please refer to the
      <a href="https://www.freqtrade.io/en/latest/">Freqtrade Documentation</a>.
    </p> -->

    <p>موفق باشید - <i>با ارزوی بهترین ها</i></p>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

@Component
export default class Home extends Vue {}
</script>

<style scoped>
.home {
  margin-top: 1.5em;
  font-family: 'iransans';
}
.logo-svg {
  /* -webkit-mask: url(../assets/voteVector.png) no-repeat center;
  mask: url(../assets/voteVector.png) no-repeat center; */
  /* mask-size: contain;
  width: 250px;
  height: 250px; */
}
</style>
