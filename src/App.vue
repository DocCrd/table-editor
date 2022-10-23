<script setup>
import TablePageComponent from './components/TablePageComponent.vue';

const url = import.meta.env.VITE_URL_PLACEHOLDER;
</script>

<template>
  <div class="important">Data Table Editor</div>
  <TablePageComponent :url="url" />
</template>
