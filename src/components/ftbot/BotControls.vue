<template>
  <div class="container-fluid">
    <div class="row">
      <button
        class="btn btn-secondary btn-sm ml-1"
        :disabled="!isTrading || isRunning"
        title="شروع ترید"
        @click="startBot()"
      >
        <PlayIcon />
      </button>
      <button
        class="btn btn-secondary btn-sm ml-1"
        :disabled="!isTrading || !isRunning"
        title="توقف ترید - همچنین معاملات باز را متوقف می کند."
        @click="handleStopBot()"
      >
        <StopIcon />
      </button>
      <button
        class="btn btn-secondary btn-sm ml-1"
        :disabled="!isTrading || !isRunning"
        title="توقف خرید - خرید را متوقف می کند ، اما همچنان معاملات باز را انجام می دهد"
        @click="handleStopBuy()"
      >
        <PauseIcon />
      </button>
      <button
        class="btn btn-secondary btn-sm ml-1"
        :disabled="!isTrading"
        title="ریست تنظیمات - ریست تنضیمات شامل استراتژی ها, ریست تمام کانفیگ های درحال اجرا میشود."
        @click="handleReloadConfig()"
      >
        <ReloadIcon />
      </button>
      <button
        v-if="botState && botState.forcebuy_enabled"
        class="btn btn-secondary btn-sm ml-1"
        :disabled="!isTrading || !isRunning"
        title="خرید اجباری - بلافاصله دارایی را با قیمت اختیاری خریداری کنید. سپس فروش طبق قوانین استراتژی انجام می شود."
        @click="initiateForcebuy"
      >
        <ForceBuyIcon />
      </button>
      <button
        v-if="isWebserverMode && false"
        :disabled="isTrading"
        class="btn btn-secondary btn-sm ml-1"
        title="فعال سازی حالت ترید"
        @click="startTrade()"
      >
        <PlayIcon />
      </button>
      <ForceBuyForm :modal-show="forcebuyShow" @close="$bvModal.hide('forcebuy-modal')" />
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import PlayIcon from 'vue-material-design-icons/Play.vue';
import StopIcon from 'vue-material-design-icons/Stop.vue';
import PauseIcon from 'vue-material-design-icons/Pause.vue';
import ReloadIcon from 'vue-material-design-icons/Reload.vue';
import ForceBuyIcon from 'vue-material-design-icons/PlusBoxMultipleOutline.vue';
import { BotState } from '@/types';
import { BotStoreGetters } from '@/store/modules/ftbot';
import ForceBuyForm from './ForceBuyForm.vue';

const ftbot = namespace('ftbot');

@Component({
  components: { ForceBuyForm, PlayIcon, StopIcon, PauseIcon, ReloadIcon, ForceBuyIcon },
})
export default class BotControls extends Vue {
  forcebuyShow = false;

  @ftbot.Getter [BotStoreGetters.botState]?: BotState;

  @ftbot.Action startBot;

  @ftbot.Action stopBot;

  @ftbot.Action stopBuy;

  @ftbot.Action reloadConfig;

  @ftbot.Action startTrade;

  @ftbot.Getter [BotStoreGetters.isTrading]!: boolean;

  @ftbot.Getter [BotStoreGetters.isWebserverMode]!: boolean;

  get isRunning(): boolean {
    return this.botState?.state === 'running';
  }

  initiateForcebuy() {
    this.$bvModal.show('forcebuy-modal');
  }

  handleStopBot() {
    this.$bvModal.msgBoxConfirm('Stop Bot?').then((value: boolean) => {
      if (value) {
        this.stopBot();
      }
    });
  }

  handleStopBuy() {
    this.$bvModal
      .msgBoxConfirm('Stop buying? Freqtrade will continue to handle open trades.')
      .then((value: boolean) => {
        if (value) {
          this.stopBuy();
        }
      });
  }

  handleReloadConfig() {
    this.$bvModal.msgBoxConfirm('Reload configuration?').then((value: boolean) => {
      if (value) {
        this.reloadConfig();
      }
    });
  }
}
</script>
