<template>
  <el-table
    :data="list"
    border
    fit
    highlight-current-row
    style="width: 100%"
    @selection-change="handleSelectionChange"
  >
    <el-table-column type="selection" width="55"> </el-table-column>
    <el-table-column label="内容">
      <template slot-scope="scope">
        <!-- <span>{{ scope.row.content }}</span> -->
        <img class="content-l" :src="scope.row.cover" alt="" width="100" />
            <div>
              <span>{{ scope.row.title }}</span> <br />
              <span class="content-price">￥： {{ scope.row.price }} </span>
            </div>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
import { fetchList as mediaList } from "@/api/media";
import { fetchList as audioList } from "@/api/audio";
import { fetchList as videoList } from "@/api/video";

export default {
  name: "ColumnPane",
  props: {
    type: {
      type: String,
      default: "TW",
    },
  },
  data() {
    return {
      list: [],
      listQuery: {
        page: 1,
        limit: 5,
        type: this.type,
        sort: "+id",
      },
      loading: false,
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      console.log(this.type);
      this.loading = true;
      this.$emit("create"); // for test
      if (this.type === "TW") {
        mediaList(this.listQuery).then((response) => {
          console.log(response);
          this.list = response.data.items;
          this.loading = false;
        });
      }else if(this.type === "SP"){
          videoList(this.listQuery).then((response) => {
          console.log(response);
          this.list = response.data.items;
          this.loading = false;
        });
      }else if(this.type === "YP"){
          audioList(this.listQuery).then((response) => {
          console.log(response);
          this.list = response.data.items;
          this.loading = false;
        });
      }
      /* fetchList(this.listQuery).then((response) => {
        console.log(response);
        this.list = response.data.items;
        this.loading = false;
      }); */
    },
    //全选相关
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
  },
};
</script>


<style scoped>
.content-l {
  float: left;
  margin-right: 20px;
}

.content-price {
  color: #f00;
}
</style>
