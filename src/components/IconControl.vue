<template>
  <i class="cursor-pointer" :class="[{'active': isActive}, iconClass]" @click.stop="$emit('emit-click')"></i>
</template>

<script>
export default {
  name: "IconControl",
  emits: ['emit-click'],
  props: {
    isActive: {
      type: Boolean,
      default: true
    },
    iconClass: {
      type: String,
      required: true
    }
  }
}
</script>

<style scoped>
i{
  margin: 0 5px;
  display: none;
}

i.active{
  display: inline;
}
</style>