<template>
  <div>
    <el-container
      style="position: absolute; top: 55px; bottom: 0;left: 0;right: 0;"
    >
      <!-- 头部 -->
      <el-header class="d-flex align-items-center border-bottom">
        <div class="d-flex mr-auto">
          <el-select
            v-model="searchForm.order"
            placeholder="请选择图片排序方式"
            size="mini"
            style="width: 150px;"
            class="mr-2"
          >
            <el-option label="降序" value="dsc"></el-option>
            <el-option label="升序" value="asc"></el-option>
          </el-select>
          <el-input
            v-model="searchForm.keyWord"
            class="mr-2"
            style="width: 150px;"
            size="mini"
            placeholder="输入图片名称"
          ></el-input>
          <el-button type="success" size="mini" @click="getImageList"
            >搜索</el-button
          >
        </div>
        <div class="d-flex">
          <el-button
            type="warning"
            size="mini"
            @click="unChoose"
            v-if="chooseList.length > 0"
            >取消选中</el-button
          >
          <el-button
            type="danger"
            size="mini"
            @click="imageDel({ all: true })"
            v-if="chooseList.length > 0"
            >批量删除</el-button
          >
          <el-button type="success" size="mini" @click="openAlbumModel()"
            >创建相册</el-button
          >
          <el-button type="warning" size="mini" @click="openUploadModel"
            >上传图片</el-button
          >
        </div>
      </el-header>

      <el-container>
        <!-- 侧部 -->
        <el-aside
          width="200px"
          style="position: absolute; top: 60px; left: 0; bottom: 60px;"
          class="bg-white border-right"
        >
          <!-- 侧边 | 相册列表 -->
          <ul class="list-group list-group-flush">
            <album-item
              v-for="(item, index) in albums"
              :key="index"
              :item="item"
              :index="index"
              :active="albumIndex === index"
              @albumChange="albumChange(index)"
              @albumDel="albumDel(index)"
              @openAlbumModel="openAlbumModel({ item, index })"
            ></album-item>
          </ul>
        </el-aside>

        <!-- 主内容 -->
        <el-main
          style="position: absolute; top: 60px; bottom: 60px; left: 200px; right: 0; "
        >
          <!-- 图片 -->
          <el-row :gutter="10">
            <el-col
              :span="24"
              :lg="4"
              :md="6"
              :sm="8"
              v-for="(item, index) in imageList"
              :key="index"
            >
              <div class="dateRange">
                <el-badge
                  :value="item.checkOrder"
                  :hidden="!item.isCheck"
                  :max="30"
                  class=""
                >
                  <el-card
                    class="box-card mb-3 position-relative"
                    :body-style="{ padding: 0 }"
                    shadow="hover"
                    style="cursor: pointer;"
                  >
                    <div
                      class="border"
                      :class="{ 'border-danger': item.isCheck }"
                    >
                      <img
                        :src="item.url"
                        class="w-100"
                        style=" height: 100px;"
                        @click="imageChoosed(item)"
                      />
                      <!-- 蒙板 -->
                      <div
                        style="background: rgba(0, 0, 0, 0.25); margin-top: -25px;"
                        class="w-100 text-white position-absolute px-1"
                      >
                        <span class="small">{{ item.name }}</span>
                      </div>
                      <div class="p-2 text-center">
                        <el-button-group>
                          <el-button
                            icon="el-icon-view"
                            size="mini"
                            class="p-2"
                            @click.stop="previewImage(item)"
                          ></el-button>
                          <el-button
                            icon="el-icon-edit"
                            size="mini"
                            class="p-2"
                            @click.stop="imageEdit(item, index)"
                          ></el-button>
                          <el-button
                            icon="el-icon-delete"
                            size="mini"
                            class="p-2"
                            @click.stop="imageDel({ index, item })"
                          ></el-button>
                        </el-button-group>
                      </div>
                    </div>
                  </el-card>
                </el-badge>
              </div>
            </el-col>
          </el-row>
        </el-main>
      </el-container>

      <!-- 底部 -->
      <el-footer class="border-top d-flex align-items-center px-0">
        <div
          style="width: 200px; flex-shrink: 0"
          class="h-100 d-flex  align-items-center justify-content-center border-right"
        >
          <el-button-group>
            <el-button
              size="mini"
              :disabled="albumPage === 1"
              @click="changeAlbumPage('pre')"
              >上一页</el-button
            >
            <el-button
              size="mini"
              @click="changeAlbumPage('next')"
              :disabled="albumPage === Math.ceil(albumTotal / 10)"
              >下一页</el-button
            >
          </el-button-group>
        </div>
        <div style="flex: 1" class="px-2">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="currentPage"
            :page-sizes="pageSizes"
            :page-size="pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
          >
          </el-pagination>
        </div>
      </el-footer>
    </el-container>

    <!-- 修改相册 | 创建相册 模态框-->
    <el-dialog :title="albumForm.title" :visible.sync="albumModel">
      <el-form :model="albumForm" label-width="80px">
        <el-form-item label="相册名称">
          <el-input v-model="albumForm.name"></el-input>
        </el-form-item>
        <el-form-item label="相册排序">
          <el-input-number
            v-model="albumForm.order"
            :min="0"
            size="medium"
          ></el-input-number>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="albumModel = false">取 消</el-button>
        <el-button type="primary" @click="confirmAlbumModel">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 上传图片模态框 -->
    <el-dialog title="上传图片" :visible.sync="uploadModel" @close="__init">
      <div class="text-center">
        <el-upload
          class="upload-demo"
          action="/admin/image/upload"
          drag
          multiple
          :headers="{ token: $store.state.user.token }"
          :data="{ image_class_id: image_class_id }"
          name="img"
          :on-success="uploadSuccess"
        >
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
          <div class="el-upload__tip" slot="tip">
            只能上传jpg/png文件，且不超过500kb
          </div>
        </el-upload>
      </div>
      <div slot="footer" class="dialog-footer">
        <!-- <el-button>取 消</el-button> -->
        <el-button type="primary" @click="uploadModel = false">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 预览图片模态框 -->
    <el-dialog :visible.sync="previewModel" width="60vw">
      <div style="margin: -60px -20px -30px -20px;">
        <img class="w-100 h-100" :src="previewUrl" alt="" />
      </div>
    </el-dialog>
  </div>
