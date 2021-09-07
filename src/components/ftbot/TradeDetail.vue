<template>
  <div class="container">
    <div class="row">
      <div class="col-lg-5">
        <h5 class="detail-header">عمومی</h5>
        <ValuePair description="شناسه ترید">{{ trade.trade_id }}</ValuePair>
        <ValuePair description="جفت">{{ trade.pair }}</ValuePair>
        <ValuePair description="تاریخ شروع">{{ timestampms(trade.open_timestamp) }}</ValuePair>
        <ValuePair v-if="trade.buy_tag" description="تگ خرید">{{ trade.buy_tag }}</ValuePair>
        <ValuePair description="نرخ باز">{{ formatPrice(trade.open_rate) }}</ValuePair>
        <ValuePair v-if="!trade.is_open && trade.close_rate" description="بستن نرخ">{{
          formatPrice(trade.close_rate)
        }}</ValuePair>
        <ValuePair v-if="trade.min_rate" description="حداقل نرخ">{{
          formatPrice(trade.min_rate)
        }}</ValuePair>
        <ValuePair v-if="trade.max_rate" description="حداکثر نرخ">{{
          formatPrice(trade.max_rate)
        }}</ValuePair>
        <ValuePair v-if="trade.close_timestamp" description="تاریخ بستن">{{
          timestampms(trade.close_timestamp)
        }}</ValuePair>
        <ValuePair
          v-if="trade.profit_ratio && trade.profit_abs"
          :description="`${trade.is_open ? 'سود جاری' : 'بستن سود'}`"
        >
          {{ formatPercent(trade.profit_ratio) }} | {{ trade.profit_abs }}
        </ValuePair>
      </div>
      <div class="col-lg-7">
        <h5 class="detail-header">ضرر</h5>
        <ValuePair description="ضرر">
          {{ formatPercent(trade.stop_loss_pct / 100) }} |
          {{ formatPrice(trade.stop_loss_abs) }}
        </ValuePair>
        <ValuePair
          v-if="trade.initial_stop_loss_pct && trade.initial_stop_loss_abs"
          description="ضرر الویه"
        >
          {{ formatPercent(trade.initial_stop_loss_pct / 100) }} |
          {{ formatPrice(trade.initial_stop_loss_abs) }}
        </ValuePair>
        <ValuePair
          v-if="trade.is_open && trade.stoploss_current_dist_ratio && trade.stoploss_current_dist"
          description="ضرر در حال حاضر"
        >
          {{ formatPercent(trade.stoploss_current_dist_ratio) }} |
          {{ formatPrice(trade.stoploss_current_dist) }}
        </ValuePair>
        <ValuePair v-if="trade.stoploss_last_update_timestamp" description="اخرین اپدیت ضرر">
          {{ timestampms(trade.stoploss_last_update_timestamp) }}
        </ValuePair>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';
import { formatPercent, formatPrice, timestampms } from '@/shared/formatters';
import ValuePair from '@/components/general/ValuePair.vue';
import { Trade } from '@/types';

@Component({
  components: { ValuePair },
})
export default class TradeDetail extends Vue {
  @Prop({ type: Object, required: true }) trade!: Trade;

  timestampms = timestampms;

  formatPercent = formatPercent;

  formatPrice = formatPrice;
}
</script>
<style scoped>
.detail-header {
  border-bottom: 1px solid;
  padding-bottom: 5px;
  width: 100%;
  display: block;
}
</style>
