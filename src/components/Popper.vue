<template>
  <h1 :style="`position:fixed;top:${y}px;left:${x}px;`">hello world</h1>
</template>
<script>
export default {
  name: 'popper',
  props: {
    x: Number,
    y: Number,
  },
}
</script>

<style>

</style>
