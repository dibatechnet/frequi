<template>
  <div class="container-fluid">
    <div class="row">
      <button class="m-1 btn btn-primary" @click="refreshAll(true)"><RefreshIcon /></button>

      <b-form-checkbox
        v-model="autoRefreshLoc"
        class="ml-auto float-right mr-2 my-auto"
        title="AutoRefresh"
        switch
        >ریلود اتوماتیک</b-form-checkbox
      >
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator';
import { Action, State } from 'vuex-class';
import RefreshIcon from 'vue-material-design-icons/Refresh.vue';

@Component({ components: { RefreshIcon } })
export default class ReloadControl extends Vue {
  refreshInterval: number | null = null;

  refreshIntervalSlow: number | null = null;

  created() {
    if (this.loggedIn) {
      this.refreshOnce();
      this.refreshAll(true);
    }
  }

  mounted() {
    this.startRefresh(false);
  }

  beforeDestroy() {
    this.stopRefresh();
  }

  @State loggedIn;

  @State autoRefresh!: boolean;

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  @Action setAutoRefresh!: (newValue: boolean) => void;

  @Action refreshSlow;

  @Action refreshFrequent;

  @Action refreshAll;

  @Action refreshOnce;

  get autoRefreshLoc() {
    return this.autoRefresh;
  }

  set autoRefreshLoc(newValue: boolean) {
    this.setAutoRefresh(newValue);
  }

  startRefresh(runNow: boolean) {
    if (this.loggedIn !== true) {
      console.log('وارد نشده اید.');
      return;
    }
    console.log('شروع خودکار رفرش.');
    if (runNow) {
      this.refreshFrequent(false);
    }
    if (this.autoRefresh) {
      this.refreshInterval = window.setInterval(() => {
        this.refreshFrequent();
      }, 5000);
    }
    if (runNow) {
      this.refreshSlow(true);
    }
    if (this.autoRefresh) {
      this.refreshIntervalSlow = window.setInterval(() => {
        this.refreshSlow(false);
      }, 60000);
    }
  }

  stopRefresh() {
    console.log('توقف اتوماتیک رفرش.');
    if (this.refreshInterval) {
      window.clearInterval(this.refreshInterval);
    }
    if (this.refreshIntervalSlow) {
      window.clearInterval(this.refreshIntervalSlow);
    }
  }

  @Watch('رفرش اتوماتیک')
  watchAutoRefresh(val) {
    if (val) {
      this.startRefresh(true);
    } else {
      this.stopRefresh();
    }
  }
}
</script>

<style scoped></style>
