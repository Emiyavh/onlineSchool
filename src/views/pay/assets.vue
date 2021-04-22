<template>
  <div class="app-container">
    <div>
      <el-row :gutter="24">
        <el-col :span="6">
          <el-card shadow="hover">
            <div slot="header">
              <span>可用余额（元）</span>
              <el-button
                style="float: right; padding: 0"
                size="large"
                type="primary"
                @click="dialogFormVisible = true"
                >申请提现</el-button
              >
            </div>
            <h1>20.00</h1>
          </el-card>
        </el-col>
        <el-col :span="6">
          <el-card shadow="hover">
            <div slot="header">
              <span>累计收入（元）</span>
            </div>
            <h1>20.00</h1>
          </el-card>
        </el-col>
        <el-col :span="6">
          <el-card shadow="hover">
            <div slot="header">
              <span>待结算金额（元）</span>
            </div>
            <h1>20.00</h1>
          </el-card>
        </el-col>
        <el-col :span="6">
          <el-card shadow="hover">
            <div slot="header">
              <span>待结算金额（元）</span>
            </div>
            <h1>20.00</h1>
          </el-card>
        </el-col>
      </el-row>
    </div>
    <!-- 表格 -->
    <el-table
      :data="list"
      border
      fit
      hightlight-current-row
      style="width: 100%"
    >
      <!-- 交易时间 -->
      <el-table-column label="交易时间" align="center" width="120">
        <template slot-scope="scope">
          <span>{{ scope.row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="提款账号" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.account }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开户人" align="center" width="80">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="提现金额" align="center" width="200">
        <template slot-scope="scope">
          <span>￥ {{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <!-- 状态 -->
      <el-table-column label="状态" align="center" width="100">
        <template slot-scope="scope">
          <el-tag :type="scope.row.status ? '' : 'danger'">
            {{ scope.row.status ? "提现成功" : "待提现" }}
          </el-tag>
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
        <el-form-item >
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="AddSets('withdraw')">确 定</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import { fetchAssets } from "@/api/pay";
import Pagination from "@/components/Pagination";

export default {
  name: "Assets",
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
        account:""
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
      fetchAssets(this.listQuery)
        .then((res) => {
          console.log(res);
          if (res.code === 20000) {
            this.list = res.data.items;
            this.total = res.data.total;
            let middleArr = JSON.parse(JSON.stringify(res.data.items));
            this.withdraw.user = middleArr.reduce(function (arr, middleArr) {
              arr.push(middleArr.name);
              return arr;
            }, []);
            // console.log(this.withdraw.user);
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
            console.log(val)
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      this.dialogFormVisible = false;
    },
  },
};
</script>

<style scoped>
</style>


