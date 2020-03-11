<template>
  <div id="app">
    <swiper>
      <swiper-item v-for="(item,index) in myUrl" :key="index">
        <img :src="item" alt="">
      </swiper-item>
    </swiper>
  </div>
</template>

<script>

  import Swiper from "./components/swiper/Swiper";
  import SwiperItem from "./components/swiper/SwiperItem";

  export default {
    name: 'App',
    components: {SwiperItem, Swiper},
    data() {
      return {
        myUrl: ["http://img3m4.ddimg.cn/21/16/27932574-1_h_5.jpg",
          "http://img3m4.ddimg.cn/21/16/27932574-2_e_6.jpg",
          "http://img3m4.ddimg.cn/21/16/27932574-3_e_6.jpg",
        ]
      }
    }
  }
</script>

<style>

</style>
