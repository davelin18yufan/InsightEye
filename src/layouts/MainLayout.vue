<template>
  <q-layout view="lHh Lpr lFf" class="container">
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  name: "MainLayout",

  setup() {
    return {};
  },
};
</script>
<style scoped>
.container {
}
</style>
