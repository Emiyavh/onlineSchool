<template>
  <el-table
    :data="list"
    border
    fit
    highlight-current-row
    style="width: 100%"
    @selection-change="handleSelectionChange"
  >
    <div v-if="type === 'kc'">
      <el-table-column label="课程标题">
        <template slot-scope="scope">
          <span>{{ scope.row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="购买价格">
        <template slot-scope="scope">
          <span>{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="购买时间">
        <template slot-scope="scope">
          <span>{{ scope.row.created_time }}</span>
        </template>
      </el-table-column>
    </div>
    <!-- 专栏订阅 -->
    <div v-if="type === 'zl'">
      <el-table-column label="专栏标题">
        <template slot-scope="scope">
          <span>{{ scope.row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="购买价格">
        <template slot-scope="scope">
          <span>{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="购买时间">
        <template slot-scope="scope">
          <span>{{ scope.row.created_time }}</span>
        </template>
      </el-table-column>
    </div>
    <!-- 订单记录-->
    <div v-if="type === 'dd'">
      <el-table-column label="ID">
        <template slot-scope="scope">
          <span>{{ scope.row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="订单号">
        <template slot-scope="scope">
          <span>{{ scope.row.no }}</span>
        </template>
      </el-table-column>
      <el-table-column label="购买价格">
        <template slot-scope="scope">
          <span>{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="状态">
        <template slot-scope="scope">
          <span>{{ scope.row.status }}</span>
        </template>
      </el-table-column>
      <el-table-column label="商品">
        <template slot-scope="scope">
          <span>{{ scope.row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="购买时间">
        <template slot-scope="scope">
          <span>{{ scope.row.created_time }}</span>
        </template>
      </el-table-column>
    </div>
    <!-- 观看历史 -->
    <div v-if="type === 'gk'">
      <el-table-column label="课程标题">
        <template slot-scope="scope">
          <span>{{ scope.row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="课程类型">
        <template slot-scope="scope">
          <span>{{ scope.row.type }}</span>
        </template>
      </el-table-column>
      <el-table-column label="学习时长">
        <template slot-scope="scope">
          <span>{{ scope.row.total_time }}</span>
        </template>
      </el-table-column>
    </div>
    <!-- 用户评论 -->
    <div v-if="type === 'yh'">
      <el-table-column label="评论内容">
        <template slot-scope="scope">
          <span>{{ scope.row.content }}</span>
        </template>
      </el-table-column>
      <el-table-column label="评论时间">
        <template slot-scope="scope">
          <span>{{ scope.row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="课程标题">
        <template slot-scope="scope">
          <span>{{ scope.row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="类型">
        <template slot-scope="scope">
          <span>{{ scope.row.type }}</span>
        </template>
      </el-table-column>
    </div>
  </el-table>
</template>

<script>
import {
  fetchUserCourse,
  fetchUserColumn,
  fetchUserOrder,
  fetchUserHistory,
  fetchUserComment,
} from "@/api/user";

export default {
  name: "UserPane",
  props: {
    type: {
      type: String,
      default: "kc",
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
      if (this.type === "kc") {
        fetchUserCourse(this.listQuery).then((response) => {
          console.log(response);
          this.list = response.data.items;
          this.loading = false;
        });
      } else if (this.type === "zl") {
        fetchUserColumn(this.listQuery).then((response) => {
          console.log(response);
          this.list = response.data.items;
          this.loading = false;
        });
      } else if (this.type === "dd") {
        fetchUserOrder(this.listQuery).then((response) => {
          console.log(response);
          this.list = response.data.items;
          this.loading = false;
        });
      } else if (this.type === "gk") {
        fetchUserHistory(this.listQuery).then((response) => {
          console.log(response);
          this.list = response.data.items;
          this.loading = false;
        });
      } else if (this.type === "yh") {
        fetchUserComment(this.listQuery).then((response) => {
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
