<template>
  <div class="body">
    <b-navbar toggleable="sm" type="dark" variant="primary">
      <router-link class="navbar-brand" exact to="/">
        <img class="logo" src="@/assets/voteVector.png" alt="Home Logo" />
        <span class="navbar-brand-title d-sm-none d-md-inline">ربات مدیریت رمزارز انتخاب دیبا</span>
      </router-link>

      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <router-link v-if="!canRunBacktest" class="nav-link navbar-nav navbar-dark" to="/trade"
            >ترید</router-link
          >
          <router-link
            v-if="!canRunBacktest"
            class="nav-link navbar-nav navbar-dark"
            to="/dashboard"
            >پنل مدیریتی</router-link
          >
          <router-link class="nav-link navbar-nav navbar-dark" to="/graph">نمودار</router-link>
          <router-link class="nav-link navbar-nav navbar-dark" to="/logs">لاگ ها</router-link>
          <router-link v-if="canRunBacktest" class="nav-link navbar-nav navbar-dark" to="/backtest"
            >بک تست</router-link
          >
          <BootswatchThemeSelect />
        </b-navbar-nav>
        <!-- Right aligned nav items -->
        <b-navbar-nav class="ml-auto">
          <li class="nav-item text-secondary mr-2">
            <!-- <b-nav-text class="verticalCenter small mr-2">
              {{ botName }}
            </b-nav-text> -->
            <b-nav-text class="verticalCenter">
              {{ isBotOnline ? 'آنلاین' : 'آفلاین' }}
            </b-nav-text>
          </li>
          <li v-if="loggedIn" class="nav-item">
            <b-nav-item-dropdown right>
              <b-dropdown-item>V: {{ getUiVersion }}</b-dropdown-item>
              <template #button-content>
                <b-avatar size="2em" button>FT</b-avatar>
              </template>
              <router-link class="dropdown-item" to="/settings">تنظیمات</router-link>
              <b-checkbox v-model="layoutLockedLocal" class="pl-5">قفل تم</b-checkbox>
              <b-dropdown-item @click="resetDynamicLayout">ریستارت تم</b-dropdown-item>
              <router-link class="dropdown-item" to="/" @click.native="logout()">خروج</router-link>
            </b-nav-item-dropdown>
          </li>
          <li v-else>
            <!-- should open Modal window! -->
            <LoginModal />
          </li>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator';
import { State, Action, namespace, Getter } from 'vuex-class';
import Favico from 'favico.js';
import LoginModal from '@/views/LoginModal.vue';
import userService from '@/shared/userService';
import BootswatchThemeSelect from '@/components/BootswatchThemeSelect.vue';
import { LayoutActions, LayoutGetters } from '@/store/modules/layout';
import { BotStoreGetters } from '@/store/modules/ftbot';
import { OpenTradeVizOptions, SettingsGetters } from '@/store/modules/settings';

const ftbot = namespace('ftbot');
const layoutNs = namespace('layout');
const uiSettingsNs = namespace('uiSettings');

@Component({
  components: { LoginModal, BootswatchThemeSelect },
})
export default class NavBar extends Vue {
  pingInterval: number | null = null;

  @State loggedIn!: boolean;

  @State isBotOnline!: boolean;

  @Action setLoggedIn;

  @Action loadUIVersion;

  @Getter getUiVersion!: string;

  @ftbot.Action ping;

  @ftbot.Action getState;

  @ftbot.Getter [BotStoreGetters.botName]: string;

  @ftbot.Getter [BotStoreGetters.openTradeCount]: number;

  @ftbot.Getter [BotStoreGetters.canRunBacktest]!: boolean;

  @layoutNs.Getter [LayoutGetters.getLayoutLocked]: boolean;

  @layoutNs.Action [LayoutActions.resetDashboardLayout];

  @layoutNs.Action [LayoutActions.resetTradingLayout];

  @layoutNs.Action [LayoutActions.setLayoutLocked];

  @uiSettingsNs.Getter [SettingsGetters.openTradesInTitle]: string;

  favicon: Favico | undefined = undefined;

  mounted() {
    this.ping();
    this.loadUIVersion();
    this.pingInterval = window.setInterval(this.ping, 60000);

    if (this.loggedIn) {
      // Query botstate - this will enable / disable certain modes
      this.getState();
    }
  }

  beforeDestroy() {
    if (this.pingInterval) {
      clearInterval(this.pingInterval);
    }
  }

  logout(): void {
    userService.logout();
    this.setLoggedIn(false);
  }

  get layoutLockedLocal() {
    return this.getLayoutLocked;
  }

  set layoutLockedLocal(value: boolean) {
    this.setLayoutLocked(value);
  }

  setOpenTradesAsPill(tradeCount: number) {
    console.log('setPill', tradeCount);
    if (!this.favicon) {
      this.favicon = new Favico({
        animation: 'none',
        // position: 'up',
        // fontStyle: 'normal',
        // bgColor: '#',
        // textColor: '#FFFFFF',
      });
    }
    if (tradeCount !== 0 && this.openTradesInTitle === 'showPill') {
      this.favicon.badge(tradeCount);
    } else {
      this.favicon.reset();
      console.log('reset');
    }
  }

  resetDynamicLayout(): void {
    const route = this.$router.currentRoute.path;
    console.log(`resetLayout called for ${route}`);
    switch (route) {
      case '/trade':
        this.resetTradingLayout();
        break;
      case '/dashboard':
        this.resetDashboardLayout();
        break;
      default:
    }
  }

  setTitle() {
    let title = 'freqUI';
    if (this.openTradesInTitle === OpenTradeVizOptions.asTitle) {
      title = `(${this.openTradeCount}) ${title}`;
    }
    if (this.botName) {
      title = `${title} - ${this.botName}`;
    }
    document.title = title;
  }

  @Watch(BotStoreGetters.botName)
  botnameChanged() {
    this.setTitle();
  }

  @Watch(BotStoreGetters.openTradeCount)
  openTradeCountChanged() {
    console.log('openTradeCount changed');
    if (this.openTradesInTitle === OpenTradeVizOptions.showPill) {
      this.setOpenTradesAsPill(this.openTradeCount);
    } else if (this.openTradesInTitle === OpenTradeVizOptions.asTitle) {
      this.setTitle();
    }
  }

  @Watch(SettingsGetters.openTradesInTitle)
  openTradesSettingChanged() {
    this.setTitle();
    this.setOpenTradesAsPill(this.openTradeCount);
  }
}
</script>

<style lang="scss" scoped>
.body {
  font-family: 'iransans';
}
.logo {
  vertical-align: middle;
  height: 30px;
}

.dropdown-toggle::after {
  display: none;
}

.navbar-brand-title {
  padding-left: 0.5em;
}
.navbar {
  padding: 0.2rem 1rem;
}

.router-link-active,
.nav-link:active {
  color: white !important;
}

.verticalCenter {
  align-items: center;
  display: inline-flex;
  height: 100%;
}
</style>
