<script setup>
defineProps(["src"]);
</script>
<template>
  <img :src="src" class="image" :draggable="false" />
</template>
