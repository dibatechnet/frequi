<template>
  <div v-if="botState">
    <p>
      ربات رمزارز موتور سامانه انتخاب دیبا در حال اجراست <strong>{{ version }}</strong>
    </p>
    <p>
      میانگین سود {{ formatPercent(profit.profit_all_ratio_mean) }} (&sum;
      {{ formatPercent(profit.profit_all_ratio_sum) }}) در {{ profit.trade_count }} ترید, با میانگین
      زمانی {{ profit.avg_duration }}. بهترین جفت: {{ profit.best_pair }}.
    </p>
    <p v-if="profit.first_trade_timestamp">
      اولین ترید باز شد در:

      <strong><DateTimeTZ :date="profit.first_trade_timestamp" show-timezone /></strong>
      <br />
      اخرین ترید باز شد در:
      <strong><DateTimeTZ :date="profit.latest_trade_timestamp" show-timezone /></strong>
    </p>

    <p>
      اجرا همراه با
      <strong>
        {{ botState.max_open_trades }}x{{ botState.stake_amount }} {{ botState.stake_currency }}
      </strong>
      on
      <strong>{{ botState.exchange }}</strong
      >, با استراتژی <strong>{{ botState.strategy }}</strong>
    </p>
    <p>
      در حال حاظر <strong>{{ botState.state }}</strong
      >, <strong>خرید اجباری: {{ botState.forcebuy_enabled }}</strong>
    </p>
    <p>
      <strong>{{ botState.dry_run ? 'اجرا تستی' : 'زنده' }}</strong>
    </p>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import { BotState, ProfitInterface } from '@/types';
import { BotStoreGetters } from '@/store/modules/ftbot';

import { formatPercent } from '@/shared/formatters';
import DateTimeTZ from '@/components/general/DateTimeTZ.vue';

const ftbot = namespace('ftbot');

@Component({ components: { DateTimeTZ } })
export default class BotStatus extends Vue {
  @ftbot.Getter [BotStoreGetters.version]: string;

  @ftbot.Getter [BotStoreGetters.profit]: ProfitInterface | {};

  @ftbot.Getter [BotStoreGetters.botState]?: BotState;

  formatPercent = formatPercent;
}
</script>
