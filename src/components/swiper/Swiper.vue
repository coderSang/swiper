<template>
  <div id="my-swiper">
    <div @touchstart="touchStart" @touchmove="touchMove"
         @touchend="touchEnd" ref="swiperBody" class="swiper-body">
      <slot></slot>
    </div>
    <div class="indicator" v-if="showIndicator && slideCount>1">
      <div v-for="(item, index) in slideCount"
           class="indicator-item"
           :class="{active: index === currentIndex-1}"
           :key="index">
      </div>
    </div>
  </div>

</template>

<script>
  export default {
    name: "Swiper",
    props: {
      //显示图片之后每张停留间隔时间
      interval: {
        type: Number,
        default: 3000
      },
      //滚动时间
      animDuration: {
        type: Number,
        default: 300
      },
      //拖动多少占比的图片之后可以划到下一张
      moveRatio: {
        type: Number,
        default: 0.25
      },
      //是否显示小圆点
      showIndicator: {
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        //是否在滚动，滚动期间不允许拖动
        isScrolling: false,
        //元素个数
        slideCount: 0,
        //元素的宽度
        totalWidth: 0,
        //设置移动样式
        swiperStyle: {},
        //当前图片，前面0是最后一张图，所以是1
        currentIndex: 1
      }
    },
    methods: {
      //初始化dom相关信息
      init() {
        //获取dom节点
        let swiperBody = this.$refs.swiperBody
        //获取slide节点
        let slideDom = swiperBody.getElementsByClassName('slide')
        //保存元素个数
        this.slideCount = slideDom.length
        //前后加一张假图，保证顺滑
        if (this.slideCount > 1) {
          //赋值
          let fakeFirst = slideDom[0].cloneNode(true)
          let fakeLast = slideDom[this.slideCount - 1].cloneNode(true)
          //插入
          swiperBody.insertBefore(fakeLast, slideDom[0])
          swiperBody.append(fakeFirst)
        }
        //获取偏移的距离
        this.totalWidth = swiperBody.offsetWidth
        this.swiperStyle = swiperBody.style
        //显示第一张图
        this.setTransform(-this.totalWidth)
      },
      startTimer() {
        //移动，设置计时器移动，停留时间为interval
        this.playTimer = window.setInterval(() => {
          this.currentIndex++;
          this.scrollContent(-this.currentIndex * this.totalWidth);
        }, this.interval)
      },
      stopTimer() {
        window.clearInterval(this.playTimer);
      },
      //滚动
      scrollContent(currentPosition) {
        //正在滚动
        this.isScrolling = true;
        //滚动动画animDuration秒
        this.swiperStyle.transition = 'transform ' + this.animDuration + 'ms';
        //移动到相应的位置
        this.setTransform(currentPosition);
        // 判断滚动到的位置
        this.checkPosition();
        // 滚动完成
        this.isScrolling = false
      },
      //设置每次滚动的距离
      setTransform(position) {
        this.swiperStyle.transform = `translate3d(${position}px, 0, 0)`;
        this.swiperStyle['-webkit-transform'] = `translate3d(${position}px), 0, 0`;
        this.swiperStyle['-ms-transform'] = `translate3d(${position}px), 0, 0`;
      },
      //把swiperStyle的状态复原，并且让
      // 第一张和最后一张指向真正的第一张和最后
      //保证动画的流畅，不会出现卡顿
      checkPosition() {
        window.setTimeout(() => {
          // 校验正确的位置
          //重置动画时间
          this.swiperStyle.transition = '0ms';
          //到达最后一张之后变第一张
          if (this.currentIndex >= this.slideCount + 1) {
            this.currentIndex = 1;
            this.setTransform(-this.currentIndex * this.totalWidth);
          }
          //如果是第一张，那么把他变成最后一张
          else if (this.currentIndex <= 0) {
            this.currentIndex = this.slideCount;
            this.setTransform(-this.currentIndex * this.totalWidth);
          }

          //结束移动后的回调
          this.$emit('transitionEnd', this.currentIndex - 1);
        }, this.animDuration)//在滚动完成后实现，两个时间一样
      },

      touchStart(e) {
        //只有在停留界面可拖动
        if (this.isScrolling) {
          return
        }
        //停止计时
        this.stopTimer()
        //保存当前位置
        this.pageX = e.touches[0].pageX
      },
      touchMove(e) {
        //当前位置
        this.currentPageX = e.touches[0].pageX
        //偏移位置
        this.distance = this.currentPageX - this.pageX
        //当前图片所在位置偏移
        let currentPosition = -this.currentIndex * this.totalWidth
        //相加得图片移动后偏移
        let moveDistance = this.distance + currentPosition;
        //设置移动后的正确位置
        this.setTransform(moveDistance)
      },
      touchEnd(e) {
        //保证正反都能翻页
        let currentMove = Math.abs(this.distance)
        //判断距离
        if (currentMove === 0) {
          return //没移动
        } else if (this.distance > 0 && currentMove > this.totalWidth * this.moveRatio) { // 右边移动超过0.5
          this.currentIndex--//往前移动
        } else if (this.distance < 0 && currentMove > this.totalWidth * this.moveRatio) { // 向左移动超过0.5
          this.currentIndex++//往后移动
        }

        // 移动到正确的位置
        this.scrollContent(-this.currentIndex * this.totalWidth);
        // 4.移动完成后重新开启定时器
        this.startTimer();
      }
    },

    mounted() {
      // 操作DOM, 在前后添加Slide
      setTimeout(() => {
        this.init();

        // 开启定时器
        this.startTimer();
      }, 100)
    }
  }
</script>

<style scoped>
  #my-swiper {
    position: relative;
    overflow: hidden;
  }

  .swiper-body {
    display: flex;
  }

  .indicator {
    display: flex;
    justify-content: center;
    position: absolute;
    width: 100%;
    /*在图片上显示*/
    bottom: 8px;
  }

  .indicator-item {
    box-sizing: border-box;
    width: 8px;
    height: 8px;
    border-radius: 4px;
    background-color: #cccccc;
    margin: 0 5px;
  }

  .indicator-item.active {
    background-color: rgba(212, 62, 46, 1.0);
  }
</style>