# miniprogram-qrcode-clip

clip miniprogram qrcode center circle area at client side to your custom image!

[原理(Principle)](./Principle_CN.md)

## Demo

From:

![Form](./assets/icebreaker.raw.jpg)

To:

![To](./assets/custom.jpg)

## How to use it

### uni-app:

```ts
import QrcodeImage from 'miniprogram-qrcode-clip/libs/uni/Image.vue'

export default Vue.extend({
  ...
  components: { QrcodeImage }
  ...
}

<template>
  <QrcodeImage
    :loading="false"
    :src="qrcodeUrl"
    :avatarUrl="avatarUrl"
    :width="440"
    :height="440"
    ref="QrcodeImage"
  ></QrcodeImage>
</template>


await this.$refs.QrcodeImage?.preview()
await this.$refs.QrcodeImage?.show()
await this.$refs.QrcodeImage?.getImage()
```

> notice: current version not support default qrcode size (430px * 430px)

## Props
|attr|type|desc|
|---|---|---|
|loading|Boolean|loading status|
|src|string|background image url|
|avatarUrl|string|center circle image url|
|width|number|width(rpx)|
|height|number|height(rpx)|
|covered|Boolean|default: true(for .png transparent background)|
|coveredFillStyle|string|default: #ffffff coveredFillStyle|

## Methods

|name|return type|desc|
|---|---|---|
|init|void|init|
|show|Promise\<void>|show qrcode|
|drawAvatar|Promise\<void>|drawAvatar|
|drawBg|Promise\<void>|drawBg|
|preview|Promise\<void>|preview image|
|getImage|Promise\<string>| get tempFilePath|