<template>
  <div class="h-100 overflow-auto w-100">
    <b-table
      ref="tradesTable"
      small
      hover
      stacked="md"
      :items="trades"
      :fields="tableFields"
      show-empty
      :empty-text="emptyText"
      :per-page="perPage"
      :current-page="currentPage"
      primary-key="trade_id"
      selectable
      select-mode="single"
      :filter="filterText"
      @row-contextmenu="handleContextMenuEvent"
      @row-clicked="onRowClicked"
      @row-selected="onRowSelected"
    >
      <template #cell(actions)="row">
        <b-button class="btn-xs" size="sm" title="Forcesell" @click="forcesellHandler(row.item)">
          <ForceSellIcon :size="16" title="Forcesell" />
        </b-button>
        <b-button
          class="btn-xs ml-1"
          size="sm"
          title="حذف ترید"
          @click="removeTradeHandler(row.item)"
        >
          <DeleteIcon :size="16" title="Delete trade" />
        </b-button>
      </template>
      <template #cell(pair)="row">
        <ProfitSymbol :trade="row.item" />
        <span>
          {{
            `${row.item.pair}${
              row.item.open_order_id === undefined || row.item.open_order_id === null ? '' : '*'
            }`
          }}
        </span>
      </template>
      <template #cell(profit)="row">
        {{ formatPercent(row.item.profit_ratio, 2) }}
        <small :title="row.item.stake_currency || stakeCurrency">
          {{ `(${formatPriceWithDecimals(row.item.profit_abs)})` }}
        </small>
      </template>
      <template #cell(open_timestamp)="row">
        <DateTimeTZ :date="row.item.open_timestamp" />
      </template>
      <template #cell(close_timestamp)="row">
        <DateTimeTZ :date="row.item.close_timestamp" />
      </template>
    </b-table>
    <div class="w-100 d-flex justify-content-between">
      <b-pagination
        v-if="!activeTrades"
        v-model="currentPage"
        :total-rows="rows"
        :per-page="perPage"
        aria-controls="my-table"
      ></b-pagination>
      <b-input
        v-if="showFilter"
        v-model="filterText"
        type="text"
        placeholder="فیلتر"
        size="sm"
        style="width: unset"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator';
import { namespace } from 'vuex-class';

// eslint-disable-next-line @typescript-eslint/no-unused-vars
import DeleteIcon from 'vue-material-design-icons/Delete.vue';
import ForceSellIcon from 'vue-material-design-icons/CloseBoxMultiple.vue';
import { formatPercent, formatPrice } from '@/shared/formatters';
import { Trade } from '@/types';
import DateTimeTZ from '@/components/general/DateTimeTZ.vue';
import { BotStoreGetters } from '@/store/modules/ftbot';
import ProfitSymbol from './ProfitSymbol.vue';

const ftbot = namespace('ftbot');

@Component({
  components: { ProfitSymbol, DeleteIcon, ForceSellIcon, DateTimeTZ },
})
export default class TradeList extends Vue {
  $refs!: {
    tradesTable: HTMLFormElement;
  };

  formatPercent = formatPercent;

  formatPrice = formatPrice;

  @Prop({ required: true }) trades!: Array<Trade>;

  @Prop({ default: 'Trades' }) title!: string;

  @Prop({ required: false, default: '' }) stakeCurrency!: string;

  @Prop({ default: false }) activeTrades!: boolean;

  @Prop({ default: false }) showFilter!: boolean;

  @Prop({ default: 'No Trades to show.' }) emptyText!: string;

  @ftbot.Getter [BotStoreGetters.detailTradeId]?: number;

  @ftbot.Getter [BotStoreGetters.stakeCurrencyDecimals]!: number;

  @ftbot.Action setDetailTrade;

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  @ftbot.Action forcesell!: (tradeid: string) => Promise<string>;

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  @ftbot.Action deleteTrade!: (tradeid: string) => Promise<string>;

  currentPage = 1;

  selectedItemIndex? = undefined;

  filterText = '';

