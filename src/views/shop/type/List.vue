<template>
  <div
    class="bg-white pl-2"
    style="margin: -20px; margin-top: -10px; margin-bottom: 0!important; "
  >
    <button-search
      class="pt-3"
      placeholder="要搜索的商品名称"
      :showSearch="false"
    >
      <template #left>
        <!-- 左边 -->
        <el-button type="success" size="mini" @click="openModel(false)"
          >添加类型</el-button
        >
        <el-button type="danger" size="mini" @click="deleteAll()"
          >批量删除</el-button
        >
      </template>
    </button-search>

    <!-- 商品数据列表 -->
    <div class="content">
      <el-table
        border
        :data="tableData"
        class=" mt-2"
        style="width: 100%; margin-bottom: 60px;"
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          type="selection"
          width="45"
          align="center"
        ></el-table-column>
        <el-table-column prop="name" label="类型名称"> </el-table-column>

        <el-table-column label="属性标签" width="380px" align="center">
          <template slot-scope="scope">
            {{ scope.row.value_list | formatValue }}
          </template>
        </el-table-column>

        <el-table-column
          label="属性排序"
          prop="order"
          align="center"
        ></el-table-column>

        <el-table-column label="状态" align="center">
          <template slot-scope="scope">
            <el-button
              :type="scope.row.status ? 'success' : 'danger'"
              size="mini"
              @click="changeStatus(scope.row)"
              >{{ scope.row.status ? "启用" : "禁用" }}</el-button
            >
          </template>
        </el-table-column>

        <el-table-column label="操作" align="center" width="150px">
          <template slot-scope="scope">
            <el-button-group>
              <el-button
                type="primary"
                size="mini"
                :plain="true"
                @click="openModel(scope)"
                >修改</el-button
              >
              <el-button
                type="danger"
                size="mini"
                :plain="true"
                @click="delectItem(scope.row)"
                >删除</el-button
              >
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 分页 -->
    <el-footer
      class="border-top d-flex align-items-center px-0 position-fixed bg-white"
      style="bottom: 0; left: 200px; right: 0; z-index: 9;"
    >
      <div style="flex: 1" class="px-2">
        <el-pagination
          :current-page="page.current"
          :page-sizes="page.sizes"
          :page-size="page.size"
          layout="total, sizes, prev, pager, next, jumper"
          :total="page.total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        >
        </el-pagination>
      </div>
    </el-footer>

    <!-- 新增/修改 规格模态框 -->
    <el-dialog
      title="添加类型"
      width="80%"
      :visible.sync="createModel"
      top="5vh"
    >
      <!-- 表单内容 -->
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="类型名称" prop="name">
          <el-input
            v-model="form.name"
            size="mini"
            style="width: 180px;"
            placeholder="类型名称"
          ></el-input>
        </el-form-item>
        <el-form-item label="排序">
          <el-input-number
            v-model="form.order"
            :min="0"
            placeholder=""
          ></el-input-number>
        </el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="form.status" size="mini">
            <el-radio :label="1" border>启用</el-radio>
            <el-radio :label="0" border>禁用</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="关联规格">
          <div class="d-flex">
            <span
              class="sku-list-item mx-2 px-3 py-2 border rounded"
              style="line-height: initial;"
              v-for="(item, index) in form.sku_list"
              :key="index"
            >
              <font>{{item.name}}</font>
              <i class="el-icon-delete" @click="deleteFormSkuList(index)"></i>
            </span>
            <el-button size="mini" class="el-icon-plus ml-2" @click="chooseSkus"></el-button>
          </div>
        </el-form-item>
        <el-form-item label="属性列表">
          <el-table :data="value_list" style="width: 100%">
            <el-table-column prop="order" label="排序" width="80">
              <template slot-scope="scope">
                <el-input
                  v-model="scope.row.order"
                  size="mini"
                  placeholder="排序"
                ></el-input>
              </template>
            </el-table-column>
            <el-table-column prop="name" label="属性名称" width="120">
              <template slot-scope="scope">
                <el-input
                  v-model="scope.row.name"
                  size="mini"
                  placeholder="属性名称"
                ></el-input>
              </template>
            </el-table-column>
            <el-table-column prop="type" label="所属类型" width="130">
              <template slot-scope="scope">
                <el-select
                  v-model="scope.row.type"
                  size="mini"
                  placeholder="请选择所属类型"
                >
                  <el-option label="输入框" value="input"></el-option>
                  <el-option label="单选框" value="radio"></el-option>
                  <el-option label="多选框" value="checkbox"></el-option>
                </el-select>
              </template>
            </el-table-column>
            <el-table-column prop="status" label="是否显示" width="60">
              <template slot-scope="scope">
                <el-switch
                  v-model="scope.row.status"
                  :active-value="1"
                  :inactive-value="0"
                ></el-switch>
              </template>
            </el-table-column>
            <el-table-column prop="default" label="属性值">
              <template slot-scope="scope" v-if="scope.row.type !== 'input'">
                <el-input
                  type="textarea"
                  v-model="scope.row.default"
                  size="mini"
                  placeholder="一行一个属性值，多个属性值用换行输入"
                  v-if="scope.row.isedit"
                ></el-input>
                <span v-else>{{scope.row.default}}</span>
              </template>
            </el-table-column>
            <el-table-column label="操作" width="130">
              <template slot-scope="scope">
                <el-button type="text" size="mini" v-if="scope.row.type !== 'input'" @click="editRow(scope)">{{scope.row.isedit ? '完成' : '编辑属性值'}}</el-button></el-button>
                <el-button type="text" size="mini" @click="delRow(scope.$index)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-form-item>
        <el-form-item label="">
          <el-button
            type="text"
            size="mini"
            icon="el-icon-plus"
            @click="addValue"
            >添加一个属性</el-button
          >
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="createModel = false">取 消</el-button>
        <el-button type="primary" @click="submit">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import buttonSearch from "components/common/buttonSearch";
