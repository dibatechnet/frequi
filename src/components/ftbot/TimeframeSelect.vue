<template>
  <b-form-select
    v-model="selectedTimeframe"
    placeholder="از استراتژی پیش فرض استفاده کنید"
    :options="availableTimeframes"
    @change="emitSelectedTimeframe"
  ></b-form-select>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue, Watch } from 'vue-property-decorator';

@Component({})
export default class Template extends Vue {
  selectedTimeframe = '';

  @Prop({ default: '' }) value!: string;

  // Filter available timeframes to be lower than this timeframe.
  @Prop({ default: '', required: false }) belowTimeframe!: string;

  @Emit('input')
  emitSelectedTimeframe() {
    return this.selectedTimeframe;
  }

  @Watch('value')
  watchValue() {
    this.selectedTimeframe = this.value;
  }

  get availableTimeframes() {
    if (!this.belowTimeframe) {
      return this.availableTimeframesBase;
    }
    const idx = this.availableTimeframesBase.findIndex((v) => v === this.belowTimeframe);

    return [...this.availableTimeframesBase].splice(0, idx);
  }

  // The below list must always remain sorted correctly!
  availableTimeframesBase = [
    // Placeholder value
    { value: '', text: 'از استراتژی پیش فرض استفاده کنید' },
    '۱ دقیقه',
    '۳ دقیقه',
    '۵ دقیقه',
    '۱۵ دقیقه',
    '۳۰ دقیقه',
    '۱ ساعت',
    '۲ ساعت',
    '۴ ساعت',
    '۶ ساعت',
    '۸ ساعت',
    '۱۲ ساعت',
    '۱ روز',
    '۳ روز',
    '۱ هفته',
    '۲ هفته',
    '۱ ماه',
    '۱ سال',
  ];
}
</script>

<style scoped></style>
