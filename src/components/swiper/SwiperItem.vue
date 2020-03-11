<template>
  <div class="slide">
    <slot></slot>
  </div>
</template>

<script>
  export default {
    name: "SwiperItem"
  }
</script>

<style scoped>
  .slide{
    width: 100%;
    /*保证每张图片撑到整个宽度，
    否则会由于flex布局会让其在一行内*/
    flex-shrink:0
  }
  .slide img{
    width: 100%;
  }
</style>