<template>
  <div class="container">
    <el-card class="topCard">
      <img :src="this.$route.params.item.cover" alt="" class="topCard-l" />
      <div class="topCard-r">
        <div>
          <h4 class="top-title">{{ this.$route.params.item.title }}</h4>
          <span class="fr">
            {{ this.$route.params.item.isend ? "已完结" : "连载中" }}
          </span>
        </div>
        <p class="top-content">{{ this.$route.params.item.content }}</p>
        <p class="top-price">￥：{{ this.$route.params.item.price }}</p>
        <div>
          <el-button
            @click="handleStatusChange"
            :type="this.$route.params.item.status ? 'primary' : 'info'"
          >
            {{ this.$route.params.item.status ? "上架" : "下架" }}
          </el-button>
          <el-button @click="handleIsEnd"
            >设为
            {{ this.$route.params.item.isend ? "连载中" : "已完结" }}</el-button
          >
        </div>
      </div>
    </el-card>
    <aside class="column-header">
      <el-button type="primary" icon="el-icon-edit" @click="handleAdd()">
        新增目录
      </el-button>
      <div class="column-header-r">
        <el-select v-model="listQuery.status" placeholder="商品状态">
          <el-option label="已上架" value="1"> </el-option>
          <el-option label="已下架" value="0"> </el-option>
        </el-select>
        <el-input v-model="listQuery.title" placeholder="请输入内容"></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleFilter">
          搜索
        </el-button>
      </div>
    </aside>
    <el-card>
      <el-table
        ref="dragTable"
        :data="columnList"
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
        <!-- 单品内容 -->
        <el-table-column label="单品内容" min-width="150px">
          <template slot-scope="scope">
            <img class="content-l" :src="scope.row.cover" alt="" width="100" />
            <div>
              <span>{{ scope.row.title }}</span> <br />
              <span class="content-price">￥： {{ scope.row.price }} </span>
            </div>
          </template>
        </el-table-column>
        <!-- 更新状态 -->
        <!-- <el-table-column label="更新状态" align="center">
          <template slot-scope="scope">
            <div :style="{ color: scope.row.isend ? 'black' : 'red' }">
              <span> {{ scope.row.isend ? "已完结" : "连载中" }} </span>
            </div>
          </template>
        </el-table-column> -->
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
              type="danger"
              @click="handleDelete(scope.row, scope.$index)"
            >
              删除
            </el-button>
          </template>
        </el-table-column>
        <el-table-column align="center" label="拖曳" width="80">
          <template slot-scope="{}">
            <svg-icon class="drag-handler" icon-class="drag" />
          </template>
        </el-table-column>
      </el-table>
      <div class="show-d">
        <el-tag>The default order :</el-tag> {{ oldList }}
      </div>
      <div class="show-d">
        <el-tag>The after dragging order :</el-tag> {{ newList }}
      </div>
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
              <column-pane
                v-if="activeName == item.key"
                :type="item.key"
                @create="showCreatedTimes"
              />
            </keep-alive>
          </el-tab-pane>
        </el-tabs>
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
import {
  fetchList,
  createColumn,
  updateColumn,
  deleteColumn,
} from "@/api/column";
import Pagination from "@/components/Pagination";
import Tinymce from "@/components/Tinymce";
import Dropzone from "@/components/Dropzone";
import Sortable from "sortablejs";

//新增表格相关
import ColumnPane from './components/columnPane.vue';

export default {
  name: "ColumnLog",
  components: { Pagination, Tinymce, Dropzone, ColumnPane },
  data() {
    return {
      listLoading: true,
      columnList: [],

      listQuery: {
        page: 1,
        title: "",
        limit: 20,
        status: "",
      },
      total: 0,

      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        create: "新增",
        update: "编辑",
      },

      dialogImageUrl: "",
      dialogVisible: false,
      oldList: [],
      newList: [],

      //弹出框内容
      tabMapOptions: [
        { label: "图文", key: "TW" },
        { label: "音频", key: "YP" },
        { label: "视频", key: "SP" },
      ],
      activeName: "TW",
      createdTimes: 0,
    };
  },
  created() {
    console.log(this.$route.params.item);

    this.getList();
  },
  methods: {
    //   获取列表
    /* getList() {
      this.listLoading = true;
      fetchList(this.listQuery)
        .then((res) => {
          if (res.code === 20000) {
            this.columnList = res.data.items;
            this.total = res.data.total;
            this.listLoading = false;
            this.oldList = this.list.map((v) => v.id);
            this.newList = this.oldList.slice();
            this.$nextTick(() => {
              this.setSort();
            });
          }
        })
        .catch((err) => {
          console.log("表格获取失败");
        });
    }, */
    async getList() {
      this.listLoading = true;
      const { data } = await fetchList(this.listQuery);
      this.columnList = data.items;
      this.total = data.total;
      this.listLoading = false;
      this.oldList = this.columnList.map((v) => v.id);
      this.newList = this.oldList.slice();
      this.$nextTick(() => {
        this.setSort();
      });
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll(
        ".el-table__body-wrapper > table > tbody"
      )[0];
      this.sortable = Sortable.create(el, {
        ghostClass: "sortable-ghost", // Class name for the drop placeholder,
        setData: function (dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData("Text", "");
        },
        onEnd: (evt) => {
          const targetRow = this.columnList.splice(evt.oldIndex, 1)[0];
          this.columnList.splice(evt.newIndex, 0, targetRow);

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0];
          this.newList.splice(evt.newIndex, 0, tempIndex);
        },
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
    // 添加目录
    handleAdd() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
    },
    // 编辑
    handleUpdate(row) {
      console.log(row)
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

    //删除
    handleDelete(row, index) {
      console.log(index);
      this.$confirm("确定要删除这条目录数据吗?", "温馨提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(() => {
          deleteColumn(row.id)
            .then((res) => {
              // console.log(res)
              if (res.code === 20000) {
                this.$message.success("删除成功");
                this.columnList.splice(index, 1);
                // this.getList();
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

    //上架下架
    handleStatusChange() {
      this.$route.params.item.status = !this.$route.params.item.status;
    },
    // 设置已完结
    handleIsEnd() {
      this.$route.params.item.isend = !this.$route.params.item.isend;
    },

    showCreatedTimes() {
      this.createdTimes = this.createdTimes + 1
    }
  },
};
</script>

<style>
.sortable-ghost {
  opacity: 0.8;
  color: #fff !important;
  background: #42b983 !important;
}
</style>

<style scoped>
.container {
  margin: 30px;
}
.column-header {
  display: flex;
  justify-content: space-between;
}
.column-header-r {
  display: flex;
}
.content-l {
  float: left;
  margin-right: 20px;
}

.content-price {
  color: #f00;
}

.topCard-l {
  width: 200px;
  float: left;
  margin-right: 20px;
}
.topCard-r {
  display: flex;
  flex-direction: column;
}
.fr {
  float: right;
  color: #ccc;
}
.top-title {
  margin: 5px;
}
.top-content {
  color: #ccc;
  margin: 5px;
}
.top-price {
  color: #f00;
  margin: 5px;
}
.drag-handler {
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d {
  margin-top: 15px;
}
</style>