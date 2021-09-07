<template>

  <audio preload="metadata" :src="currentMusicSource" id="audio-source"></audio>

  <!-- home section -->

  <section class="home-section">
    <!-- carousel -->
    <Carousel :covers="carouselCovers"></Carousel>

    <!-- recently played -->
    <RecentlyPlayed :songs="recentlyPlayedSongs" @click-random-song="clickRandomSong"></RecentlyPlayed>
  </section>

<!--Music player section-->
  <section class="music-player-section" @click="toggleMusicPlayer(true)"
           :class="{'active': isMusicPlayerActive}">

    <img src="assets/covers/back.png" class="back-btn cursor-pointer icon hide" alt="back button"
         @click.stop="toggleMusicPlayer(false)">
    <img src="assets/covers/nav.png" class="nav-btn cursor-pointer icon hide" alt="" @click="togglePlaylist(true)">

    <h1 class="current-song-name">{{currentSongName}}</h1>
    <p class="artist-name hide">{{currentSongArtist}}</p>

    <img :src="currentSongCover" class="cover hide" alt="">

    <!--   Music Player   -->
    <Seekbar :seekBar="seekBar" :seekBarMax="seekBarMax"
             @change="jumpSeekBar"
             :currentSongTime="currentSongTime"
             :currentSongDuration="currentSongDuration">
    </Seekbar>

    <div class="controls">
      <span class="fas fa-redo cursor-pointer" :class="{'active': shouldSongRepeat}" @click="toggleRepeat"></span>
      <div class="main">
        <IconControl iconClass="fas fa-backward" :isActive="true" @emit-click="previousSong"></IconControl>
        <IconControl iconClass="fas fa-play" :isActive="isPaused" @emit-click="hitPlay"></IconControl>
        <IconControl iconClass="fas fa-pause" :isActive="!isPaused" @emit-click="hitPause"></IconControl>
        <IconControl iconClass="fas fa-forward" :isActive="true" @emit-click="nextSong"></IconControl>
      </div>
<!--      Volume control slider-->
      <input type="range" class="volume-slider" max="1" step="0.1"
             @input="changeVolume"
             v-model="musicVolume"
             :class="{'active': isVolumeSliderActive}">
      <span class="fas fa-volume-up cursor-pointer" :class="{'active': isVolumeSliderActive}"
            @click.stop="toggleVolumeSlider"></span>
    </div>

  </section>

<!--Playlist section-->
  <section class="playlist" :class="{'active': isPlaylistActive}">

    <img src="assets/covers/back.png" class="back-btn icon cursor-pointer"
         @click="togglePlaylist(false)" alt="">

    <h1 class="title">Playlist</h1>

    <div v-for="(song, ind) in allSongs" :key="ind" class="queue cursor-pointer"
         :class="{'active': currentMusic === ind}"
         @click.stop="clickRandomSong(ind)">
      <div class="queue-cover">
        <img :src="song.cover" alt="">
        <i class="fas fa-pause"></i>
      </div>
      <p class="name">{{song.name}}</p>
    </div>

  </section>
</template>

