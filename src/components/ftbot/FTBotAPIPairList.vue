<template>
  <!-- TODO We could move the list into a component since we are reusing the same code for both lists. -->
  <div>
    <div>
      <h3>روش های لیست مجاز</h3>

      <div v-if="pairlistMethods.length" class="list">
        <b-list-group v-for="(method, key) in pairlistMethods" :key="key">
          <b-list-group-item href="#" class="pair white">{{ method }}</b-list-group-item>
        </b-list-group>
      </div>
    </div>
    <!-- Show Whitelist -->
    <h3>لیست مجاز</h3>
    <div v-if="whitelist.length" class="list">
      <b-list-group v-for="(pair, key) in whitelist" :key="key">
        <b-list-group-item href="#" class="pair white">{{ pair }}</b-list-group-item>
      </b-list-group>
    </div>
    <p v-else>لیست در دسترس نیست لطفا وارد شوید و مطمئن شوید که سرور در حال اجرا است.</p>
    <hr />

    <!-- Blacklsit -->
    <div>
      <label class="mr-auto h3">لیست سیاه</label>
      <b-button id="blacklist-add-btn" class="float-right" size="sm">+</b-button>
      <b-popover target="blacklist-add-btn" triggers="click" :show.sync="blackListShow">
        <form ref="form" @submit.prevent>
          <div>
            <b-form-group label-cols="2" label="جفت" label-for="pair-input">
              <b-form-input
                id="pair-input"
                v-model="newblacklistpair"
                required
                autofocus
              ></b-form-input>
            </b-form-group>
            <b-button
              id="blacklist-submit"
              class="float-right mb-2"
              size="sm"
              type="submit"
              @click="addBlacklistPair"
              >افزودن</b-button
            >
          </div>
        </form>
      </b-popover>
    </div>
    <div v-if="blacklist.length" class="list">
      <b-list-group v-for="(pair, key) in blacklist" :key="key">
        <b-list-group-item href="#" class="pair black">{{ pair }}</b-list-group-item>
      </b-list-group>
    </div>
    <p v-else>لیست سیاه در دسترس نیست. لطفا وارد شوید و مطمئن شوید که سرور در حال اجرا است.</p>
    <!-- Pagination -->
    <!-- TODO Add pagination support -->
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import { BlacklistPayload, BlacklistResponse } from '@/types';
import { BotStoreGetters } from '@/store/modules/ftbot';

const ftbot = namespace('ftbot');

@Component({})
export default class FTBotAPIPairList extends Vue {
  newblacklistpair = '';

  blackListShow = false;

  @ftbot.Action getWhitelist;

  @ftbot.Action getBlacklist;

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  @ftbot.Action addBlacklist!: (payload: BlacklistPayload) => Promise<BlacklistResponse>;

  @ftbot.Getter [BotStoreGetters.whitelist]!: string[];

  @ftbot.Getter [BotStoreGetters.blacklist]!: string[];

  @ftbot.Getter [BotStoreGetters.pairlistMethods]!: string[];

  created() {
    this.initBlacklist();
  }

  initBlacklist() {
    if (this.whitelist.length === 0) {
      this.getWhitelist();
    }
    if (this.blacklist.length === 0) {
      this.getBlacklist();
    }
  }

  addBlacklistPair() {
    if (this.newblacklistpair) {
      this.blackListShow = false;

      this.addBlacklist({ blacklist: [this.newblacklistpair] });
      this.newblacklistpair = '';
    }
  }
}
</script>

<style scoped>
.list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
  grid-gap: 0.5rem;
  padding-bottom: 1rem;
}
.pair {
  border: 1px solid #ccc;
  background: #41b883;
  padding: 0.5rem;
  border-radius: 5px;
  text-align: center;
  position: relative;
  cursor: pointer;
}
.white {
  background: white;
  color: black;
}

.black {
  background: black;
  color: white;
}
</style>
