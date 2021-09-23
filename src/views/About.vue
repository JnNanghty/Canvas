<template>
  <div class="about">
    <video ref="video" id="video" :width="width" :height="height" controls
           src="http://clips.vorwaerts-gmbh.de/big_buck_bunny.mp4"></video>
    <canvas ref="canvas" :width="width" :height="height"></canvas>
  </div>
</template>

<script>
export default {
  name: 'About',
  data() {
    return {
      ctx: null,
      video: null,
      width: 400,
      height: 300
    }
  },
  mounted() {
    this.loadVideo()
  },
  methods: {
    loadVideo() {
      this.video = this.$refs.video;
      this.ctx = this.$refs.canvas.getContext('2d');
      this.video.addEventListener('play', () => {
        this.timerCallback();
      });
    },
    timerCallback() {
      if(this.video.paused || this.video.ended) return
      this.drawFrame();
      setTimeout(this.timerCallback, 0);
    },
    drawFrame() {
      this.ctx.drawImage(this.video, 0, 0, this.width, this.height)
      // let frame = this.ctx.getImageData(0, 0, this.width, this.height)
    }
  }
}
</script>
<style scoped lang="stylus">
.about
  display flex
  justify-content space-around

</style>
