<!-- <template>
  <main
    class="
      h-full w-full
      flex justify-start items-center flex-col
      text-slate-700
    "
  >
    <nav-bar></nav-bar>

    <main-content></main-content>

    <read-me
      class="
        p-4 m-4
        max-h-2/3 overflow-auto
        border border-slate-100 rounded
      "
    ></read-me>
  </main>
</template>

<script setup lang="ts">
import NavBar from 'src/components/nav-bar.vue';
import MainContent from 'src/components/main-content.vue';
import ReadMe from 'src/components/read-me.vue';
</script> -->

<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script lang="ts">
export default {
  name: 'App',
};
</script>

<style>
/* Add your global styles here if needed */
</style>
