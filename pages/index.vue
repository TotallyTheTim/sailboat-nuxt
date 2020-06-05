<template>
  <div class="main">
    <Banner />
    <h1>welcome {{ title }}</h1>
  </div>
</template>

<script>
import Banner from '@/components/Banner'
export default {
  name: 'Index',
  components: { Banner },
  data() {
    return { title: 'test' }
  }
}
</script>

<style>
</style>
