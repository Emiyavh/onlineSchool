<template>
  <div class="container">
    <aside class="user-header">
      <el-dropdown>
        <el-button type="danger">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item>禁止评论</el-dropdown-item>
          <el-dropdown-item>禁止访问</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <div class="user-header-r">
        <el-input v-model="listQuery.title" placeholder="请输入内容"></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleFilter">
          搜索
        </el-button>
      </div>
    </aside>
    <el-card>
      <el-table
        :data="userList"
        v-loading="listLoading"
        border
        fit
        hightlight-current-row
        style="width: 100%"
        @sort-change="sortChange"
      >
        <!-- id -->
        <el-table-column
          label="ID"
          prop="id"
          sortable="custom"
          align="center"
          width="80"
        >
          <template slot-scope="scope">
            <span>{{ scope.row.id }}</span>
          </template>
        </el-table-column>
        <!-- 用户内容 -->
        <el-table-column label="用户" min-width="150px">
          <template slot-scope="scope">
            <img
              class="content-l"
              :src="scope.row.user.avatar"
              alt=""
              width="50"
            />
            <div>
              <span class="content-r">{{ scope.row.user.username }}</span>
            </div>
          </template>
        </el-table-column>
        <!-- 消费总额 -->
        <el-table-column label="消费总额" align="center" width="95">
          <template slot-scope="scope">
            <span> {{ scope.row.total_consume }} </span>
          </template>
        </el-table-column>

        <!-- 创建时间 -->
        <el-table-column label="创建时间" width="150px" align="center">
          <template slot-scope="scope">
            <span>{{ scope.row.created_time }}</span>
          </template>
        </el-table-column>
        <!-- 操作 -->
        <el-table-column label="操作" align="center" width="300">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              @click="handleMessage(scope.row.user)"
            >
              详情
            </el-button>
            <el-button size="mini" type="success"> 联系用户 </el-button>
            <el-dropdown>
              <el-button type="danger" size="mini">
                黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
              </el-button>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item>禁止评论</el-dropdown-item>
                <el-dropdown-item>禁止访问</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
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
      <!-- 弹窗 -->
      <el-dialog
        :title="textMap[dialogStatus]"
        :visible.sync="dialogFormVisible"
        :dialog-data="dialogData"
      >
        <div class="dialog-top">
          <div>
            <span>ID：{{ dialogData.id }} </span>
            <span>用户名：{{ dialogData.username }} </span>
            <span
              >昵称：{{ dialogData.nickname ? dialogData.nickname : "未设定" }}
            </span>
            <span>手机号：{{ dialogData.phone }} </span>
          </div>

          <div>
            <span>锁定：{{ dialogData.status ? "未锁定" : "已锁定" }} </span>
            <span>会员：{{ dialogData.user_level }} </span>
            <span>会员到期时间：{{ dialogData.user_level_expire }} </span>
            <span>注册时间：{{ dialogData.created_time }} </span>
          </div>
        </div>
        <el-tabs
          v-model="activeName"
          style="margin-top: 15px"
          type="border-card"
        >
          <el-tab-pane
            v-for="item in tabMapOptions"
            :key="item.key"
            :label="item.label"
            :name="item.key"
          >
            <keep-alive>
              <user-pane
                v-if="activeName == item.key"
                :type="item.key"
                @create="showCreatedTimes"
              />
            </keep-alive>
          </el-tab-pane>
        </el-tabs>
        <div slot="footer" class="dialog-footer">
          <!-- 页码 -->
          <pagination
            v-show="total > 0"
            :total="total"
            :page.sync="listQuery.page"
            :limit.sync="listQuery.limit"
            @pagination="getList"
          />
        </div>
      </el-dialog>
    </el-card>
  </div>
</template>


<script>
import { fetchList, createUser, updateUser, deleteUser } from "@/api/user";
import Pagination from "@/components/Pagination";
import Tinymce from "@/components/Tinymce";
import Dropzone from "@/components/Dropzone";
import UserPane from "./components/userPane";

export default {
  name: "User",
  components: { Pagination, Tinymce, Dropzone, UserPane },
  data() {
    return {
      listLoading: true,
      userList: [],
      listQuery: {
        page: 1,
        title: "",
        limit: 20,
      },
      total: 0,

      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        create: "用户详情",
        update: "编辑",
      },
      rules: {
        title: [{ required: true, message: "请填写标题", trigger: "blur" }],
        trycontent: [
          { required: true, message: "请填写试看内容", trigger: "blur" },
        ],
        coursecontent: [
          { required: true, message: "请填写课程内容", trigger: "blur" },
        ],
      },
      dialogImageUrl: "",
      dialogVisible: false,

      //弹出框内容
      tabMapOptions: [
        { label: "课程订阅", key: "kc" },
        { label: "专栏订阅", key: "zl" },
        { label: "订单记录", key: "dd" },
        { label: "观看历史", key: "gk" },
        { label: "用户评论", key: "yh" },
      ],
      activeName: "kc",
      createdTimes: 0,
      dialogData: {},
    };
  },
  created() {
    this.getList();
  },
  methods: {
    //   获取列表
    getList() {
      this.listLoading = true;
      fetchList(this.listQuery)
        .then((res) => {
          console.log(res);
          //   console.log(this.listQuery)
          if (res.code === 20000) {
            this.userList = res.data.items;
            this.total = res.data.total;
            this.listLoading = false;
          }
        })
        .catch((err) => {
          console.log("表格获取失败");
        });
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        importance: 1,
        remark: "",
        timestamp: new Date(),
        title: "",
        status: "",
        type: "",
      };
    },
    // 添加用户
    handleMessage(val) {
      this.resetTemp();
      console.log(val);
      this.dialogData = val;
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
    },

    //排序
    sortChange(data) {
      const { prop, order } = data;
      if (prop === "id") {
        this.sortByID(order);
      }
    },
    sortByID(order) {
      if (order === "ascending") {
        this.listQuery.sort = "+id";
      } else {
        this.listQuery.sort = "-id";
      }
      this.handleFilter();
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
    },
    //搜索
    handleSearch(val) {
      // console.log(val)
      this.listQuery.page = 1;
      this.getList();
    },

    //上传图片
    dropzoneS(file) {
      console.log(file);
      this.$message({ message: "Upload success", type: "success" });
    },
    dropzoneR(file) {
      console.log(file);
      this.$message({ message: "Delete success", type: "success" });
    },
    //修改价格
    priceChange(val) {
      // console.log(val);
    },
    showCreatedTimes() {
      this.createdTimes = this.createdTimes + 1;
    },
  },
};
</script>

<style scoped>
.container {
  margin: 30px;
}
.user-header {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}

.user-header-r {
  display: flex;
}
.content-l {
  float: left;
  margin-right: 20px;
  border-radius: 50%;
}
.content-r {
  vertical-align: middle;
}
.content-price {
  color: #f00;
}
.dialog-top div{
  display: table;
}
.dialog-top span {
  display: table-cell;
  text-align: center;
  width: 25%;
}
</style>