<script>
import {songs} from '/src/data/songs'
import Carousel from "@/components/Carousel";
import RecentlyPlayed from "@/components/RecentlyPlayed";
import Seekbar from "@/components/Seekbar";
import IconControl from "@/components/IconControl";
export default {
  name: 'App',
  components: {IconControl, Seekbar, RecentlyPlayed, Carousel},
  data () {
    return {
      allSongs: [],
      recentlyPlayedSongs: [],
      carouselCovers: [],
      carouselCoverIndex: 0,
      isMusicPlayerActive: false,
      isPlaylistActive: false,
      isVolumeSliderActive: false,
      music: null,
      seekBar: 0,
      seekBarMax: 0,
      currentMusic: 0,
      musicVolume: 1,
      isPaused: true,
      currentSongDuration: '00:00',
      currentSongTime: '00:00',
      shouldSongRepeat: false
    }
  },
  created() {
    this.allSongs = [...songs]
    this.carouselCovers = this.allSongs.map((s, ind) => {
      const {cover, alt} = s
      return {
        cover, alt,
        isActive: ind === 0 ? true : false // make the first cover as active in the beginning
      }
    })
  },
  mounted() {
    // set interval to change the cover carousel
    setInterval(() => {
      this.changeCarousel();
    }, 3000);
    this.music = document.querySelector('#audio-source')
    const ctx = this
    this.music.addEventListener("loadeddata", function() {
      ctx.audioDataLoaded(this.duration, this.currentTime)
    });
    this.setMusic(0)
    this.seekBarProgression()
  },
  computed: {
    currentMusicSource() {
      return this.allSongs[this.currentMusic].path
    },
    currentSongName() {
      return this.allSongs[this.currentMusic].name
    },
    currentSongArtist() {
      return this.allSongs[this.currentMusic].artist
    },
    currentSongCover() {
      return this.allSongs[this.currentMusic].cover
    }
  },
  methods: {
    formatTime(time) {
      let min = Math.floor(time / 60);
      if(min < 10){
        min = `0` + min;
      }

      let sec = Math.floor(time % 60);
      if(sec < 10){
        sec = `0` + sec;
      }

      return `${min} : ${sec}`;
    },
    // method for continously changing the cover on home section within 3 secs interval
    changeCarousel() {
      // set the previously active cover as inactive
      this.carouselCovers[this.carouselCoverIndex].isActive = false

      if(this.carouselCoverIndex >= this.carouselCovers.length - 1){
        this.carouselCoverIndex = 0;
      } else{
        this.carouselCoverIndex++;
      }
      // set the next carousel as active
      this.carouselCovers[this.carouselCoverIndex].isActive = true
    },
    toggleMusicPlayer(bool) {
      this.isMusicPlayerActive = bool
      this.isVolumeSliderActive = false
    },
    togglePlaylist(bool) {
      this.isPlaylistActive = bool
      this.isVolumeSliderActive = false
    },
    toggleVolumeSlider() {
      this.isVolumeSliderActive = !this.isVolumeSliderActive
    },
    hitPlay() {
      this.music.play();
      this.isPaused = false
    },
    hitPause() {
      this.music.pause();
      this.isPaused = true
    },
    nextSong() {
      if(this.currentMusic >= this.allSongs.length - 1){
        this.setMusic(0);
      } else{
        this.setMusic(this.currentMusic + 1);
      }
      this.$nextTick(() => {
        this.hitPlay()
      })
    },
    previousSong() {
      if(this.currentMusic <= 0){
        this.setMusic(this.allSongs.length - 1);
      } else{
        this.setMusic(this.currentMusic - 1);
      }
      this.$nextTick(() => {
        this.hitPlay()
      })
    },
    changeVolume() {
      this.music.volume = this.musicVolume
    },
    setMusic(i) {
      this.seekBar = 0;
      this.currentMusic = i
      this.addToRecentlyPlayed(i)
    },
    // when a random song is clicked upon from the playlist section
    clickRandomSong(i) {
      this.setMusic(i)
      this.$nextTick(() => {
        this.hitPlay()
      })
      this.isPlaylistActive = false
    },
    toggleRepeat() {
      this.shouldSongRepeat = !this.shouldSongRepeat
    },
    // this method runs every time a new song is added to the <audio> tag
    audioDataLoaded(duration, currentTime) {
      this.seekBarMax = duration
      this.currentSongDuration = this.formatTime(duration)
      this.currentSongTime = this.formatTime(currentTime)
    },
    jumpSeekBar(event) {
      this.music.currentTime = +event.target.value
    },
    // update the progress bar of the song every 500ms to move forward
    seekBarProgression() {
      setInterval(() => {
        this.seekBar = this.music.currentTime
        this.currentSongTime = this.formatTime(this.music.currentTime)
        // if the song ends, then check the condition whether repeat icon is on or off
        if(Math.floor(this.music.currentTime) === Math.floor(this.seekBarMax)){
          // is repeat icon ON
          if(this.shouldSongRepeat){
            this.setMusic(this.currentMusic);
            this.hitPlay()
          } else{
            this.nextSong()
          }
        }
      }, 500)
    },
    addToRecentlyPlayed(index) {
      // add the latest played song to the front of the array
      this.recentlyPlayedSongs = [this.allSongs[index], ...this.recentlyPlayedSongs]
      // if the array exceeds the song list number, then pop the song which was last
      if (this.recentlyPlayedSongs.length > this.allSongs.length) {
        this.recentlyPlayedSongs.pop()
      }
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap');

*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root{
  --background: #141414;
  --text-color: #fff;
  --primary-color: #63ff69;
  --secondary-color: #000;
  --alpha-color: rgba(0, 0, 0, 0.5);
  --shadow: 0 15px 40px var(--alpha-color);
}

html{
  background: var(--background);
  display: flex;
  justify-content: center;
}

body{
  width: 100%;
  height: 100vh;
  max-width: 375px;
  position: relative;
  background: var(--background);
  font-family: 'roboto', sans-serif;
  color: var(--text-color);
}

::-webkit-scrollbar{
  display: none;
}

/* home section */

.home-section{
  width: 100%;
  padding: 20px;
  height: 100%;
  padding-bottom: 100px;
  overflow-y: auto;
}

/* music player */

/* minimize view */

.music-player-section{
  width: 100%;
  height: 100px;
  position: fixed;
  bottom: 0;
  left: 0;
  background: var(--alpha-color);
  backdrop-filter: blur(50px);
  transition: 1s;
}

.current-song-name{
  font-weight: 300;
  font-size: 20px;
  text-align: center;
  margin-top: 5px;
  text-transform: capitalize;
}

.controls{
  position: relative;
  width: 80%;
  margin: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 60px;
  font-size: 30px;
}

.controls span{
  display: none;
  opacity: 0;
  transition: 1s;
}

.music-player-section.active .controls{
  justify-content: space-between;
}

.music-player-section.active .controls span{
  font-size: 25px;
  display: block;
  opacity: 0.5;
}

.music-player-section.active .controls span.active{
  color: var(--primary-color);
  opacity: 1;
}

/* maximize music player styles */

.music-player-section .hide{
  display: none;
  opacity: 0;
  transition: 1s;
}

.music-player-section.active .hide{
  display: block;
  opacity: 1;
}

.music-player-section.active{
  width: 100%;
  height: 100%;
  padding: 30px;
  display: flex;
  flex-direction: column;
}

.music-player-section.active .current-song-name{
  font-size: 40px;
}

.music-player-section.active .controls{
  width: 100%;
  font-size: 50px;
}

.artist-name{
  text-align: center;
  font-size: 20px;
  text-transform: capitalize;
}

.cover{
  width: 30vh;
  height: 30vh;
  object-fit: cover;
  margin: auto;
  border-radius: 20px;
  box-shadow: var(--shadow);
}

.icon{
  position: absolute;
  top: 60px;
  transform: scale(1.3);
}

.back-btn{
  left: 40px;
}

.nav-btn{
  right: 40px;
}

/* volume button */

.volume-slider{
  -webkit-appearance: none;
  width: 100px;
  /*height: 40px;*/
  position: absolute;
  right: -35px;
  bottom: 80px;
  transform: rotate(-90deg);
  border-radius: 20px;
  background: var(--alpha-color);
  overflow: hidden;
  opacity: 0;
  display: none;
}

.volume-slider.active{
  opacity: 1;
  display: block;
}

.volume-slider::-webkit-slider-thumb{
  -webkit-appearance: none;
  height: 40px;
  width: 10px;
  background: var(--primary-color);
  box-shadow: -200px 0 1px 200px var(--primary-color);
}

/* playlist section */

.playlist{
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  right: -100%;
  padding: 30px 0;
  background: var(--background);
  z-index: 3;
  transition: 1s;
  overflow: auto;
}

.playlist.active{
  right: 0;
}

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
.cursor-pointer {
  cursor: pointer;
}

input[type=range] {
  -webkit-appearance: none;
  /*width: 100%;*/
  background: transparent;
}

input[type=range]::-webkit-slider-runnable-track {
  -webkit-appearance: none;
  height: 1.5rem;
  border-radius: 0.75rem;
  background: #211f1f;
}

input[type=range]::-moz-range-track {
  -webkit-appearance: none;
  height: 1.5rem;
  border-radius: 0.75rem;
  background: #211f1f;
}

input[type=range]::-moz-range-progress {
  background: #211f1f;
  height: 1.5rem;
  border-radius: 0.75rem 0 0 0.75rem;
}

input[type=range]::-ms-track {
  -webkit-appearance: none;
  height: 1.5rem;
  border-radius: 0.75rem;
  background: #211f1f;
  color: transparent;
  border: 0;
}

input[type=range]::-ms-fill-lower {
  background: #1e3650;
  border-radius: 0.75rem 0 0 0.75rem;
}


input[type=range]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 1.5rem;
  width: 1.5rem;
  border-radius: .75rem;
  background: var(--primary-color);
  cursor: pointer;
  margin-top: 0;
}

input[type=range]::-moz-range-thumb {
  -webkit-appearance: none;
  height: 1.5rem;
  width: 1.5rem;
  border-radius: 0.75rem;
  background: var(--primary-color);
  cursor: pointer;
  margin-top: -14px;
}

input[type=range]::-ms-thumb {
  height: 1.5rem;
  width: 1.5rem;
  border-radius: 0.75rem;
  background: var(--primary-color);
  cursor: pointer;
  margin-top: -14px;
}
</style>
