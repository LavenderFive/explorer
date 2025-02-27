<script lang="ts" setup>
import MdEditor from 'md-editor-v3';
import PriceMarketChart from '@/components/charts/PriceMarketChart.vue';

import { Icon } from '@iconify/vue';
import {
  useBlockchain,
  useFormatter,
  useTxDialog,
  useWalletStore,
  useStakingStore,
useParamStore,
} from '@/stores';
import { onMounted, ref } from 'vue';
import { useIndexModule } from './indexStore';
import { computed } from '@vue/reactivity';

import CardStatisticsVertical from '@/components/CardStatisticsVertical.vue';
import ProposalListItem from '@/components/ProposalListItem.vue';
import ArrayObjectElement from '@/components/dynamic/ArrayObjectElement.vue'

const props = defineProps(['chain']);

const blockchain = useBlockchain();
const store = useIndexModule();
const walletStore = useWalletStore();
const format = useFormatter();
const dialog = useTxDialog();
const stakingStore = useStakingStore();
const paramStore = useParamStore()
const coinInfo = computed(() => {
  return store.coinInfo;
});

onMounted(() => {
  store.loadDashboard();
  walletStore.loadMyAsset();
  paramStore.handleAbciInfo()
  // if(!(coinInfo.value && coinInfo.value.name)) {
  // }
});
const ticker = computed(() => store.coinInfo.tickers[store.tickerIndex]);

const currName = ref("")
blockchain.$subscribe((m, s) => {
  if (s.chainName !== currName.value) {
    currName.value = s.chainName
    store.loadDashboard();
    walletStore.loadMyAsset();
    paramStore.handleAbciInfo()
  }
});
function shortName(name: string, id: string) {
  return name.toLowerCase().startsWith('ibc/') ||
    name.toLowerCase().startsWith('0x')
    ? id
    : name;
}

const comLinks = [
  {
    name: 'Website',
    icon: 'mdi-web',
    href: store.homepage,
  },
  {
    name: 'Twitter',
    icon: 'mdi-twitter',
    href: store.twitter,
  },
  {
    name: 'Telegram',
    icon: 'mdi-telegram',
    href: store.telegram,
  },
  {
    name: 'Github',
    icon: 'mdi-github',
    href: store.github,
  },
];

// wallet box
const change = computed(() => {
  const token = walletStore.balanceOfStakingToken;
  return token ? format.priceChanges(token.denom) : 0;
});
const color = computed(() => {
  switch (true) {
    case change.value > 0:
      return 'text-green-600';
    case change.value === 0:
      return 'text-grey-500';
    case change.value < 0:
      return 'text-red-600';
  }
});
</script>

