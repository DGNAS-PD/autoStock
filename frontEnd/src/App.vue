<template>
  <div id="app">
    <AppNav/>
    <router-view></router-view>
  </div>
</template>

<script>

import AppNav from "@/components/AppNav";

export default {
  name: 'App',
  components: {AppNav},
}
</script>
