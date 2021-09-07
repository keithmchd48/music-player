<template>
  <h1 class="heading">recently played</h1>
  <div class="playlists-group">
    <div class="playlist-card cursor-pointer" v-for="(song, index) in songs" :key="index"
         @click="clickRandomSong(song.id)">
      <img :src="song.cover" class="playlist-card-img" :alt="song.alt">
      <p class="playlist-card-name">{{ song.name }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "RecentlyPlayed",
  emits: ['click-random-song'],
  props: {
    songs: {
      type: Object,
      required: true
    }
  },
  methods: {
    clickRandomSong(id) {
      this.$emit('click-random-song', id)
    }
  }
}
</script>

<style scoped>
.heading{
  margin: 30px 0 10px;
  text-transform: capitalize;
  font-weight: 400;
  font-size: 30px;
}

.playlists-group{
  position: relative;
  width: 100%;
  min-height: 200px;
  height: auto;
  display: flex;
  flex-wrap: nowrap;
  overflow-x: auto;
}

.playlist-card{
  flex: 0 0 auto;
  max-width: 150px;
  height: 100%;
  margin-right: 20px;
}

.playlist-card-img{
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 20px;
}

.playlist-card-name{
  width: 100%;
  /*text-align: justify;*/
  font-size: 20px;
  text-transform: capitalize;
  padding: 5px;
}
</style>