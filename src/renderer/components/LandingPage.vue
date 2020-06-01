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
  import ArkCard from './ArkCard'
  import ArkButton from './ArkButton'
  import ScreenshotVideo from './ScreenshotVideo'

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
        screenshotActive: false
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
      analyzeScreenshot(screenshot) {
        this.screenshotActive = false
        console.log(screenshot)
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
