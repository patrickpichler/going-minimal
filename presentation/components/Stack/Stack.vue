<script setup lang="ts">

import VClicks from '@slidev/client/builtin/VClicks.ts'
import { h } from 'vue'

const clicks = h(VClicks, {}, $slots.default)

</script>

<template>
  <div class="stack">
    <v-clicks>
      <div v-for="(child, index) in $slots.default()" :key="index">
        <component :is="child" />
      </div>
    </v-clicks>
  </div>
</template>