  @Watch('detailTradeId')
  watchTradeDetail(val) {
    const index = this.trades.findIndex((v) => v.trade_id === val);
    // Unselect when another tradeTable is selected!
    if (index < 0) {
      this.$refs.tradesTable.clearSelected();
    }
  }

  get rows(): number {
    return this.trades.length;
  }

  perPage = this.activeTrades ? 200 : 15;

  // Added to table-fields for current trades
  openFields: Record<string, string | Function>[] = [{ key: 'actions' }];

  // Added to table-fields for historic trades
  closedFields: Record<string, string | Function>[] = [
    { key: 'close_timestamp', label: 'تاریخ بستن' },
    { key: 'sell_reason', label: 'دلیل بستن' },
  ];

  tableFields: Record<string, string | Function>[] = [
    { key: 'trade_id', label: 'شناسه' },
    { key: 'pair', label: 'جفت' },
    { key: 'amount', label: 'مقدار' },
    {
      key: 'stake_amount',
      label: 'مقدار سهام',
      formatter: (value: number) => this.formatPriceWithDecimals(value),
    },
    {
      key: 'open_rate',
      label: 'نرخ باز',
      formatter: (value: number) => this.formatPrice(value),
    },
    {
      key: this.activeTrades ? 'current_rate' : 'close_rate',
      label: this.activeTrades ? 'نرخ فعلی' : 'نرخ بستن',
      formatter: (value: number) => this.formatPrice(value),
    },
    {
      key: 'profit',
      label: this.activeTrades ? 'سود جاری %' : 'سود %',
      formatter: (value: number, key, item: Trade) => {
        const percent = formatPercent(item.profit_ratio, 2);
        return `${percent} ${`(${this.formatPriceWithDecimals(item.profit_abs)})`}`;
      },
    },
    { key: 'open_timestamp', label: 'تاریخ باز کردن' },
    ...(this.activeTrades ? this.openFields : this.closedFields),
  ];

  formatPriceWithDecimals(price) {
    return formatPrice(price, this.stakeCurrencyDecimals);
  }

  forcesellHandler(item: Trade) {
    this.$bvModal
      .msgBoxConfirm(`Really forcesell ترید ${item.trade_id} (جفت ${item.pair})?`)
      .then((value: boolean) => {
        if (value) {
          this.forcesell(String(item.trade_id))
            .then((xxx) => console.log(xxx))
            .catch((error) => console.log(error.response));
        }
      });
  }

  handleContextMenuEvent(item, index, event) {
    // stop browser context menu from appearing
    if (!this.activeTrades) {
      return;
    }
    event.preventDefault();
    // log the selected item to the console
    console.log(item);
  }

  removeTradeHandler(item) {
    console.log(item);
    this.$bvModal
      .msgBoxConfirm(`مطمئن از حذف هستید  ${item.trade_id} (جفت ${item.pair})?`)
      .then((value: boolean) => {
        if (value) {
          this.deleteTrade(item.trade_id).catch((error) => console.log(error.response));
        }
      });
  }

  onRowClicked(item, index) {
    // Only allow single selection mode!
    if (
      item &&
      item.trade_id !== this.detailTradeId &&
      !this.$refs.tradesTable.isRowSelected(index)
    ) {
      this.setDetailTrade(item);
    } else {
      console.log('unsetting item');
      this.setDetailTrade(null);
    }
  }

  onRowSelected() {
    if (this.detailTradeId) {
      const itemIndex = this.trades.findIndex((v) => v.trade_id === this.detailTradeId);
      if (itemIndex >= 0) {
        this.$refs.tradesTable.selectRow(itemIndex);
      } else {
        console.log(`Unsetting item for tradeid ${this.selectedItemIndex}`);
        this.selectedItemIndex = undefined;
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.card-body {
  padding: 0 0.2em;
}
.table-sm {
  font-size: $fontsize-small;
}
.btn-xs {
  padding: 0.1rem 0.25rem;
  font-size: 0.75rem;
}
</style>
