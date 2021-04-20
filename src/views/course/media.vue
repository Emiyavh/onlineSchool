<template>
  <div class="container">
    <aside class="media-header">
      <el-button type="primary" icon="el-icon-edit" @click="handleAdd()">
        新增图文
      </el-button>
      <div class="media-header-r">
        <el-select v-model="listQuery.status" placeholder="图书状态">
          <el-option label="已上架" value="1"> </el-option>
          <el-option label="已下架" value="0"> </el-option>
        </el-select>
        <el-input
          v-model="listQuery.title"
          placeholder="请输入内容"
        ></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleFilter">
          搜索
        </el-button>
      </div>
    </aside>
    <el-card>
      <el-table
        :data="mediaList"
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
        <!-- 图文内容 -->
        <el-table-column label="图文内容" min-width="150px">
          <template slot-scope="scope">
            <img class="content-l" :src="scope.row.cover" alt="" width="100" />
            <div>
              <span>{{ scope.row.title }}</span> <br />
              <span class="content-price">￥： {{ scope.row.price }} </span>
            </div>
          </template>
        </el-table-column>
        <!-- 订阅量 -->
        <el-table-column label="订阅量" align="center" width="95">
          <template slot-scope="scope">
            <span> {{ scope.row.sub_count }} </span>
          </template>
        </el-table-column>
        <!-- 状态 -->
        <el-table-column label="状态" align="center" width="100">
          <template slot-scope="scope">
            <el-tag :type="scope.row.status ? '' : 'danger'">
              {{ scope.row.status ? "已上架" : "已下架" }}
            </el-tag>
          </template>
        </el-table-column>
        <!-- 创建时间 -->
        <el-table-column label="创建时间" width="150px" align="center">
          <template slot-scope="scope">
            <span>{{ scope.row.created_time }}</span>
          </template>
        </el-table-column>
        <!-- 操作 -->
        <el-table-column label="操作" align="center" width="230">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              @click="handleUpdate(scope.row)"
            >
              编辑
            </el-button>
            <el-button
              size="mini"
              :type="scope.row.status ? 'info' : 'success'"
              @click="handleStatusChange(scope.row)"
            >
              {{ scope.row.status ? "下架" : "上架" }}
            </el-button>
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
      <!-- 弹窗 -->
      <el-dialog
        :title="textMap[dialogStatus]"
        :visible.sync="dialogFormVisible"
      >
        <el-form
          ref="dataForm"
          :rules="rules"
          :model="temp"
          label-position="left"
          label-width="70px"
          style="width: 400px; margin-left: 50px"
        >
          <!-- 标题 -->
          <el-form-item label="标题" prop="title">
            <el-input v-model="temp.title" />
          </el-form-item>
          <!-- 封面 -->
          <el-form-item label="封面">
            <dropzone
              id="myVueDropzone"
              url="https://httpbin.org/post"
              @dropzone-removedFile="dropzoneR"
              @dropzone-success="dropzoneS"
            />
          </el-form-item>
          <!-- 试看内容 -->
          <el-form-item label="试看内容" prop="trycontent">
            <tinymce v-model="temp.trycontent" :height="200" :width="400" />
          </el-form-item>
          <!-- 课程内容 -->
          <el-form-item label="课程内容" prop="coursecontent">
            <tinymce v-model="temp.coursecontent" :height="200" :width="400" />
          </el-form-item>
          <!-- 课程价格 -->
          <el-form-item label="课程价格">
            <el-input-number
              v-model="temp.num"
              @change="priceChange"
              :min="0"
              :max="10"
              label="描述文字"
            ></el-input-number>
          </el-form-item>
          <!-- 状态 -->
          <el-form-item label="状态">
            <el-radio-group v-model="temp.status">
              <el-radio :label="3">下架</el-radio>
              <el-radio :label="6">上架</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false"> Cancel </el-button>
          <el-button
            type="primary"
            @click="dialogStatus === 'create' ? createData() : updateData()"
          >
            Confirm
          </el-button>
        </div>
      </el-dialog>
    </el-card>
  </div>
</template>


<script>
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";
import Pagination from "@/components/Pagination";
import Tinymce from "@/components/Tinymce";
import Dropzone from "@/components/Dropzone";

export default {
  name: "Media",
  components: { Pagination, Tinymce, Dropzone },
  data() {
    return {
      listLoading: true,
      mediaList: [],
      listQuery: {
        page: 1,
        title: "",
        limit: 20,
        status: "",
      },
      total: 0,
      temp: {
        title: "",
        cover: "",
        trycontent: "",
        coursecontent: "",
        num: 0,
        status: "",
      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        create: "新增",
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
          // console.log(res);
          console.log(this.listQuery)
          if (res.code === 20000) {
            this.mediaList = res.data.items;
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
    // 添加图文
    handleAdd() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          createMedia(this.temp)
            .then((res) => {
              // console.log(res)
              console.log(this.temp)
              if (res.code === 20000) {
                this.$message.success(res.msg);
                this.getList()
              } else {
                this.$message.error(res.msg);
              }
            })
            .catch((err) => {
              this.$message.error(err);
            });
        }
      });
    },
    // 编辑
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          tempData.timestamp = +new Date(tempData.timestamp);
          updateMedia(tempData).then((res) => {
            if (res.code === 20000) {
                this.$message.success(res.msg);
                this.getList()
              } else {
                this.$message.error(res.msg);
              }
            /* const index = this.mediaList.findIndex((v) => v.id === this.temp.id);
            this.mediaList.splice(index, 1, this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Update Successfully",
              type: "success",
              duration: 2000,
            }); */
          });
        }
      });
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
    //上架下架
    handleStatusChange(row) {
      // console.log(row)
      row.status = !row.status;
    },
    //删除
    handleDelete(row, index) {
      console.log(index);
      this.$confirm("确定要删除这条图文数据吗?", "温馨提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(() => {
          deleteMedia(row.id)
            .then((res) => {
              // console.log(res)
              if (res.code === 20000) {
                this.$message.success("删除成功");
                this.getList();
              }
            })
            .catch((err) => {
              this.$message.error("删除失败");
            });
        })
        .catch((err) => {
          this.$message.error("删除失败");
        });
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
  },
};
</script>

<style scoped>
.container {
  margin: 30px;
}
.media-header {
  display: flex;
  justify-content: space-between;
}
.media-header-r {
  display: flex;
}
.content-l {
  float: left;
  margin-right: 20px;
}

.content-price {
  color: #f00;
}
</style>