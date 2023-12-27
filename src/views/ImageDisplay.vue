<template>
  <div>
<!--    <img :src="imageUrl" alt="Displayed Image" />-->
    <img :src="require(`./R.jpg`)" alt="Displayed Image" />

  </div>
</template>

<script>
export default {
   props: {
    imageUrl: {
      type: String,
      required: true
    }
  }
}
</script>

<style scoped>
img {
  max-width: 100%; /* 图片宽度最大为容器宽度 */
  height: auto; /* 高度自适应 */
  display: block; /* 块级展示 */
  margin: 0 auto; /* 图片水平居中 */
}
</style>
