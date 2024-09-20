<template>
  <div class="video-player-container">
    <video
      ref="videoPlayer"
      class="video-js vjs-default-skin"
      controls
      preload="auto"
      width="100%"
      height="100%"
    >
      <source :src="src" type="video/mp4">
    </video>
  </div>
</template>

<script>
import videojs from 'video.js'
import 'video.js/dist/video-js.css'

export default {
  name: 'VideoPlayer',
  props: {
    src: {
      type: String,
      required: true
    },
    subtitles: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      player: null
    }
  },
  mounted() {
    this.player = videojs(this.$refs.videoPlayer, {
      fluid: true,
      aspectRatio: '16:9'
    })
    this.player.src({ type: 'video/mp4', src: this.src })
    this.player.on('timeupdate', () => {
      this.$emit('timeupdate', this.player.currentTime())
    })
  },
  beforeUnmount() {  // 将 beforeDestroy 改为 beforeUnmount
    if (this.player) {
      this.player.dispose()
    }
  },
  methods: {
    setCurrentTime(time) {
      if (this.player) {
        this.player.currentTime(time)
      }
    },
    setCurrentTimeAndPlay(time) {
      if (this.player) {
        this.player.currentTime(time)
        this.player.play()
      }
    },
    updateSize() {
      if (this.player) {
        this.player.fluid(true)
      }
    }
  },
  watch: {
    src(newSrc) {
      if (this.player) {
        this.player.src({ type: 'video/mp4', src: newSrc })
      }
    }
  }
}
</script>

<style scoped>
.video-player-container {
  width: 100%;
  height: 100%;
}
</style>