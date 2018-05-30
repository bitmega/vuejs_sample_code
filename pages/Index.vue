<template>
  <div class="index-page">
    <app-header></app-header>
    <div class="container">
      <h1>Index page</h1>
    </div>
    <app-footer></app-footer>
  </div>
</template>

<script>

import Header from '@/components/Header'
import Footer from '@/components/Footer'

export default {
  name: 'Index',
  components: {
    'app-header': Header,
    'app-footer': Footer
  },
  data () {
    return {}
  }
}
</script>

<style lang="scss" scoped>
.index-page {
  margin: 0 auto;

  .container {
    h1 {
      font-weight: normal;
    }
  }
}
</style>
