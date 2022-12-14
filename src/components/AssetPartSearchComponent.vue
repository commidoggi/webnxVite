<!-- Identical to PartSearchComponent but without quantities and query strings -->
<template>
    <div v-smooth-resize="{ delay: 50, transition: 800, fineTune: 27 }">
        <form class="flex justify-between" @submit.prevent="search">
            <!-- Search box -->
            <input class="textbox" type="text" v-model="searchText" placeholder="🔍 keywords...">
            <!-- Toggle advance search button -->
            <img class="w-10 h-10 p-2 bg-zinc-400 hover:bg-green-500 shadow-lg rounded-lg inline-block transition"
                @click="toggleAdvanced" src="../assets/sliders-solid.svg">
            <!-- Search button -->
            <input class="submit w-[calc(20%)] mt-0" type="submit" value="Search">
            <!-- Advanced search object -->
            <AdvancedSearchComponent :http="http" v-show="showAdvanced" @partSearch="advancedSearch"
                @toggle="toggleAdvanced" />
        </form>
        <!-- If there are parts -->
        <div v-if="parts.length != 0">
            <!-- Headers -->
            <div class="grid grid-cols-4 relative leading-10 text-center p-2 transition font-bold">
                <p>NXID</p>
                <p>Manufacturer</p>
                <p>Name</p>
                <p></p>
            </div>
            <!-- Asset part component for each search result -->
            <AssetPartComponent v-for="part in parts" v-bind:key="part._id"
                @addPartAction='$emit("addPartAction", part)' :part="part" />
        </div>
        <!-- If there are no results -->
        <div v-else>
            <p>No results...</p>
        </div>
        <!-- Next and previous buttons for search -->
        <div class="text-right">
            <!-- Current page number -->
            <p class="inline-block mr-3">{{ `Page: ${pageNum}` }}</p>
            <!-- Previous page button -->
            <img v-show="multiplePages || pageNum > 1"
                class="h-10 w-10 p-2 bg-zinc-400 hover:bg-green-500 shadow-lg rounded-lg inline-block transition"
                src="../assets/caret-left-solid.svg" v-on:click="prevPage">
            <!-- Next page button -->
            <img v-show="multiplePages || pageNum > 1"
                class="h-10 w-10 p-2 bg-zinc-400 hover:bg-green-500 shadow-lg rounded-lg inline-block transition"
                src="../assets/caret-right-solid.svg" v-on:click="nextPage">
        </div>
    </div>
</template>
<script setup lang="ts">
import type { AxiosError, AxiosInstance } from 'axios';
import { Ref, onBeforeMount, ref } from 'vue';
import { getPartByID, getPartsByData, getPartsByTextSearch } from '../plugins/dbCommands/partManager';
import type { PartSchema } from '../plugins/interfaces';
import AssetPartComponent from './AssetPartComponent.vue';
import AdvancedSearchComponent from './PartAdvancedSearchComponent.vue';

// Props interface
interface Props {
    http: AxiosInstance,
    errorHandler: (err: Error | AxiosError | string) => void,
    displayMessage: (message: string) => void,
}

// Define shit
let props = defineProps<Props>()
let { http, errorHandler, displayMessage } = props
const emit = defineEmits(['addPartAction'])
defineExpose({
    search
})

// component variables
let location = ''
let building = 3
let searchText = ref("")
let pageNum = ref(1)
let parts: Ref<PartSchema[]> = ref([])
let showAdvanced = ref(false);
let multiplePages = ref(false);

// Before component is mounted
onBeforeMount(async () => {
    search()
});

// Previous search page
function prevPage() {
    if (pageNum.value > 1) {
        pageNum.value -= 1
        search()
    }
}

// Next search page
function nextPage() {
    if (multiplePages) {
        pageNum.value += 1
        search()
    }
}

// Toggle advanced search
function toggleAdvanced() {
    showAdvanced.value = !showAdvanced.value
}

// Advanced search
async function advancedSearch(part: PartSchema) {
    // Query the API
    getPartsByData(http, part, building, location, (data, err) => {
        // Hide advanced search
        showAdvanced.value = false
        // Error
        if (err) {
            // Handle the error
            return errorHandler(err)
        } else if (data) {
            // Set parts list to API response
            parts.value = data as PartSchema[];
        }
    })
}

// Search function
async function search() {
    // Check for webnx regex
    if (/WNX([0-9]{7})+/.test(searchText.value)) {
        // temp value
        let query = searchText.value
        searchText.value = ""
        // Search and add to cart
        getPartByID(http, query, building, location, (data, err) => {
            if (err) {
                // Part not found
                return errorHandler(err)
            }
            // Typecast data
            let part = data as PartSchema
            if (part == null) {
                // If no part was found
                return errorHandler("Part not found.")
            }
            // Emit actions
            emit("addPartAction", part)
        })
    }
    else {
        multiplePages.value = false;
        // Text search
        getPartsByTextSearch(http, searchText.value, pageNum.value, building, location, (data: any, err) => {
            if (err) {
                // Send error to error handler
                return errorHandler(err)
            }
            // typecast 
            parts.value = data as PartSchema[];
            if (parts.value.length > 50) {
                parts.value.pop;
                multiplePages.value = true;
            } else if (parts.value.length === 0 && pageNum.value != 1) {
                pageNum.value = 1
                search()
            }
        })
    }
}
</script>