</template>

<script>
import albumItem from "components/image/albumItem";
export default {
  name: "index",
  inject: ["layout"],
  data() {
    return {
      searchForm: {
        order: "dsc",
        keyWord: "",
      },
      albumIndex: 0, // 选中相册索引
      albumModel: false,
      albums: [], // 相册列表
      albumPage: 1, // 相册分页
      albumTotal: 0, // 相册列表总条数
      albumForm: {
        title: "",
        name: "",
        order: 0,
      },
      albumEditIndex: -1,
      uploadModel: false,
      previewModel: false,
      imageList: [], // 图片列表
      previewUrl: null,
      chooseList: [], // 选中图片
      currentPage: 1, // 图片列表选中页
      pageSize: 10, //每页默认显示数量
      pageSizes: [10, 20, 50, 100], // 每页显示条数的可选项
      total: 10, // 总条数
    };
  },
  components: {
    albumItem,
  },
  computed: {
    // 选中相册的id
    image_class_id() {
      let item = this.albums[this.albumIndex];
      if (item) {
        return item.id;
      }
      return 0;
    },

    // 选中相册的图片列表url
    getImageListUrl() {
      // '?limit=[:limit]&order=[:order]&keyword=[:keyword]'
      let other = "";
      if (this.searchForm.keyWord != "") {
        other = `&keyword=${this.searchForm.keyWord}`;
      }
      return `/admin/imageclass/${this.image_class_id}/image/${this.currentPage}?limit=${this.pageSize}&order=${this.searchForm.order}${other}`;
    },
  },
  created() {
    this.__init();
  },
  methods: {
    // 获取对应相册下的图片列表
    getImageList() {
      this.layout.showLoading();
      setTimeout(() => {
        this.axios
          .get(this.getImageListUrl, {
            token: true,
          })
          .then((res) => {
            let result = res.data.data;
            this.imageList = result.list.map((item) => {
              return {
                id: item.id,
                url: item.url,
                name: item.name,
                isCheck: false,
                checkOrder: 0,
              };
            });
            this.total = result.totalCount;
            // this.total = 20;
            // for (let i = 1; i <= 20; i++) {
            //   this.imageList.push({
            //     id: i,
            //     url:
            //       "https://dss0.bdstatic.com/k4oZeXSm1A5BphGlnYG/skin/879.jpg?2",
            //     name: "图片" + i,
            //     isCheck: false,
            //     checkOrder: 0,
            //   });
            // }
            this.layout.hideLoading();
            // console.log(res);
          })
          .catch((err) => {
            this.layout.hideLoading();
          });
      }, 400);
    },

    __init() {
      // 获取相册列表
      this.layout.showLoading();
      this.axios
        .get("/admin/imageclass/" + this.albumPage, {
          token: true,
        })
        .then((res) => {
          let result = res.data.data;
          this.albums = result.list;
          this.albumTotal = result.totalCount;
          // 获取选中相册第一页图片列表
          this.getImageList();
        })
        .catch((err) => {
          this.layout.hideLoading();
        });

      /* for (let i = 1; i <= 10; i++) {
        this.albums.push({
          name: "相册" + i,
          num: Math.floor(Math.random() * 100),
          order: 0,
        });
      } */
    },

    // 切换相册
    albumChange(index) {
      this.albumIndex = index;
      this.getImageList();
    },

    // 删除相册
    albumDel(index) {
      this.$confirm("是否永久删除该相册?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          let id = this.albums[index].id;
          this.layout.showLoading();
          this.axios
            .delete("/admin/imageclass/" + id, {
              token: true,
            })
            .then((res) => {
              this.$message({
                type: "success",
                message: "删除成功!",
              });
              this.__init();
              this.layout.hideLoading();
            })
            .catch((err) => {
              this.layout.hideLoading();
            });

          // 删除相册
          // this.albums.splice(index, 1);
        })
        .catch(() => {});
    },

    // 打开相册 修改|创建 模态框
    openAlbumModel(obj) {
      // 修改
      if (obj) {
        let { item, index } = obj;
        // 初始化表单
        this.albumForm.title = "修改相册";
        this.albumForm.name = item.name;
        this.albumForm.order = item.order;
        this.albumEditIndex = index;
        // 打开模态框
        return (this.albumModel = true);
      } else {
        // 创建
        this.albumForm = {
          title: "创建相册",
          name: "",
          order: 0,
        };
        this.albumEditIndex = -1;
        this.albumModel = true;
      }
    },

    // 点击模态框按钮创建或修改相册
    confirmAlbumModel() {
      // 判断是否为修改  albumEditIndex > -1 为修改相册
      if (this.albumEditIndex > -1) {
        this.albumEdit();
        return (this.albumModel = false);
      } else {
        // 创建相册
        this.layout.showLoading();
        this.axios
          .post("/admin/imageclass", this.albumForm, {
            token: true,
          })
          .then((res) => {
            // 隐藏表单
            this.albumModel = false;
            this.layout.hideLoading();
            this.$message({
              type: "success",
              message: "创建成功！",
            });
            this.__init();
          })
          .catch((err) => {
            this.layout.hideLoading();
          });
        // this.albums.unshift({
        //   name: this.albumForm.name,
        //   order: this.albumForm.order,
        //   num: 0,
        // });
        // this.albumModel = false;
      }
    },

    // 修改相册
    albumEdit() {
      let item = this.albums[this.albumEditIndex];
      this.axios
        .post("/admin/imageclass/" + item.id, this.albumForm, {
          token: true,
        })
        .then((res) => {
          this.$message({
            type: "success",
            message: "修改成功！",
          });
          this.layout.hideLoading();
          this.__init();
        })
        .catch((err) => {
          this.layout.hideLoading();
        });
      // item.name = this.albumForm.name;
      // item.order = this.albumForm.order;
    },

    // 发开上传图片的模态框
    openUploadModel() {
      this.uploadModel = true;
    },

    // 预览大图
    previewImage(item) {
      this.previewUrl = item.url;
      this.previewModel = true;
    },

    // 修改图片名称
    imageEdit(item, index) {
      this.$prompt("请输入图片的新名称", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        inputValue: item.name,
        // 校验函数
        inputValidator(val) {
          if (val === "") {
            return "图片名不能为空";
          }
        },
      })
        .then(({ value }) => {
          // item.name = value;
          this.layout.showLoading();
          this.axios
            .post(
              "/admin/image/" + item.id,
              { name: value },
              {
                token: true,
              }
            )
            .then((res) => {
              this.$message({
                type: "success",
                message: "修改成功！",
              });
              this.getImageList();
              this.layout.hideLoading();
            })
            .catch((err) => {
              this.layout.hideLoading();
            });
        })
        .catch(() => {});
    },

    // // 删除图片
    // imageDel(index) {
    //   this.$confirm("是否删除该图片？", "提示", {
    //     confirmButtonText: "确定",
    //     cancelButtonText: "取消",
    //     type: "warning",
    //   }).then(() => {
    //     this.imageList.splice(index, 1);
    //     this.$message({
    //       type: "success",
    //       message: "删除成功！",
    //     });
    //   });
    // },
    // 删除图片
    imageDel(obj) {
      this.$confirm(
        obj.all ? "是否删除选中图片？" : "是否删除选该图片？",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.layout.showLoading();
        if (obj.all) {
          // 批量删除
          // let list = this.imageList.filter((img) => {
          //   return !this.chooseList.some((c) => c.id === img.id);
          // });
          // this.imageList = list;
          // this.chooseList = [];
          let ids = this.chooseList.map((item) => item.id);
          this.axios
            .post(
              "/admin/image/delete_all",
              { ids: ids },
              {
                token: true,
              }
            )
            .then((res) => {
              this.$message({
                type: "success",
                message: "删除成功！",
              });
              this.__init();
              this.chooseList = [];
              this.layout.hideLoading();
            })
            .catch((err) => {
              this.layout.hideLoading();
            });
        } else {
          // 删除单张图片
          // this.imageList.splice(obj.index, 1);
          this.axios
            .delete("/admin/image/" + obj.item.id, {
              token: true,
            })
            .then((res) => {
              this.$message({
                type: "success",
                message: "删除成功！",
              });
              this.__init();
              this.layout.hideLoading();
            })
            .catch((err) => {
              this.layout.hideLoading();
            });
        }
      });
    },

    // 选中图片
    imageChoosed(item) {
      if (!item.isCheck) {
        // 将选中照片加入到相应的数组中
        this.chooseList.push({ id: item.id, url: item.url });
        // 计算选中的序号
        item.checkOrder = this.chooseList.length;
        // 修改照片状态
        item.isCheck = true;
      } else {
        // 找到当前被点击照片在chooseList中的索引，删除
        let n = this.chooseList.findIndex((v) => v.id === item.id);
        if (n === -1) {
          return;
        } else {
          // 重新就算序号
          let length = this.chooseList.length;
          if (n + 1 < length) {
            for (let i = n; i < length; i++) {
              let no = this.imageList.findIndex(
                (v) => v.id === this.chooseList[i].id
              );
              if (no > -1) {
                this.imageList[no].checkOrder--;
              }
            }
          }
        }
        // 删除
        this.chooseList.splice(n, 1);
        // 修改状态、重置序号
        item.isCheck = false;
        item.checkOrder = 0;
      }
    },

    // 取消选中
    unChoose() {
      // 找到所有imageList中选中的图片
      this.imageList.forEach((img) => {
        let i = this.chooseList.findIndex((c) => c.id === img.id);
        if (i > -1) {
          // 取消选中状态，排序归零
          img.isCheck = false;
          img.checkOrder = 0;
          this.chooseList.splice(i, 1);
        }
      });
    },

    // 批量删除
    /* imageDelAll() {
      this.$confirm("是否删除选中的所有图片？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        // for (let i = 0; i < this.chooseList.length; i++) {
        //   let x = this.imageList.findIndex(
        //     (v) => v.id === this.chooseList[i].id
        //   );
        //   this.imageList.splice(x, 1);
        // }
        let list = this.imageList.filter((img) => {
          return !this.chooseList.some((c) => c.id === img.id);
        });
        this.imageList = list;
        this.chooseList = [];
        this.$message({
          type: "success",
          message: "删除成功！",
        });
      });
    },
   */

    // 分页，每页条数 console.log(`每页 ${val} 条`);
    handleSizeChange(val) {
      this.pageSize = val;
      this.getImageList();
    },

    // 分页 当前页 console.log(`当前页: ${val}`);
    handleCurrentChange(val) {
      this.currentPage = val;
      this.getImageList();
    },

    // 相册分页功能
    changeAlbumPage(type) {
      if (type === "pre") {
        this.albumPage--;
      } else {
        this.albumPage++;
      }
    },

    // 图片上传成功后的钩子
    uploadSuccess(response, file, fileList) {
      // console.log(response, file, fileList);
    },
  },
};
</script>
<style scoped>
.dateRange >>> .el-badge__content {
  margin: 3px 6px;
  z-index: 1;
}
</style>
