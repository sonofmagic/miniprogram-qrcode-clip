<template>
  <view
    class="image-container"
    @click="preview"
  >
    <canvas
      :class="visible ? '': 'canvas pro' "
      type="2d"
      id="canvas"
      :style="{
      width:width+'rpx',
      height:height+'rpx'
    }"
    ></canvas>
    <view
      :style="{
      width:width+'rpx',
      height:height+'rpx'
    }"
      v-show="!visible"
      class="flex-center"
    >
      <Loading size="72"></Loading>
    </view>
  </view>

</template>

<script>
import Loading from './Loading.vue'
let canvas
let ctx
export default {
  components: {
    Loading
  },
  data () {
    return {
      visible: false
    }
  },
  props: {
    show: {
      type: Boolean,
      default: false
    },
    src: {
      type: String,
      default: ''
    },
    avatarUrl: {
      type: String,
      default: ''
    },
    width: {
      type: Number,
      default: 300
    },
    height: {
      type: Number,
      default: 300
    }
  },
  watch: {
    async show (newValue) {
      if (newValue) {
        await this.drawBg()
        await this.drawAvatar()
      }
      this.visible = newValue
    }
  },
  methods: {
    set (k, v) {
      this[k] = v
    },
    async drawAvatar () {
      const [err, res] = await uni.getImageInfo({
        src: this.avatarUrl
      })
      if (err) {
        throw err
      }
      // cloud:// 前缀的云存储url也可以哟
      const { path } = res
      console.log(res)
      const img = canvas.createImage()
      img.src = path
      const offsetX = 120
      const offsetY = 120
      const diam = 190
      const radius = diam / 2 // (430 - 120* 2) / 2
      const borderWidth = 2
      const circle = {
        x: offsetX + radius,
        y: offsetY + radius,
        radius: radius + borderWidth // 多加2px来把纯色边抹除
      }
      await new Promise((resolve, reject) => {
        img.onload = () => {
          console.log(img)
          ctx.save()
          // start 把中间干掉!
          ctx.arc(circle.x, circle.y, circle.radius, 0, Math.PI * 2, false)
          ctx.clip()
          // end
          ctx.drawImage(
            img,
            offsetX - borderWidth,
            offsetY - borderWidth,
            circle.radius * 2,
            circle.radius * 2
          )
          ctx.restore()
          resolve()
        }
        img.onerror = (event) => {
          reject(event)
        }
      })
    },
    async drawBg () {
      const [err, res] = await uni.getImageInfo({
        src: this.src
      })
      if (err) {
        throw err
      }
      const { path } = res
      const img = canvas.createImage()
      img.src = path
      await new Promise((resolve, reject) => {
        img.onload = () => {
          ctx.drawImage(img, 0, 0, 430, 430)
          resolve()
        }
        img.onerror = (event) => {
          reject(event)
        }
      })
      // const img = canvas.createImage()
    },
    async preview () {
      if (this.src) {
        const src = await this.getImage()
        uni.previewImage({
          urls: [src]
        })
      }
    },
    async getImage () {
      const [err, res] = await uni.canvasToTempFilePath({
        canvas
      })
      if (err) {
        throw err
      }
      return res.tempFilePath
    }
  },
  mounted () {
    // console.log('canvase mounted')
    uni
      .createSelectorQuery()
      .in(this)
      .select('#canvas')
      .fields({
        node: true,
        size: true
      })
      .exec((res) => {
        // （更换api后废弃） 强行设置 dpr 不然太慢 dpr 3.5 真机3800+ms，超过了3秒钟，自动 fallback
        // console.log(res)
        if (res[0]) {
          // const width = res[0].width
          // const height = res[0].height

          this.canvas = canvas = res[0].node
          this.ctx = ctx = canvas.getContext('2d')
          canvas.width = 430
          canvas.height = 430
        }

        // dpr = this.$store.getters.pixelRatio // this.dpr // this.$store.getters.pixelRatio
        // canvas.width = width * dpr
        // canvas.height = height * dpr
      })
  }
}
</script>

<style lang="scss">
.canvas.pro {
  position: absolute;
  bottom: 0;
  left: -9999rpx;
}
.image-container {
  position: relative;
}
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
