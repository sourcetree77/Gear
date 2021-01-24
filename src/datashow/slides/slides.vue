<template>
  <div ref="sliderWrapper" class="sliderWrapper" :style="imgContentStyle"
       @mouseover="stop"
       @mouseleave="play">
    <ul class="container" :style="containerStyle">
      <li :style="imgContentStyle">
        <a :href="slides[slides.length - 1].clickUrl">
          <img class="img-content" :style="imgContentStyle" :src="slides[slides.length - 1].img" alt="">
        </a>
      </li>
      <li v-for="(item, index) in slides" :key="item.img" :style="imgContentStyle">
        <a :href="item.clickUrl">
          <img class="img-content" :style="imgContentStyle" :src="item.img" alt="">
        </a>
      </li>
      <li :style="imgContentStyle">
        <a :href="slides[0].clickUrl">
          <img class="img-content" :style="imgContentStyle" :src="slides[0].img" alt="">
        </a>
      </li>
    </ul>
    <ul class="direction" v-if="showControl">
      <li class="left" @click="move(imgWidth, 1, speed)"
          :style="iconWrapperStyle">
        <gear-icon name="left"
                   :style="iconStyle"
        ></gear-icon>
      </li>
      <li class="right" @click="move(imgWidth, -1, speed)"
          :style="iconWrapperStyle">
        <gear-icon name="right"
                   :style="iconStyle"
        ></gear-icon>
      </li>
    </ul>
    <ul class="dots" v-if="showControl && showSelected">
      <li v-for="(dot, i) in slides" :key="i"
          :class="{dotted: i === (currentIndex-1)}"
          :style="{width: (i === (currentIndex-1)?imgWidth/10 +'px':imgWidth/40 +'px'), height: imgWidth/40 + 'px', borderRadius: imgWidth/80 + 'px' }"
          @click=switchItem(i+1)
      >
      </li>
    </ul>
  </div>
</template>

<script>
  import Icon from '../../base/icon/icon'

  export default {
    name: 'GearSlides',
    components: {
      'gear-icon': Icon
    },
    props: {
      // 轮播速度
      rollSpeed: {
        type: Number,
        default: 20
      },
      // 自动播放间隔
      rollAuto: {
        type: Number,
      },
      // 轮播图数据
      slides: {
        type: Array
      },
      // 设置宽度
      setWidth: {
        type: Number,
        default: 600
      },
      // 设置高度
      setHeight: {
        type: Number
      },
      // 控制按钮显示
      selectedButton: {
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        imgWidth: this.setWidth,
        imgHeight: this.setHeight,
        currentIndex: 1,
        distance: -this.setWidth,
        transitionEnd: true,
        speed: this.rollSpeed,
        showControl: true,
        showSelected: true
      }
    },
    computed: {
      // 格式化轮播样式
      imgContentStyle() {
        if(this.imgHeight) {
          // 传 setHight 时, 设置高度
          return {
            width: this.imgWidth + 'px',
            height: this.imgHeight + 'px'
          }
        } else {
          // 不传 setHight 时, 宽高比 1.5 比例自适应
          return {
            width: this.imgWidth + 'px',
            height: this.imgWidth / 1.5 + 'px'
          }
        }
      },
      iconWrapperStyle() {
        return {
          width: this.imgWidth / 12 + 'px',
          height: this.imgWidth / 12 + 'px'
        }
      },
      iconStyle() {
        return {
          width: this.imgWidth / 20 + 'px',
          height: this.imgWidth / 20 + 'px'
        }
      },
      containerStyle() {
        return {
          transform: `translateX(${this.distance}px)`
        }
      },
      // 自动播放时间计算
      interval() {
        if(this.rollAuto) {
          return this.rollAuto * 1000
        }
      }
    },
    mounted() {
      // 底部选择按钮 是否显示
      if(!this.selectedButton) {
        this.showSelected = false
      }
      // 图片只有一张 不显示 control 按钮
      if(this.slides.length <= 1) {
        this.showControl = false
      }
      console.log(this.slides)
      console.log(this.imgHeight)
      let sliderWrapper = this.$refs.sliderWrapper
      console.log(sliderWrapper)
      this.play()
      sliderWrapper.addEventListener('mouseenter', this.stop())
      sliderWrapper.addEventListener('mouseleave', this.play())
    },
    methods: {
      move(offset, direction, speed) {
        console.log(speed)
        // 少于等于一张 不做轮播操作
        if(this.slides.length <= 1) {
          return
        }
        if (!this.transitionEnd) return
        this.transitionEnd = false
        direction === -1 ? this.currentIndex += offset / this.imgWidth : this.currentIndex -= offset / this.imgWidth
        if (this.currentIndex > this.slides.length) this.currentIndex = 1
        if (this.currentIndex < 1) this.currentIndex = this.slides.length

        const destination = this.distance + offset * direction
        this.animate(destination, direction, speed)
      },
      animate(des, direc, speed) {
        if (this.temp) {
          clearInterval(this.temp);
          this.temp = null;
        }
        this.temp = setInterval(() => {
          if ((direc === -1 && des < this.distance) || (direc === 1 && des > this.distance)) {
            this.distance += speed * direc
          } else {
            this.transitionEnd = true
            clearInterval(this.temp)
            this.distance = des
            if (des < -this.imgWidth * this.slides.length) this.distance = -this.imgWidth
            if (des > -this.imgWidth) this.distance = -this.imgWidth * this.slides.length
          }
        }, 20)
      },
      // 按钮跳转
      switchItem(index) {
        const direction = index - this.currentIndex >= 0 ? -1 : 1;
        const offset = Math.abs(index - this.currentIndex) * this.imgWidth;
        const switchSpeed = Math.abs(index - this.currentIndex) === 0 ? this.speed : Math.abs(index - this.currentIndex) * this.speed;
        this.move(offset, direction, switchSpeed)
      },
      // 轮播
      play() {
        // 设置了 rollAuto 参数才会自动播放
        if(this.rollAuto) {
          console.log('开始')
          if (this.timer) {
            clearInterval(this.timer)
            this.timer = null
          }
          this.timer = setInterval(() => {
            this.move(this.imgWidth, -1, this.speed)
          }, this.interval)
        }
      },
      // 暂停
      stop() {
        if(this.rollAuto) {
          console.log('暂停')
          clearInterval(this.timer)
          this.timer = null
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  ol, ul {
    list-style: none;
  }

  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  .sliderWrapper {
    position: relative;
    width: 600px;
    height: 400px;
    overflow: hidden;

    .container {
      display: flex;
      position: absolute;
    }

    .left, .right {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      width: 50px;
      height: 50px;
      background-color: rgba(0, 0, 0, .3);
      border-radius: 50%;
      cursor: pointer;
    }

    .left {
      left: 3%;
      padding-left: 12px;
      padding-top: 10px;
    }

    .right {
      right: 3%;
      padding-right: 12px;
      padding-top: 10px;
    }

    .img-content {
      user-select: none;
    }

    .dots {
      position: absolute;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
    }

    .dots li {
      display: inline-block;
      width: 15px;
      height: 15px;
      margin: 0 3px;
      border: 1px solid #f9f9f9;
      background-color: #333;
      opacity: 0.8;
      cursor: pointer;
      border-radius: 50%;
      transform: 1s;
    }

    .dots .dotted {
      background-color: #f9f9f9;
      opacity: 0.8;
      width: 60px;
      height: 15px;
      border-radius: 7.5px;
    }

    .gear-icon {
      width: 30px;
      height: 30px;
      fill: white;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
  }


</style>
