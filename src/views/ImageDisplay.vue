<template>
  <div>
<!--    <img :src="imageUrl" alt="Displayed Image" />-->
    <img :src="require(`./R.jpg`)"  alt="Displayed Image" />

  </div>
</template>

<script>
export default {
   props: {
    imageUrl: {
      type: String,
      required: false,
      default: `./R.jpg`
    }
  }
}
</script>

<style scoped>
.img {
  width: 200px;
  height: auto;
  border-radius: 5%;
  border-style: groove;
}
.liveroom{
  width: 200px;
  height: auto;
  border-radius: 5%;
  border-style: groove;
}
</style>
