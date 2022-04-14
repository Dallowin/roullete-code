<template>
    <MainInventory :items="allItems" v-model:selected-items="selectedItems">
        <template v-slot:cart-buttons>
            <button @click="withdraw()" class="btn-main btn-lg">WITHDRAW</button>
        </template>
    </MainInventory>
</template>

<script lang="ts" setup>
import { notify } from '@kyvg/vue3-notification'
import { onMounted, ref } from 'vue'
import MainInventory from '../components/Inventory/Main.vue'
import { useCartStore } from '../store/cart.store'
import { TItem } from '../types/api/item.type'
import { TMarketSearch } from '../types/api/market-search.type'
import { ApiRequest } from '../utils/api'

const cartStore = useCartStore()

const allItems = ref<TItem[]>([])
const selectedItems = ref<TItem[]>([])

onMounted(() => loadMarket())

async function loadMarket() {
    const {
        data: {
            response: { items }
        }
    } = await ApiRequest.get('/user/inventory/site')

    allItems.value = (items as TMarketSearch[]).map((x) => ({
        id: x.id,
        name: x.item.name,
        priceUsd: x.item.priceUsd,
        appId: x.item.appId,
        icon: x.item.icon
    }))

    cartStore.clear()
}

function withdraw() {
    notify({
        title: 'Error',
        text: 'Users need to be at least Level 10 to get the winnings',
        type: 'error',
        duration: 10000
    })
    cartStore.clear()
}
</script>
