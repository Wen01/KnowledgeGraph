<template>
  <div class="dialog-container">
    <div class="dialog-content">
      <div class="search">
        <el-input v-model.trim="searchInput"></el-input>
        <el-button type="primary" icon="el-icon-search" @click="search"></el-button>
      </div>
      <div class="search-records">
        <p v-if="searchRecords.length === 0">No search Records</p>
        <ul v-else>
          <li v-for="(record, index) in searchRecords" :key="index">{{ record }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchInput: '', // Use a local data property for two-way binding
      searchRecords: []
    };
  },
  watch: {
    searchInput(newVal) {
      this.localSearchInput = newVal;
    },
  },
  methods: {
    search() {
      if (this.searchInput !== '') {
        this.searchRecords.unshift(this.searchInput); // Add the search term to the beginning of the array
        this.$emit('sendWord', this.searchInput)
        this.searchInput = ''; // Clear the input after searching
      }
    }
  }
}
</script>

<style scoped lang="scss">
.dialog-container {
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  width: 20%;
  background-color: rgba(0, 7, 44);
  overflow-y: auto; /* 允许滚动 */
  display: flex;
  flex-direction: column;
}
.dialog-content {
  flex: 1;
  margin: 10px;
  box-sizing: border-box;
  background-color: rgba(0, 11, 66); /* 对话框内容背景色 */
  border: 1px solid cyan;
}
.search {
  display: flex;
  margin: 10px;
  .el-input {
    margin-right: 10px;
  }
  ::v-deep .el-input__inner {
    height: 30px;
  }
  ::v-deep .el-button {
    padding: 7px 12px;
  }
}
.search-records {
  margin-top: 10px;
  ul {
    list-style: none;
    padding: 0;
  }
  li {
    background-color: rgba(255, 255, 255, 0.1); /* Style for each search record */
    margin: 5px 0;
    padding: 5px 12px;
    color: cyan;
    text-align: left;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
</style>