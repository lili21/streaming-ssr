<script>
export default {
  data() {
    return {
      p3: false,
      p2: false,
    }
  },
  methods: {
    load() {
      this.p2 = false;
      this.p3 = false;
      setTimeout(() => {
        this.p3 = true;
      }, 1000)

      setTimeout(() => {
        this.p2 = true;
      }, 2000)
    }
  },
  mounted() {
    
  }
}
</script>

<template>
  <button @click="load">load</button>
  <div v-if="p2" class="part-2">
    data - 第二屏
  </div>
  <div v-if="p3" class="part-3">
    hero - 第三屏
  </div>
</template>

<style scoped>
button {
  width: 80px;
  height: 30px;
  border: 1px solid;
  line-height: 20px;
  text-align: center;
  margin-bottom: 10px;
}
.part-2 {
  width: 130px;
  height: 200px;
  /* border: 1px solid pink; */
  background-color: pink;
}

.part-3 {
  width: 130px;
  height: 200px;
  background-color:aqua;
}
</style>