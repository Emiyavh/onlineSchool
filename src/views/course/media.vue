<template>
  <div class="container">
    <aside class="media-header">
      <el-button type="primary" icon="el-icon-edit"> 新增图文 </el-button>
      <div class="media-header-r">
        <el-select v-model="value" placeholder="图书状态">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
        <el-input
          v-model="listQuery.content"
          placeholder="请输入内容"
        ></el-input>
        <el-button type="primary" icon="el-icon-search"> 搜索 </el-button>
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
        <el-table-column label="状态" width="100">
          <template slot-scope="scope">
            <el-tag>
              {{ scope.row.status ? "已上架" : "已下架" }}
            </el-tag>
          </template>
        </el-table-column>
        <!-- 创建时间 -->
        <el-table-column label="创建时间" width="150px" align="center">
          <template slot-scope="scope">
            <span>{{
              scope.row.created_time | parseTime("{y}-{m}-{d} {h}:{i}")
            }}</span>
          </template>
        </el-table-column>
        <!-- 操作 -->
        <el-table-column label="操作" align="center" width="230">
          <template slot-scope="{ row, $index }">
            <el-button type="primary" size="mini" @click="handleUpdate(row)">
              编辑
            </el-button>
            <el-button
              v-if="row.status != 'published'"
              size="mini"
              type="success"
              @click="handleModifyStatus(row, 'published')"
            >
              上架
            </el-button>
            <el-button
              v-if="row.status != 'deleted'"
              size="mini"
              type="danger"
              @click="handleDelete(row, $index)"
            >
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";
import { parseTime } from "@/utils";

export default {
  name: "Media",
  data() {
    return {
      options: [
        {
          value: "选项1",
          label: "已上架",
        },
        {
          value: "选项2",
          label: "已下架",
        },
      ],
      listLoading: true,
      mediaList: [],
      listQuery: {},
      total: 0,
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
    // 编辑
    handleUpdate(row) {},
    //上架下架
    handleModifyStatus(row, status) {},
    //删除
    handleDelete(row, index) {},
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
.content-l{
    float: left;
    margin-right: 20px;
}

.content-price {
  color: #f00;
}
</style>