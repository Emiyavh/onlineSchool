<template>
  <div class="app-container">
    <el-button type="primary" icon="el-icon-edit" @click="dialogFormVisible = true">新增收款账号</el-button>
    <!-- 表格 -->
    <el-table
      :data="list"
      border
      fit
      hightlight-current-row
      style="width: 100%"
    >
      <el-table-column label="账号" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.account }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开户人" align="center" width="80">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开户行" align="center" width="300">
        <template slot-scope="scope">
          <span>{{ scope.row.path }}</span>
        </template>
      </el-table-column>
      <!-- 操作 -->
      <el-table-column label="操作" align="center" width="300">
        <template slot-scope="scope">
          <el-button size="mini" type="primary"> 编辑 </el-button>
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
    <!-- 页码 -->
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
    <!-- 弹出框 -->
    <el-dialog title="提现" :visible.sync="dialogFormVisible">
      <el-form :model="withdraw" ref="withdraw">
        <el-form-item label="提现金额">
          <el-input v-model="withdraw.price" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="体现账户">
          <el-select v-model="withdraw.users" placeholder="请选择提现账户">
            <el-option
              v-for="item in withdraw.user"
              :key="item.value"
              :label="item"
              :value="item.price"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="AddSets('withdraw')"
            >确 定</el-button
          >
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>


<script>
import { fetchPayments,deletePayment } from "@/api/pay";
import Pagination from "@/components/Pagination";

export default {
  name: "PaySetting",
  components: { Pagination },
  data() {
    return {
      listLoading: true,
      list: [],
      total: 0,
      listQuery: {
        page: 1,
        limit: 20,
      },
      dialogFormVisible: false,
      withdraw: {
        price: "",
        users: "",
        user: {},
        account: "",
      },
      user: "",
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      fetchPayments(this.listQuery)
        .then((res) => {
          console.log(res);
          if (res.code === 20000) {
            this.list = res.data.items;
            this.total = res.data.total;

            this.listLoading = false;
          }
        })
        .catch();
    },
    AddSets(val) {
      //   console.log(val);
      this.$refs[val].validate((valid) => {
        if (valid) {
          // alert(valid);
          console.log(val);
        } else {
          console.log("error submit!!");
          return false;
        }
      });
      this.dialogFormVisible = false;
    },
    //删除
    handleDelete(row, index) {
    //   console.log(index);
      this.$confirm("确定要删除这条视频数据吗?", "温馨提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(() => {
          deletePayment(row.id)
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
</style>


