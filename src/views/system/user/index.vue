<template>
  <div>
    <div class="head-container">
      <!--增删改查按钮-->
      <div class="crud-opts">
        <span class="crud-opts-left">
          <!--左侧插槽-->
          <slot name="left"/>
          <el-button
              class="filter-item"
              size="mini"
              type="primary"
              icon="el-icon-plus"
          >
            新增
          </el-button>
          <el-button
              class="filter-item"
              size="mini"
              type="success"
              icon="el-icon-edit"
          >
            修改
          </el-button>
          <el-button
              slot="reference"
              class="filter-item"
              type="danger"
              icon="el-icon-delete"
              size="mini"
          >
            删除
          </el-button>
          <el-button
              class="filter-item"
              size="mini"
              type="warning"
              icon="el-icon-download"
          >
            导出
          </el-button>
          <!--右侧-->
          <slot name="right"/>
        </span>
        <el-button-group class="crud-opts-right">
          <el-button
              size="mini"
              plain
              type="info"
              icon="el-icon-search"
          />
          <el-button
              size="mini"
              icon="el-icon-refresh"
          />
          <el-popover
              placement="bottom-end"
              width="150"
              trigger="click"
          >
            <el-button
                slot="reference"
                size="mini"
                icon="el-icon-s-grid"
            >
              <i
                  class="fa fa-caret-down"
                  aria-hidden="true"
              />
            </el-button>
            <el-checkbox>
              全选
            </el-checkbox>
          </el-popover>
        </el-button-group>
      </div>
    </div>
    <!--用户列表信息表格-->
    <el-table
        :data="tableData"
        style="width: 100%"
    >
      <el-table-column
          fixed
          prop="username"
          label="用户名"
          width="150">
      </el-table-column>
      <el-table-column
          prop="nickName"
          label="昵称"
          width="120">
      </el-table-column>
      <el-table-column
          prop="gender"
          label="性别"
          width="120">
      </el-table-column>
      <el-table-column
          prop="phone"
          label="电话"
          width="200">
      </el-table-column>
      <el-table-column
          prop="email"
          label="邮箱"
          width="200">
      </el-table-column>
      <el-table-column
          prop="dept.name"
          label="部门"
          width="120">
      </el-table-column>
      <el-table-column
          prop="enabled"
          label="状态"
          width="120">
        <template slot-scope="scope">
          <el-switch ref="enabled"></el-switch>
        </template>
      </el-table-column>
      <el-table-column
          :show-overflow-tooltip="true"
          prop="createTime"
          label="创建日期"
          width="200">
      </el-table-column>
      <el-table-column
          prop="operation"
          label="操作"
          width="120">
        <template slot-scope="scope">
          <el-button type="text" size="small">Edit</el-button>
          <el-button type="text" size="small">Delete</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  created() {
    this.getUserInfo()
  },

  name:'User',
  data() {
    return {
      tableData: [],
    }
  },
  methods: {
    //从后端获取用户信息
    getUserInfo() {
      this.$request.get('http://127.0.0.1:8000/api/users').then(res=>{
        this.tableData = res.data.content
        console.log('tableData:'+this.tableData)
      })
    },

    handleEdit(index, row) {
      console.log(index, row);
    },
    handleDelete(index, row) {
      console.log(index, row);
    }
  }
}
</script>
