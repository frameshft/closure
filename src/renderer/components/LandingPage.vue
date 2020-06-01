<template>
  <div class="p-landing-page">
    <ark-loader v-if="analyzing"></ark-loader>
    <screenshot-video
      v-if="screenshotActive"
      :windowSource="windowSource"
      @screenshotComplete="analyzeScreenshot"
    ></screenshot-video>
    <ark-card>
      <template v-slot:title>Controls</template>
      <ark-button :modifiers="['outline']" @click.native="getVideoSources"
        >Select Arknights</ark-button
      >
      <ark-button :modifiers="['action']" @click.native="screenshot"
        >Analyze</ark-button
      >
    </ark-card>
    <div class="p-landing-page__results">
      <ark-card v-for="(ops, index) in filteredOperators" :key="index">
        <template v-slot:title>{{ ops.tags }}</template>
        <div class="p-landing-page__ops">
          <div
            class="p-landing-page__operator"
            v-for="op in ops.operators"
            :key="op.name"
          >
            <ark-avatar
              :src="`static/operators/${op.name}.png`"
              :alt="op.name"
              :level="op.level"
            ></ark-avatar>
          </div>
        </div>
      </ark-card>
    </div>
  </div>
</template>

<script>
  import { desktopCapturer, remote } from 'electron'
  import Jimp from 'jimp'
  import path from 'path'
  import { createWorker } from 'tesseract.js'

  import ArkCard from './ArkCard'
  import ArkButton from './ArkButton'
  import ArkAvatar from './ArkAvatar'
  import ArkLoader from './ArkLoader'
  import ScreenshotVideo from './ScreenshotVideo'
  import operatorsJSON from '../assets/json/operators.json'

  import { TAGS, TAGS_DIMENSIONS, RECRUITMENT_TAGS } from '../utils/constants'

  const { Menu } = remote

  operatorsJSON.map(operator => {
    operator.tags.push(operator.type)
    operator.tags = operator.tags.map(tag => tag.toLowerCase())
    return operator
  })

  const operators = operatorsJSON.filter(operator => {
    return !operator.hidden
  })

  export default {
    components: {
      ArkCard,
      ArkButton,
      ScreenshotVideo,
      ArkAvatar,
      ArkLoader
    },
    data() {
      return {
        windowSource: {},
        screenshotActive: false,
        detectedTags: [],
        filteredOperators: [],
        analyzing: false
      }
    },
    methods: {
      async getVideoSources() {
        const inputSources = await desktopCapturer.getSources({
          types: ['window', 'screen']
        })

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
        if (Object.keys(this.windowSource).length === 0) {
          alert('Please select an active Arknights window')
          return
        }
        if (this.analyzing) {
          return
        }
        this.screenshotActive = true
        this.analyzing = true
      },
      async analyzeScreenshot(screenshot) {
        this.screenshotActive = false
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

        const detectedTagsTemp = []
        for (const tag of croppedTagImages) {
          const detectedTag = await this.detectTagFromImage(tag)
          detectedTagsTemp.push(detectedTag)
        }
        this.detectedTags = detectedTagsTemp

        this.filteredOperators = this.filterOperators(this.detectedTags)
        this.analyzing = false
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
      },
      async detectTagFromImage(image) {
        const worker = createWorker({
          cachePath: path.join(__static, 'lang-data'),
          logger: m => console.log(m)
        })

        const base64string = image.split(',')[1]

        await worker.load()
        await worker.loadLanguage('eng')
        await worker.initialize('eng')

        const job = await worker.recognize(Buffer.from(base64string, 'base64'))
        await worker.terminate()
        return RECRUITMENT_TAGS.find(tag =>
          new RegExp('\\b' + tag + '\\b', 'i').test(job.data.text)
        )
      },
      filterOperators(tags) {
        const tagCombinations = this.getCombinations(tags)
        const filteredTagOperators = tagCombinations
          .map(combination => {
            const matchedOperators = operators.filter(operator => {
              if (
                operator.tags.includes('top operator') &&
                !combination.includes('top operator')
              ) {
                return false
              }
              return this.arrayContainsAnotherArray(combination, operator.tags)
            })

            const score =
              matchedOperators.length > 0
                ? matchedOperators.reduce(
                    (acc, operator) => acc + operator.level,
                    0
                  ) / matchedOperators.length
                : 0
            const minLevel =
              score !== 0
                ? Math.min(
                    ...matchedOperators
                      .map(op => op.level)
                      .filter(level => level > 1)
                  )
                : 0
            return {
              tags: combination,
              minLevel,
              score,
              operators: matchedOperators
            }
          })
          .filter(tags => tags.operators.length > 0)

        const weightedOperatorTags = filteredTagOperators
          .sort((a, b) => {
            if (a.minLevel > b.minLevel) return -1
            if (a.minLevel < b.minLevel) return 1
            if (a.score > b.score) return -1
            if (b.score > a.score) return 1
            if (a.tags.length > b.tags.length) return 1
            if (a.tags.length < b.tags.length) return -1
            if (a.operators.length > b.operators.length) return 1
            if (a.operators.length < b.operators.length) return -1
          })
          .map(entry => {
            return {
              tags: entry.tags
                .map(tag => {
                  return tag.charAt(0).toUpperCase() + tag.slice(1)
                })
                .toString(),
              operators: entry.operators.reverse(),
              score: entry.score,
              minLevel: entry.minLevel
            }
          })

        // console.log(weightedOperatorTags)
        // // Delete me later
        // const tableFormat = weightedOperatorTags.map(entry => {
        //   return {
        //     tags: entry.tags.toString(),
        //     operators: entry.operators.map(op => op.name).toString(),
        //     score: entry.score,
        //     minLevel: entry.minLevel
        //   }
        // })
        // console.table(tableFormat)
        return weightedOperatorTags
      },
      getCombinations(tags) {
        // array to hold results
        const result = []
        // number of times we need to iterate to check over all possible combinations
        const setCount = Math.pow(2, tags.length) - 1
        // loop over iterations
        for (let i = 0; i < setCount; i++) {
          // array to hold this result
          const innerList = []

          for (let j = 0; j < tags.length; j++) {
            // Each position in the initial list maps to a bit here
            let position = 1 << j
            // if the bits when bitwise AND are position then this is unique match
            if ((i & position) == position) {
              // insert into inner list
              innerList.push(tags[j])
            }
          }
          // insert into results if it isn't empty or a size of 4
          if (innerList.length !== 0 && innerList.length !== 4) {
            result.push(innerList)
          }
        }

        // purely for printing
        // for (let i = 0; i < result.length; i++) {
        //   console.log(result[i])
        // }
        return result
      },
      arrayContainsAnotherArray(needle, haystack) {
        for (var i = 0; i < needle.length; i++) {
          if (haystack.indexOf(needle[i]) === -1) return false
        }
        return true
      }
    }
  }
</script>

<style lang="scss" scoped>
  .p-landing-page {
    &__ops {
      display: flex;
      flex-wrap: wrap;
      margin: 0 -0.5rem;
    }

    &__operator {
      margin: 0 0.5rem 1rem 0.5rem;
    }
  }
</style>
