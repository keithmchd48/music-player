<template>
  <div class="seek-bar-container">
    <input type="range" class="music-seek-bar cursor-pointer" @change="jumpSeekBar" :value="seekBar" :max="seekBarMax">
    <p class="current-time hide">{{currentSongTime}}</p>
    <p class="duration hide">{{currentSongDuration}}</p>
  </div>
</template>

<script>
export default {
  name: "Seekbar",
  props: {
    seekBar: {
      type: Number,
      required: true
    },
    seekBarMax: {
      type: Number,
      required: true
    },
    currentSongTime: {
      type: String,
      required: true
    },
    currentSongDuration: {
      type: String,
      required: true
    }
  },
  methods: {
    jumpSeekBar(event) {
      this.$emit('change', event)
    }
  }
}
</script>

<style scoped>
.music-seek-bar{
  -webkit-appearance: none;
  width: 100%;
  position: absolute;
  top: -4px;
  height: 8px;
  background: var(--secondary-color);
  overflow: hidden;
}

.music-player-section.active .music-seek-bar{
  position: relative;
  display: block;
  border-radius: 50px;
  margin: auto;
}
.current-time{
  position: absolute;
  margin-top: 5px;
  left: 30px;
}
.duration{
  position: absolute;
  margin-top: 5px;
  right: 30px;
}
</style>