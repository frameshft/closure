<template>
  <div>
    <ark-card>
      <template v-slot:title>
        Controls
      </template>
      <ark-button :modifiers="['outline']" @click.native="getVideoSources"
        >Select Arknights
      </ark-button>
      <ark-button :modifiers="['action']">Analyze</ark-button>
    </ark-card>
  </div>
</template>

<script>
  import { desktopCapturer, remote } from 'electron'
  import ArkCard from './ArkCard'
  import ArkButton from './ArkButton'

  const { Menu } = remote

  export default {
    components: {
      ArkCard,
      ArkButton
    },
    data() {
      return {
        windowSource: {}
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