import { table_page_Mixin } from "common/mixin.js";
export default {
  name: "List",
  inject: ["layout", 'app'],
  mixins: [table_page_Mixin],
  data() {
    return {
      preUrl: 'goods_type',
      tabIndex: 0,
      tableData: [
        // {
        //   name: "鞋子",
        //   order: 50,
        //   status: 1,
        //   sku_list: [
        //     { id: 1, name: "颜色" },
        //     { id: 2, name: "尺寸" },
        //   ],
        //   value_list: [
        //     {
        //       order: 50,
        //       name: "特性",
        //       type: "input",
        //       value: "",
        //     },
        //   ],
        // },
      ],
      // multipleSelection: [],
      createModel: false,
      editIndex: -1,  // 用于判断是修改或者添加
      form: {
        name: "",
        order: 50,
        status: 1,
        sku_list: [],
      },
      value_list: [
        // {
        //   id: 0,
        //   order: 50,
        //   name: "属性名称",
        //   type: "input",
        //   default: "属性值",
        //   isedit: false,
        // },
      ],
      rules: {
        name: [
          { required: true, message: "类型名称不能为空", trigger: "blur" },
        ],
      },
    };
  },
  components: {
    buttonSearch,
  },
  computed: {
    // 关联规格id组成的一维数组
    skus_id() {
      return this.form.sku_list.map(item => item.id);
    }
  },
  created() {},
  filters: {
    formatValue(value) {
      let arr = value.map((v) => v.name);
      return arr.join("，");
    },
  },
  methods: {
    // 处理请求结果， 可以在引入页面中重写
    getListResult(data) {
      // console.log(data);
      this.tableData = data.list.map(item => {
        item.value_list = item.goods_type_values;
        item.sku_list = item.skus;
        return item;
      });
      // console.log(data);
    },

    // 打开模态框
    openModel(e = false) {
      if (!e) {
        // 添加
        // 初始化表单
        this.form = {
          name: "",
          order: 50,
          status: 1,
          sku_list: [],
        };
        this.value_list = [];
        this.editIndex = -1;
      } else {
        // 修改
        this.form = {
          ...e.row
        };
        // 解构赋值
        this.value_list = [...e.row.value_list];
        this.editIndex = e.$index;
      }
      this.createModel = true;
    },

    // 禁启用状态
    /* changeStatus(item) {
      // 请求服务端数据修改状态
      item.status = !item.status;
      this.$message({
        message: !item.status ? "启用成功！" : "禁用成功！",
        type: "success",
      });
    }, */

    // 提交，添加类型
    submit() {
      this.$refs.form.validate((res) => {
        // 验证属性列表
        var result = true;
        var message = [];
        this.value_list.forEach((item, index) => {
          let no = index + 1;
          if (item.order == '') {
            result = result && false;
            message.push('第' + no + '行：排序不能为空');
          }
          if (item.name == '') {
            result = result && false;
            message.push('第' + no + '行：属性名称不能为空');
          }
          if (item.type !== 'input' && item.default == '') {
            result = result && false;
            message.push('第' + no + '行：属性值不能为空');
          }
        });

        if (!result) {
          var temp = '';
          message.forEach(v => {
            temp += `<li>${v}</li>`
          })
          return this.$notify({
            title: '提示',
            dangerouslyUseHTMLString: true,
            type: 'warning',
            duration: 3000,
            message: `<ul>${temp}</ul>`,
          });
        }

        if (res) {
          let value_list = this.value_list.map(item => {
              if (item.default) {
                item.default = item.default.replace(/\n/g, '，');
              }
              return item
            })
            let data = {
              ...this.form,
              skus_id: this.skus_id,
              value_list: [...value_list]
            }
            let id = 0;
            if(this.editIndex !== -1) {
              id = this.tableData[this.editIndex].id;
            }
            this.addOrEdit(data,id);
          /* if (this.editIndex === -1) {
            // this.tableData.unshift({
            //   ...this.form,
            //   value_list: [...this.value_list],
            // });
          } else {
            let item = this.tableData[this.editIndex];
            item.name = this.form.name;
            item.order = this.form.order;
            item.status = this.form.status;
            item.sku_list = this.form.sku_list;
            item.value_list = this.form.value_list;
          } */
          // 关闭模态框
          this.createModel = false;
          // this.$message({
          //   type: "success",
          //   message: this.editIndex === -1 ? "添加成功！" : "修改成功！",
          // });
        }
      });
    },

    // 删除单个
    /* delectItem(scope) {
      this.$confirm("是否要删除该规格？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "waring",
      }).then(() => {
        this.tableData.splice(scope.$index, 1);
        this.$message({
          type: "success",
          message: "删除成功！",
        });
      });
    }, */

    // // 批量删除
    // deleteAll() {
    //   this.multipleSelection.forEach((item) => {
    //     let index = this.tableData.findIndex((v) => v.id === item.id);
    //     if (index !== -1) {
    //       this.tableData.splice(index, 1);
    //     }
    //   });
    //   this.multipleSelection = [];
    // },

    /* // 监听表格项的选择
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
 */
   
    // 添加属性行
    addValue() {
      let item = {
        id: 0,
        order: 50,
        name: "",
        type: "input",
        value: "",
        isedit: false,
      };
      this.value_list.push(item);
    },
    
    // 编辑属性
    editRow(scope) {
      // if(scope.row.isedit) {
      //   let value_list = this.value_list.map(item => {
      //     item.default = item.default.replace(/\n/g, '，');
      //     return item
      //   })
      //   this.value_list = value_list
      // }
      scope.row.isedit = !scope.row.isedit;
    },
  
    // 删除属性行
    delRow(index) {
      this.value_list.splice(index, 1);
    },
  
    // 添加关联规格
    chooseSkus() {
      this.app.chooseSkus((e) => {
        let index = this.form.sku_list.findIndex(item => {
          return item.id === e.id;
        })
        if (index === -1) {
          this.form.sku_list.push(e);
        }
        
      })
    },
  
    // 删除关联规格
    deleteFormSkuList(index) {
      this.form.sku_list.splice(index, 1);
    }

  },
};
</script>
<style scoped>
.sku-list-item > i {
  display: none;
  cursor: pointer;
}

.sku-list-item:hover {
  background-color: #f4f4f4;
}

.sku-list-item:hover > font {
  display: none;
}

.sku-list-item:hover > i {
  display: inline-block;
}
</style>
