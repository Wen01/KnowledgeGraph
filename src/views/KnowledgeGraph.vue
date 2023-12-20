<template>
  <div id="mainScreen">
    <div class="title">
      <el-input
        v-model="searchWord"
      >
        <i slot="suffix" class="el-icon-search"></i>
      </el-input>
    </div>
    <div class="content">
      <ForceDirectGraph />
    </div>
  </div>
</template>

<script>
import ForceDirectGraph from '@/components/ForceDirectGraph.vue'
export default {
  name: 'KnowledgeGraph',
  components: {
    ForceDirectGraph
  },
  data() {
    return {
      searchWord: ''
    }
  }
}
</script>

<style scoped lang="scss">
//1920*1080为设计稿，头部字体大小为25px
$ratio: 1920 / 76.8; // 缩放比   25
// 返回对应屏幕下的rem值
@function px2rem($px) {
  @return $px / $ratio + rem;
}
#mainScreen {
  background: url('../assets/fullScreen_bigBg.png') center no-repeat;
  background-size: 100%;
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
  min-height: 100vh;
}
.title {
  background: url('../assets/fullScreen_titleBg.png') center
  no-repeat;
  background-size: 100%;
  width: 100%;
  height: px2rem(90);
  position: relative;
}
::v-deep .title .el-input__inner,
::v-deep .title .el-input__inner:focus {
  width: 230px;
  height: 23px;
  margin-top: 18px;
  border-radius: 15px;
  margin-left: -10px;
  background-color: transparent;
  caret-color: cyan;
  outline: 2px solid cyan; // 调整边框颜色，border-color不管用
  text-indent: 10px; // 调整光标位置
  color: cyan;
}

::v-deep .el-con-search {
  color: cyan;
  font-size: 16px;
}

</style>