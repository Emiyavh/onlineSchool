<template>
  <div class="app-container">
    <aside class="order-header">
      <el-button
        :loading="downloadLoading"
        type="primary"
        icon="el-icon-document"
        @click="handleDownload"
      >
        导出选中
      </el-button>
      <div class="order-header-r">
        <el-input v-model="listQuery.title" placeholder="请输入内容"></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleFilter">
          搜索
        </el-button>
      </div>
    </aside>

    <!-- 表格 -->
    <el-table
      ref="multipleTable"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" align="center" />
      <el-table-column align="center" label="订单号" width="95">
        <template slot-scope="scope">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column label="商品名称" width="300">
        <template slot-scope="scope">
          {{ scope.row.goods[0].title }}
        </template>
      </el-table-column>
      <el-table-column label="订单类型" width="115" align="center">
        <template slot-scope="scope">
          {{ scope.row.type }}
        </template>
      </el-table-column>
      <el-table-column label="订单状态" width="110" align="center">
        <template slot-scope="scope">
          <el-tag :type="scope.row.status">{{ scope.row.status }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" label="原付/实付（元）" width="220">
        <template slot-scope="scope">
          
          <span>{{ scope.row.total_price }} / {{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="支付方式">
        <template slot-scope="scope">
          {{ scope.row.pay_method }}
        </template>
      </el-table-column>
      <el-table-column label="创建/支付时间">
        <template slot-scope="scope">
          {{ scope.row.pay_time }}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.row, scope.$index)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { fetchOrder, deleteOrder } from "@/api/pay";

export default {
  name: "Order",
  data() {
    return {
      list: [],
      listLoading: true,
      multipleSelection: [],
      downloadLoading: false,
      filename: "",
      total: 0,
      listQuery: {
        page: 1,

        title: "",
      },
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      this.listLoading = true;
      fetchOrder(this.listQuery).then((response) => {
        console.log(response);
        this.list = response.data.items;
        this.total = response.data.total;
        this.listLoading = false;
      });
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleDownload() {
      if (this.multipleSelection.length) {
        this.downloadLoading = true;
        import("@/vendor/Export2Excel").then((excel) => {
          const tHeader = [
            "id",
            "goods[0].title",
            "type",
            "status",
            "total_price",
            "pay_method",
            "pay_time",
          ];
          const filterVal = [
            "id",
            "goods[0].title",
            "type",
            "status",
            "total_price",
            "pay_method",
            "pay_time",
          ];
          const list = this.multipleSelection;
          const data = this.formatJson(filterVal, list);
          excel.export_json_to_excel({
            header: tHeader,
            data,
            filename: this.filename,
          });
          this.$refs.multipleTable.clearSelection();
          this.downloadLoading = false;
        });
      } else {
        this.$message({
          message: "Please select at least one item",
          type: "warning",
        });
      }
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map((v) => filterVal.map((j) => v[j]));
    },
    //搜索
    handleFilter() {
      this.listQuery.page = 1;
      this.fetchData();
    },
    //删除
    handleDelete(row, index) {
    //   console.log(index);
      this.$confirm("确定要删除这条视频数据吗?", "温馨提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(() => {
          deleteOrder(row.id)
            .then((res) => {
            //   console.log(res)
              if (res.code === 20000) {
                this.$message.success("删除成功");
                this.list.splice(index, 1);
                // this.getList();
              }
            })
            .catch((err) => {
              this.$message.error("失败了");
            });
        })
        .catch((err) => {
          this.$message.error(err);
        });
    },
  },
};
</script>

<style scoped>
.order-header {
  display: flex;
  justify-content: space-between;
}
.order-header-r {
  display: flex;
}
</style>

