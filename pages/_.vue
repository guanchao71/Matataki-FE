<template>
  <div class="no-page">
    <noPage />
  </div>
</template>

<script>
import noPage from '@/components/404/index'
export default {
  layout: 'empty',
  components: {
    noPage
  }
}
</script>

<style lang="less" scoped>
.no-page {
    background-color: #23294b;
    overflow: hidden;
    min-height: 100%;
}
</style>
