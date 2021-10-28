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
      v-show="loading || !visible"
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
      visible: false,
      inited: false
    }
  },
  props: {
    loading: {
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
    },
    // png 的情况下，背景是透明的，在裁剪覆盖不设置此项会变成2个logo叠加态
    covered: {
      type: Boolean,
      default: true
    },
    coveredFillStyle: {
      type: String,
      default: '#ffffff'
    }
  },
  methods: {
    init () {
      return new Promise((resolve, reject) => {
        uni
          .createSelectorQuery()
          .in(this)
          .select('#canvas')
          .fields({
            node: true,
            size: true
          })
          .exec((res) => {
            if (res[0]) {
              this.canvas = canvas = res[0].node
              this.ctx = ctx = canvas.getContext('2d')
              canvas.width = 430
              canvas.height = 430
              this.inited = true
              resolve()
            } else {
              reject(new Error('can not find canvas node!'))
            }
          })
      })
    },
    async show () {
      if (!this.inited) {
        await this.init()
      }
      await this.drawBg()
      if (this.avatarUrl) {
        await this.drawAvatar()
      }
      this.visible = true
    },
    async drawAvatar () {
      const img = canvas.createImage()
      img.src = this.avatarUrl
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
          // console.log(img)
          ctx.save()
          // start 把中间干掉!
          ctx.arc(circle.x, circle.y, circle.radius, 0, Math.PI * 2, false)
          ctx.clip()

          const x1 = offsetX - borderWidth
          const y1 = offsetY - borderWidth
          const realDiam = circle.radius * 2
          if (this.covered) {
            ctx.fillStyle = this.coveredFillStyle
            ctx.fillRect(x1, y1, realDiam, realDiam)
          }
          // end
          ctx.drawImage(img, x1, y1, realDiam, realDiam)
          ctx.restore()
          resolve()
        }
        img.onerror = (event) => {
          reject(event)
        }
      })
    },
    async drawBg () {
      const img = canvas.createImage()
      img.src = this.src
      await new Promise((resolve, reject) => {
        img.onload = () => {
          ctx.drawImage(img, 0, 0, 430, 430)
          resolve()
        }
        img.onerror = (event) => {
          reject(event)
        }
      })
    },
    async preview () {
      if (this.src) {
        const src = await this.getImage()
        uni.previewImage({
          urls: [src]
        })
        return src
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
  async mounted () {
    await this.init()
    // console.log('canvase mounted')
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
