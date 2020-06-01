<template>
  <div>
    <screenshot-video
      v-if="screenshotActive"
      :windowSource="windowSource"
      @screenshotComplete="analyzeScreenshot"
    ></screenshot-video>
    <ark-card>
      <template v-slot:title>
        Controls
      </template>
      <ark-button :modifiers="['outline']" @click.native="getVideoSources"
        >Select Arknights
      </ark-button>
      <ark-button :modifiers="['action']" @click.native="screenshot"
        >Analyze</ark-button
      >
    </ark-card>
  </div>
</template>

<script>
  import { desktopCapturer, remote } from 'electron'
  import Jimp from 'jimp'

  import ArkCard from './ArkCard'
  import ArkButton from './ArkButton'
  import ScreenshotVideo from './ScreenshotVideo'

  import { TAGS, TAGS_DIMENSIONS } from '../utils/constants'

  const { Menu } = remote

  export default {
    components: {
      ArkCard,
      ArkButton,
      ScreenshotVideo
    },
    data() {
      return {
        windowSource: {},
        screenshotActive: false,
        detectedTags: []
      }
    },
    methods: {
      async getVideoSources() {
        console.log('Fired')
        const inputSources = await desktopCapturer.getSources({
          types: ['window', 'screen']
        })

        console.log(inputSources)

        const videoOptionsMenu = Menu.buildFromTemplate(
          inputSources.map(source => {
            return {
              label: source.name,
              click: () => (this.windowSource = source)
            }
          })
        )

        videoOptionsMenu.popup()
      },
      screenshot() {
        this.screenshotActive = true
      },
      async analyzeScreenshot(screenshot) {
        this.screenshotActive = false
        console.log(screenshot)
        const imageSplit = screenshot.image.split(',')[1]

        const prepTagsImagesPromises = []
        TAGS.forEach(tag => {
          prepTagsImagesPromises.push(
            this.prepTag(
              imageSplit,
              screenshot.width,
              screenshot.height,
              tag.x,
              tag.y
            )
          )
        })

        const croppedTagImages = await Promise.all(prepTagsImagesPromises)
          .then(result => {
            return result
          })
          .catch(e => {
            console.log(e)
          })

        console.log(croppedTagImages)
      },
      async prepTag(image, imageWidth, imageHeight, cropX, cropY) {
        return await Jimp.read(Buffer.from(image, 'base64'))
          .then(readImage => {
            return (
              readImage
                .crop(
                  cropX * imageWidth,
                  cropY * imageHeight,
                  TAGS_DIMENSIONS.WIDTH * imageWidth,
                  TAGS_DIMENSIONS.HEIGHT * imageHeight
                )
                .greyscale()
                .autocrop()
                .invert() // set greyscale
                .contrast(1)
                .posterize(2)
                // .write(cropX + cropY + '.jpg')
                .getBase64Async(Jimp.MIME_JPEG)
            )
          })
          .catch(err => {
            console.error(err)
          })
      }
    }
  }
</script>

<style lang="scss" scoped>
  body {
    $background: #333;
    background-color: $background;
  }
</style>
