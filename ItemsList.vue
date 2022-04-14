<template>
    <div class="inventory__items">
        <div class="inventory__list">
            <Item v-for="item of viewInventoryItems" :item="item"></Item>
        </div>
        <div class="pagination">
            <button @click="changePagePrev()" class="btn-main btn-pg btn-icon">
                <i class="icon-cheveron-left"></i>
            </button>

            <button
                v-for="page in totalPages"
                :class="['btn-main btn-pg btn-icon', { 'btn-pg-active': currentPage === page }]"
                @click="currentPage = page"
            >
                {{ page }}
            </button>

            <button @click="changePageNext()" class="btn-main btn-pg btn-icon">
                <i class="icon-cheveron-right"></i>
            </button>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { computed, ref } from 'vue'
import { TItem } from '../../types/api/item.type'
import Item from './ItemsListItem.vue'

const props = defineProps<{ items: TItem[]; orderDesc: Boolean; searchName: string }>()

const itemsPerPage = 100
const currentPage = ref(1)
const totalPages = computed(() => Math.ceil(totalItems.value.length / itemsPerPage))

const totalItems = computed(() =>
    Array.from(props.items)
        .filter((x) => x.name.toLowerCase().includes(props.searchName.toLowerCase()))
        .sort((a, b) => (props.orderDesc ? b.priceUsd - a.priceUsd : a.priceUsd - b.priceUsd))
)

const viewInventoryItems = computed(() =>
    totalItems.value.slice((currentPage.value - 1) * itemsPerPage, currentPage.value * itemsPerPage)
)

function changePageNext() {
    if (currentPage.value === totalPages.value) {
        return true
    }
    currentPage.value++
}

function changePagePrev() {
    if (currentPage.value === 1) {
        return true
    }
    currentPage.value--
}
</script>
