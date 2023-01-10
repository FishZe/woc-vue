<template>
  <el-footer class="Footer">
    <span class="Footer-Words">
      <a href="https://beian.miit.gov.cn" rel="external nofollow" target="_blank" style="color:RGB(153,156,158);text-decoration:none;">苏ICP备2022044127号</a>
    </span>
  </el-footer>
</template>

<script>
export default {
  name: "Footer"
}
</script>

<style scoped>
.Footer {
  color:RGB(153,156,158);
  background-color: RGB(54,59,63);
  display: flex;
  width: 100%;
}
.Footer-Words {
  margin: auto;
  text-align: center;
  line-height: 40px;
}
</style>