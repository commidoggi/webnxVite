<script setup lang="ts">
import type { AssetSchema, LoadedCartItem, PartSchema } from '../plugins/interfaces';
import { ref } from 'vue';
import FullScreenPopupComponent from './FullScreenPopupComponent.vue';
import AssetManagerComponent from './AssetManagerComponent.vue';

// Props interface
let partsOnAsset = [] as LoadedCartItem[]
let oldAsset = ref({} as AssetSchema)

</script>

<template>
    <FullScreenPopupComponent>
        <AssetManagerComponent :title="'Advanced Search:'" :submitText="'Search'"
        :strict="false" :oldAsset="oldAsset" :parts="partsOnAsset"
        @assetSubmit="$emit('assetSearch', oldAsset)"/>
    </FullScreenPopupComponent>
</template>
