<template>
  <h1 class="title"><slot></slot></h1>

  <div v-for="(song, ind) in songs" :key="ind" class="queue cursor-pointer"
       :class="{'active': currentMusic === ind}"
       @click.stop="$emit('click-random-song', ind)">
    <div class="queue-cover">
      <img :src="song.cover" alt="">
      <i class="fas fa-pause"></i>
    </div>
    <p class="name">{{song.name}}</p>
  </div>
</template>

<script>
export default {
  name: "Playlist",
  emits: ['click-random-song'],
  props: {
    songs: {
      type: Object,
      required: true
    },
    currentMusic: {
      type: Number,
      required: true
    }
  }
}
</script>

<style scoped>
.title{
  font-weight: 300;
  font-size: 40px;
  text-align: center;
  margin-top: 15px;
  text-transform: capitalize;
  margin-bottom: 30px;
}
.queue{
  width: 100%;
  height: 80px;
  padding: 0 30px;
  display: flex;
  align-items: center;
  border-top: 2px solid var(--alpha-color);
}

.queue-cover{
  width: 60px;
  height: 60px;
  border-radius: 10px;
  overflow: hidden;
  margin-right: 20px;
  position: relative;
}

.queue-cover img{
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.queue-cover i{
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 30px;
  color: var(--primary-color);
  display: none;
}

.queue.active i{
  display: block;
}

.queue .name{
  font-size: 22px;
  text-transform: capitalize;
}
</style>