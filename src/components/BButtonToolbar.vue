<template>
  <div class="btn-toolbar" role="toolbar" :aria-label="ariaRole">
    <slot />
  </div>
</template>

<script lang="ts">
import {defineComponent} from 'vue'

export default defineComponent({
  name: 'BButtonToolbar',
  props: {
    ariaRole: {type: String, default: 'group'},
  },
})
</script>
