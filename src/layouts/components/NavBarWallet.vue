<script setup lang="ts">
import { useBaseStore, useBlockchain, useWalletStore } from '@/stores';
import { Icon } from '@iconify/vue';
import { ref, computed } from 'vue';
const walletStore = useWalletStore();
const chainStore = useBlockchain();
const baseStore = useBaseStore();
// walletStore.$subscribe((m, s) => {
//   console.log(m, s);
// });
function walletStateChange(res: any) {
  walletStore.setConnectedWallet(res.detail?.value);
}
let showCopyToast = ref(0);
async function copyAdress(address: string) {
  try {
    await navigator.clipboard.writeText(address);
    showCopyToast.value = 1;
    setTimeout(() => {
      showCopyToast.value = 0;
    }, 1000);
  } catch (err) {
    showCopyToast.value = 2;
    setTimeout(() => {
      showCopyToast.value = 0;
    }, 1000);
  }
}
const tipMsg = computed(() => {
  return showCopyToast.value === 2
    ? { class: 'error', msg: 'Copy Error!' }
    : { class: 'success', msg: 'Copy Success!' };
});
</script>

<template>
  <div class="dropdown dropdown-hover dropdown-end">
    <label
      tabindex="0"
      class="btn btn-sm m-1 lowercase hidden truncate md:!inline-flex text-xs md:!text-sm"
    >
      <Icon icon="mdi:wallet" />
      <span class="ml-1 hidden md:block">
        {{ walletStore.shortAddress || 'Wallet' }}</span
      >
    </label>
    <div
      tabindex="0"
      class="dropdown-content menu shadow p-2 bg-base-100 rounded w-64 overflow-auto"
    >
      <label
        v-if="!walletStore?.currentAddress"
        for="PingConnectWallet"
        class="btn btn-sm"
        ><Icon icon="mdi:wallet" /><span class="ml-1 hidden md:block"
          >Connect Wallet</span
        ></label
      >
      <div class="px-2 mb-1 text-gray-500 dark:text-gray-400 font-semibold">
        {{ walletStore.connectedWallet?.wallet }}
      </div>
      <div class="">
        <a
          v-if="walletStore.currentAddress"
          class="block py-2 px-2 hover:bg-gray-100 dark:hover:bg-[#353f5a] rounded cursor-pointer"
          style="overflow-wrap: anywhere"
          @click="copyAdress(walletStore.currentAddress)"
        >
          {{ walletStore.currentAddress }}
        </a>
        <div class="divider mt-1 mb-1"></div>
        <RouterLink
          class="block py-2 px-2 hover:bg-gray-100 dark:hover:bg-[#353f5a] rounded cursor-pointer"
          to="/wallet/accounts"
          >Accounts</RouterLink
        >
        <RouterLink
          class="block py-2 px-2 hover:bg-gray-100 dark:hover:bg-[#353f5a] rounded cursor-pointer"
          to="/wallet/portfolio"
          >Portfolio</RouterLink
        >
        <div v-if="walletStore.currentAddress" class="divider mt-1 mb-1"></div>
        <a
          v-if="walletStore.currentAddress"
          class="block py-2 px-2 hover:bg-gray-100 dark:hover:bg-[#353f5a] rounded cursor-pointer"
          @click="walletStore.disconnect()"
          >Disconnected</a
        >
      </div>
    </div>
    <div class="toast" v-show="showCopyToast === 1">
      <div class="alert alert-success">
        <div class="text-sm">
          <span>{{ tipMsg.msg }}</span>
        </div>
      </div>
    </div>
    <div class="toast" v-show="showCopyToast === 2">
      <div class="alert alert-error">
        <div class="text-sm">
          <span>{{ tipMsg.msg }}</span>
        </div>
      </div>
    </div>
  </div>
  <Teleport to="body">
    <ping-connect-wallet
      :chain-id="baseStore.currentChainId"
      :hd-path="chainStore.defaultHDPath"
      @connect="walletStateChange"
    />
  </Teleport>
</template>

<style>
.ping-connect-btn,
.ping-connect-dropdown {
  display: none !important;
}
</style>
