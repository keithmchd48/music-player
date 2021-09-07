<template>
  <div class="carousel">
    <img v-for="(cover, index) in covers" :key="index"
         :src="cover.cover"
         :class="{'active': cover.isActive}"
         :alt="cover.alt">
  </div>
</template>

<script>
export default {
  name: "Carousel",
  props: {
    covers: {
      type: Object,
      required: true
    }
  }
}
</script>

<style scoped>
.carousel{
  width: 100%;
  height: 200px;
  overflow: hidden;
  border-radius: 20px;
  box-shadow: var(--shadow);
  position: relative;
}

.carousel img{
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: 1s;
}

.carousel img.active{
  opacity: 1;
}

</style>