<template>
  <div>
    <div
      v-if="coinInfo && coinInfo.name"
      class="bg-base-100 rounded shadow mb-4"
    >
      <div class="grid grid-cols-2 md:grid-cols-3 p-4">
        <div class="col-span-2 md:col-span-1">
          <div class="text-xl font-semibold text-main">
            {{ coinInfo.name }} (<span class="uppercase">{{
              coinInfo.symbol
            }}</span>)
          </div>
          <div class="text-xs mt-2">
            Rank:
            <div
              class="badge text-xs badge-error bg-[#fcebea] dark:bg-[#41384d] text-red-400"
            >
              #{{ coinInfo.market_cap_rank }}
            </div>
          </div>

          <div class="my-4 flex flex-wrap items-center">
            <a
              v-for="(item, index) of comLinks"
              :key="index"
              :href="item.href"
              class="link link-primary px-2 py-1 rounded-sm no-underline hover:text-primary hover:bg-gray-100 dark:hover:bg-slate-800 flex items-center"
            >
              <Icon :icon="item?.icon" />
              <span class="ml-1 text-sm uppercase">{{ item?.name }}</span>
            </a>
          </div>

          <div>
            <div
              class="dropdown dropdown-hover w-full"
            >
              <label>
                <div
                  class="bg-gray-100 dark:bg-[#384059] flex items-center justify-between px-4 py-2 cursor-pointer rounded"
                >
                  <div>
                    <div
                      class="font-semibold text-xl text-[#666] dark:text-white"
                    >
                      {{ ticker?.market?.name || '' }}
                    </div>
                    <div class="text-info text-sm">
                      {{ shortName(ticker?.base, ticker.coin_id) }}/{{
                        shortName(ticker?.target, ticker.target_coin_id)
                      }}
                    </div>
                  </div>

                  <div class="text-right">
                    <div
                      class="text-xl font-semibold text-[#666] dark:text-white"
                    >
                      ${{ ticker.converted_last.usd }}
                    </div>
                    <div class="text-sm" :class="store.priceColor">
                      {{ store.priceChange }}%
                    </div>
                  </div>
                </div>
              </label>
              <div class="dropdown-content pt-1">
                <div
                  class="h-64 overflow-auto w-full shadow rounded"
                >
                  <ul class="menu w-full bg-gray-100 rounded dark:bg-[#384059]">
                    <li
                      v-for="(item, index) in store.coinInfo.tickers"
                      :key="index"
                      @click="store.selectTicker(index)"
                    >
                      <div
                        class="flex items-center justify-between hover:bg-base-100"
                      >
                        <div class="flex-1">
                          <div class="text-main text-sm">
                            {{ item?.market?.name }}
                          </div>
                          <div class="text-sm text-gray-500 dark:text-gray-400">
                            {{ shortName(item?.base, item.coin_id) }}/{{
                              shortName(item?.target, item.target_coin_id)
                            }}
                          </div>
                        </div>

                        <div class="text-base text-main">
                          ${{ item.converted_last.usd }}
                        </div>
                      </div>
                    </li>
                  </ul>
                </div>
              </div>
            </div>

            <a
              :color="store.trustColor"
              class="my-5 !text-white btn !bg-yes !border-yes w-full"
              :href="ticker.trade_url"
              target="_blank"
            >
              Buy {{ coinInfo.symbol || '' }}
            </a>
          </div>
        </div>

        <div class="col-span-2">
          <PriceMarketChart />
        </div>
      </div>
      <div class="h-[1px] w-full bg-gray-100 dark:bg-[#384059]"></div>
      <div class="max-h-[250px] overflow-auto p-4 text-sm">
        <MdEditor
          :model-value="coinInfo.description?.en"
          previewOnly
        ></MdEditor>
      </div>
      <div class="mx-4 flex flex-wrap items-center">
        <div
          v-for="tag in coinInfo.categories"
          class="mr-2 mb-4 text-xs bg-gray-100 dark:bg-[#384059] px-3 rounded-full py-1"
        >
          {{ tag }}
        </div>
      </div>
    </div>

    <div class="grid grid-cols-2 gap-4 md:!grid-cols-3 lg:!grid-cols-6">
      <div v-for="item in store.stats">
        <CardStatisticsVertical v-bind="item" />
      </div>
    </div>

    <div v-if="blockchain.supportModule('governance')" class="bg-base-100 rounded mt-4 shadow">
      <div class="px-4 pt-4 pb-2 text-lg font-semibold text-main">
        Active Proposals
      </div>
      <div class="px-4 pb-4">
        <ProposalListItem :proposals="store?.proposals" />
      </div>
      <div class="pb-8 text-center" v-if="store.proposals?.proposals?.length === 0">
        No active proposals
      </div>
    </div>

    <div class="bg-base-100 rounded mt-4 shadow">
      <div class="px-4 pt-4 pb-2 text-lg font-semibold text-main">
        {{ walletStore.currentAddress || 'Not Connected' }}
        <RouterLink
          v-if="walletStore.currentAddress"
          class="float-right text-sm cursor-pointert link link-primary no-underline font-medium"
          :to="`/${chain}/account/${walletStore.currentAddress}`"
          >More</RouterLink
        >
      </div>
      <div
        class="grid grid-cols-1 md:!grid-cols-4 auto-cols-auto gap-4 px-4 pb-6"
      >
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">Balance</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.balanceOfStakingToken) }}
          </div>
          <div class="text-sm" :class="color">
            <span class="ml-1">
              ${{ format.tokenValue(walletStore.balanceOfStakingToken) }}
            </span>
          </div>
        </div>
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">Staking</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.stakingAmount) }}
          </div>
          <div class="text-sm" :class="color">
            ${{ format.tokenValue(walletStore.stakingAmount) }}
          </div>
        </div>
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">Reward</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.rewardAmount) }}
          </div>
          <div class="text-sm" :class="color">
            ${{ format.tokenValue(walletStore.rewardAmount) }}
          </div>
        </div>
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">Unbonding</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.unbondingAmount) }}
          </div>
          <div class="text-sm" :class="color">
            ${{ format.tokenValue(walletStore.unbondingAmount) }}
          </div>
        </div>
      </div>

      <div v-if="walletStore.delegations.length > 0" class="px-4 pb-4">
        <table class="table table-compact w-full table-zebra">
          <thead>
            <tr>
              <th>Validator</th>
              <th>Delegations</th>
              <th>Rewards</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in walletStore.delegations" :key="index">
              <td>
                {{
                  format.validatorFromBech32(
                    item?.delegation?.validator_address
                  )
                }}
              </td>
              <td>{{ format.formatToken(item?.balance) }}</td>
              <td>
                {{
                  format.formatTokens(
                    walletStore?.rewards?.rewards?.find(
                      (el) =>
                        el?.validator_address ===
                        item?.delegation?.validator_address
                    )?.reward)
                }}
              </td>
              <td>
                <div>
                  <label for="delegate"
                    class="btn !btn-xs !btn-primary btn-ghost rounded-sm mr-2"
                    @click="dialog.open('delegate', { validator_address: item.delegation.validator_address})"
                  >
                    Delegate
                  </label>
                  <label for="withdraw"
                    class="btn !btn-xs !btn-primary btn-ghost rounded-sm"
                    @click="dialog.open('withdraw', { validator_address: item.delegation.validator_address})"
                  >
                    Withdraw Rewards
                  </label>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="grid grid-cols-3 gap-4 px-4 pb-6 mt-4">
        <label for="PingTokenConvert" class="btn btn-primary text-white">Swap</label>
        <label for="send" class="btn !bg-yes !border-yes text-white" @click="dialog.open('send', {})">Send</label>
        <RouterLink to="/wallet/receive" class="btn !bg-info !border-info text-white">Receive</RouterLink>
      </div>
      <Teleport to="body">
        <ping-token-convert
          :chain-name="blockchain?.current?.prettyName"
          :endpoint="blockchain?.endpoint?.address"
          :hd-path="walletStore?.connectedWallet?.hdPath"
        ></ping-token-convert>
      </Teleport>
    </div>

    <div class="bg-base-100 rounded mt-4 shadow">
      <div class="px-4 pt-4 pb-2 text-lg font-semibold text-main">
        Application Versions
      </div>
      <!-- Application Version -->
      <ArrayObjectElement :value="paramStore.appVersion?.items" :thead="false" />
    </div>

    <div v-if="!store.coingeckoId" class="bg-base-100 rounded mt-4 shadow">
      <div class="px-4 pt-4 pb-2 text-lg font-semibold text-main">
        Node Information
      </div>
      <ArrayObjectElement :value="paramStore.nodeVersion?.items" :thead="false" />
    </div>
  </div>
</template>

<route>
  {
    meta: {
      i18n: 'dashboard',
      order: 1,
    }
  }
</route>
