<template>
  <div class="screenshot-video">
    <video ref="video" @loadedmetadata="setup"></video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
  export default {
    props: {
      windowSource: {
        type: Object,
        default: {}
      }
    },
    data() {
      return {
        stream: {}
      }
    },
    async mounted() {
      const constraints = {
        audio: false,
        video: {
          mandatory: {
            chromeMediaSource: 'desktop',
            chromeMediaSourceId: this.windowSource.id
          }
        }
      }

      // Create a Stream
      this.stream = await navigator.mediaDevices.getUserMedia(constraints)
      this.$refs.video.srcObject = this.stream
    },
    beforeDestroy() {
      try {
        // Destroy connect to stream
        this.stream.getTracks()[0].stop()
      } catch (e) {}
    },
    methods: {
      setup() {
        const video = this.$refs.video
        const canvas = this.$refs.canvas
        video.style.height = video.videoHeight + 'px' // videoHeight
        video.style.width = video.videoWidth + 'px' // videoWidth

        video.play()

        canvas.width = video.videoWidth
        canvas.height = video.videoHeight
        const ctx = canvas.getContext('2d')
        // Draw video on canvas
        ctx.drawImage(video, 0, 0, canvas.width, canvas.height)

        this.$emit('screenshotComplete', {
          image: canvas.toDataURL('image/jpeg'),
          width: video.videoWidth,
          height: video.videoHeight
        })
      }
    }
  }
</script>

<style lang="scss" scoped>
  .screenshot-video {
    position: absolute;
    top: -10000px;
    left: -10000px;
  }
</style>
