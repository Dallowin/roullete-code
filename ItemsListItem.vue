<template>
    <div @click="select()" style="border-color: aquamarine" :class="['item item-lg', { 'item-selected': isSelected }]">
        <div :class="[isSelected && 'item__overlay-selected']">
            <div v-if="isSelected" class="item__selected"><i class="icon-selected"></i></div>
        </div>
        <div class="item__img item__img-lg">
            <img :src="item.icon" />
        </div>
        <div class="item__name">
            <span class="item__name-grey">{{ nameParts[0].slice(0, 30) }}</span>
            <span class="item__name-yellow">{{ nameParts[1] || 'Uncategorized' }}</span>
        </div>
        <div style="border-color: aquamarine" class="item__price">${{ item.priceUsd }}</div>
    </div>
</template>

<script lang="ts" setup>
import { computed, ref } from 'vue'
import { useCartStore } from '../../store/cart.store'
import { useDepositStore } from '../../store/deposit.store'
import { TItem } from '../../types/api/item.type'

const props = defineProps<{
    item: TItem
}>()

const cartStore = useCartStore()

const isSelected = computed(() => cartStore.selectedItemIds.has(props.item.id))

function select() {
    cartStore.setSelectedById(props.item.id, !isSelected.value)
}

const nameParts = computed(() => {
    const parts = props.item.name.split(' (')

    if (parts.length === 2) {
        return [parts[0], '(' + parts[1]]
    } else {
        return [props.item.name]
    }
})
</script>
