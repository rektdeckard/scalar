<script setup lang="ts">
import { OpenApiClientButton } from '@scalar/api-client/components'
import {
  ScalarColorModeToggleButton,
  ScalarSidebarFooter,
} from '@scalar/components'
import { useBreakpoints } from '@scalar/use-hooks/useBreakpoints'
import { watch } from 'vue'

import ApiReferenceLayout from '@/components/ApiReferenceLayout.vue'
import MobileHeader from '@/components/MobileHeader.vue'
import { SearchButton } from '@/features/Search'
import { useNavState } from '@/hooks/useNavState'
import { useSidebar } from '@/hooks/useSidebar'
import type {
  DocumentSelectorSlot,
  ReferenceLayoutProps,
  ReferenceLayoutSlots,
} from '@/types'

const props = defineProps<ReferenceLayoutProps>()
defineEmits<{
  (e: 'toggleDarkMode'): void
  (e: 'updateContent', v: string): void
}>()

const slots = defineSlots<ReferenceLayoutSlots & DocumentSelectorSlot>()

const { mediaQueries } = useBreakpoints()
const { isSidebarOpen } = useSidebar()
const isDevelopment = import.meta.env.MODE === 'development'

watch(mediaQueries.lg, (newValue, oldValue) => {
  // Close the drawer when we go from desktop to mobile
  if (oldValue && !newValue) {
    isSidebarOpen.value = false
  }
})

const { hash } = useNavState()
watch(hash, (newHash, oldHash) => {
  if (newHash && newHash !== oldHash) {
    isSidebarOpen.value = false
  }
})
</script>
<template>
  <ApiReferenceLayout
    :class="{
      'scalar-api-references-standalone-mobile':
        configuration.showSidebar ?? true,
    }"
    :configuration="configuration"
    :parsedSpec="parsedSpec"
    :rawSpec="rawSpec"
    @updateContent="$emit('updateContent', $event)">
    <template
      v-for="(_, name) in slots"
      #[name]="slotProps">
      <slot
        :name="name"
        v-bind="slotProps || {}" />
    </template>
    <template #header>
      <MobileHeader
        v-if="configuration.showSidebar ?? true"
        v-model:open="isSidebarOpen" />
    </template>
    <template #sidebar-start="{ spec }">
      <!-- Wrap in a div when slot is filled -->
      <div v-if="$slots['document-selector']">
        <slot name="document-selector" />
      </div>
      <div
        v-if="!props.configuration.hideSearch"
        class="scalar-api-references-standalone-search">
        <SearchButton
          :searchHotKey="props.configuration?.searchHotKey"
          :spec="spec" />
      </div>
    </template>
    <template #sidebar-end>
      <ScalarSidebarFooter class="darklight-reference">
        <OpenApiClientButton
          v-if="!props.configuration.hideClientButton"
          buttonSource="sidebar"
          :integration="configuration._integration"
          :isDevelopment="isDevelopment"
          :url="configuration.url" />
        <!-- Override the dark mode toggle slot to hide it -->
        <template #toggle>
          <ScalarColorModeToggleButton
            v-if="!props.configuration.hideDarkModeToggle"
            :modelValue="isDark"
            @update:modelValue="$emit('toggleDarkMode')" />
          <span v-else />
        </template>
      </ScalarSidebarFooter>
    </template>
  </ApiReferenceLayout>
</template>
<style>
@media (max-width: 1000px) {
  .scalar-api-references-standalone-mobile {
    --scalar-header-height: 50px;
  }
}
</style>
<style scoped>
.scalar-api-references-standalone-search {
  display: flex;
  flex-direction: column;
  padding: 12px 12px 6px 12px;
}
.darklight-reference {
  width: 100%;
  margin-top: auto;
}
</style>
