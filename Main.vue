<template>
    <MainInventory :items="allItems" v-model:selected-items="selectedItems">
        <template v-slot:cart-buttons>
            <button @click="deposit(false)" class="btn-main btn-lg">DEPOSIT</button>
            <button @click="deposit(true)" class="btn-main btn-lg">CONVERT TO CREDITS</button>
        </template>

        <template v-slot:filter-buttons>
            <div>
                <span class="inventory__selectgame">Select game:</span>
                <button
                    v-for="(game, key) in gamesData"
                    :class="['btn-game', { 'btn-game-active': selectedGame === key }]"
                    @click="selectGame(key)"
                >
                    <i :class="'icon-' + key"></i> {{ game.name }}
                </button>
            </div>
            <button class="btn-main btn-sm">DEPOSIT CRYPTO</button>
        </template>
    </MainInventory>
    <DepositModal v-if="tradeId" :trade-id="tradeId" @closed="tradeId = undefined"></DepositModal>
</template>

<script lang="ts" setup>
import { onMounted, ref } from 'vue'
import MainInventory from '../components/Inventory/Main.vue'
import { useCartStore } from '../store/cart.store'
import { TItem } from '../types/api/item.type'
import { TSteamInventoryItem } from '../types/api/steam-inventory.type'
import { ApiRequest } from '../utils/api'
import DepositModal from '../components/Inventory/DepositModal.vue'

const cartStore = useCartStore()

const gamesData = {
    csgo: { appId: 730, contextId: 2, name: 'CSGO' },
    dota: { appId: 570, contextId: 2, name: 'DOTA2' },
    rust: { appId: 252490, contextId: 2, name: 'RUST' }
}
const selectedGame = ref<keyof typeof gamesData>('csgo')
const allItems = ref<TItem[]>([])
const selectedItems = ref<TItem[]>([])

onMounted(() => loadSteamInventory())

async function loadSteamInventory() {
    const gameData = gamesData[selectedGame.value]

    const {
        data: {
            response: { items, itemsValueUsd }
        }
    } = await ApiRequest.post('/user/inventory/steam', {
        appId: gameData.appId,
        contextId: gameData.contextId
    })

    allItems.value = (items as TSteamInventoryItem[]).map((x) => ({
        id: x.assetId,
        name: x.name,
        priceUsd: x.priceUsd,
        appId: x.appId,
        icon: x.icon
    }))

    cartStore.clear()
}

const tradeId = ref<string | undefined>(undefined)
async function deposit(credits: boolean = false) {
    const game = gamesData[selectedGame.value]

    const steamItems = selectedItems.value.map((x) => ({
        assetid: x!.id,
        contextid: game.contextId,
        appid: game.appId
    }))

    const {
        data: {
            response: { offerId }
        }
    } = await ApiRequest.post('/market/buy/steam', {
        steamItems,
        convertToBalance: credits
    })

    cartStore.clear()
    tradeId.value = offerId
}

function selectGame(key: typeof selectedGame.value) {
    selectedGame.value = key
    cartStore.clear()
    loadSteamInventory()
}
</script>
