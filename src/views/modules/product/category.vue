<template>
  <div class="pd-category">
    <el-switch
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
      v-model="draggable"
    ></el-switch>
    <el-button @click="batchSaveHandle" v-if="draggable">批量保存</el-button>
    <el-button @click="batchDeleteHandle" type="danger" size="mini"
      >批量删除</el-button
    >
    <el-tree
      :draggable="draggable"
      ref="menusTree"
      :data="mens"
      show-checkbox
      node-key="catId"
      :expand-on-click-node="false"
      :props="mensProps"
      :default-checked-keys="expandKey"
      :allow-drag="allowDragHandle"
      :node-drag="nodeDrayEndHandle"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="appendHandle(data)"
          >
            添加
          </el-button>
          <el-button type="text" size="mini" @click="editHandle(data)">
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="removeHandle(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="addMenusVisible"
      :before-close="clearFromHandle"
    >
      <el-form
        :model="fromMenus"
        :rules="menuRules"
        ref="menusFrom"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="name">
          <el-input
            v-model="fromMenus.name"
            placeholder="请输入分类名称"
            clearable
            size="mini"
          ></el-input>
        </el-form-item>
        <el-form-item label="请输入计量单位" prop="productUnit">
          <el-input
            v-model="fromMenus.productUnit"
            placeholder="请输入计量单位"
            clearable
            size="mini"
          ></el-input>
        </el-form-item>
        <el-form-item label="菜单图标" prop="icon">
          <el-row>
            <el-col :span="22">
              <el-popover
                ref="iconListPopover"
                placement="bottom-start"
                trigger="click"
                popper-class="mod-menu__icon-popover"
              >
                <div class="mod-menu__icon-inner">
                  <div class="mod-menu__icon-list">
                    <el-button
                      v-for="(item, index) in iconList"
                      :key="index"
                      @click="iconActiveHandle(item)"
                      :class="{ 'is-active': item === fromMenus.icon }"
                    >
                      <icon-svg :name="item"></icon-svg>
                    </el-button>
                  </div>
                </div>
              </el-popover>
              <el-input
                v-model="fromMenus.icon"
                v-popover:iconListPopover
                :readonly="true"
                placeholder="菜单图标名称"
                class="icon-list__input"
              ></el-input>
            </el-col>
            <el-col :span="2" class="icon-list__tips">
              <el-tooltip placement="top" effect="light">
                <div slot="content">
                  全站推荐使用SVG Sprite, 详细请参考:<a
                    href="//github.com/daxiongYang/renren-fast-vue/blob/master/src/icons/index.js"
                    target="_blank"
                    >icons/index.js</a
                  >描述
                </div>
                <i class="el-icon-warning"></i>
              </el-tooltip>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="  排序号" prop="sort">
          <el-input-number
            size="mini"
            v-model="fromMenus.sort"
            controls-position="right"
            :min="0"
            label="排序号"
          ></el-input-number>
        </el-form-item>
        <el-form-item label="是否同源" v-if="isEditFrom">
          <el-checkbox-group v-model="categoryType">
            <el-checkbox label="是" name="is"></el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item label="标识" prop="catId" v-show="false"><el-input v-model="fromMenus.catId" type="hidden"></el-input></el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addMenusVisible = false">取 消</el-button>
        <el-button type="primary" @click="addMenusHandle('menusFrom')"
          >确 定</el-button
        >
      </div>
    </el-dialog>

    <section v-if="mens.length == 0" id="pd-not-category-from">
      <el-form
        :model="fromMenus"
        :rules="menuRules"
        ref="menusFrom"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="name">
          <el-input
            v-model="fromMenus.name"
            placeholder="请输入分类名称"
            clearable
            size="mini"
          ></el-input>
        </el-form-item>
        <el-form-item label="请输入计量单位" prop="productUnit">
          <el-input
            v-model="fromMenus.productUnit"
            placeholder="请输入计量单位"
            clearable
            size="mini"
          ></el-input>
        </el-form-item>
        <el-form-item label="菜单图标" prop="icon">
          <el-row>
            <el-col :span="22">
              <el-popover
                ref="iconListPopover"
                placement="bottom-start"
                trigger="click"
                popper-class="mod-menu__icon-popover"
              >
                <div class="mod-menu__icon-inner">
                  <div class="mod-menu__icon-list">
                    <el-button
                      v-for="(item, index) in iconList"
                      :key="index"
                      @click="iconActiveHandle(item)"
                      :class="{ 'is-active': item === fromMenus.icon }"
                    >
                      <icon-svg :name="item"></icon-svg>
                    </el-button>
                  </div>
                </div>
              </el-popover>
              <el-input
                v-model="fromMenus.icon"
                v-popover:iconListPopover
                :readonly="true"
                placeholder="菜单图标名称"
                class="icon-list__input"
              ></el-input>
            </el-col>
            <el-col :span="2" class="icon-list__tips">
              <el-tooltip placement="top" effect="light">
                <div slot="content">
                  全站推荐使用SVG Sprite, 详细请参考:<a
                    href="//github.com/daxiongYang/renren-fast-vue/blob/master/src/icons/index.js"
                    target="_blank"
                    >icons/index.js</a
                  >描述
                </div>
                <i class="el-icon-warning"></i>
              </el-tooltip>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="  排序号" prop="sort">
          <el-input-number
            size="mini"
            v-model="fromMenus.sort"
            controls-position="right"
            :min="0"
            label="排序号"
          ></el-input-number>
        </el-form-item>
        <el-button @click="addMenusHandle" size="mini">提交</el-button>
        <el-button @click="resetFormHandle" size="mini">重置</el-button>
      </el-form>
    </section>
  </div>
</template>

<script>
import Icon from "@/icons";
export default {
  name: "category",
  data() {
    return {
      draggable: false,
      maxLevel: 0,
      iconList: [],
      tempCategory: {},
      categoryType: false,
      title: "",
      updateNodes: [],
      fromMenus: {
        name: "",
        catId: null,
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        icon: "",
        productUnit: 0,
        sort: 0,
      },
      menuRules: {
        name: [{ required: true, message: "请输入分类名称", trigger: "blur" }],
      },
      addMenusVisible: false,
      mens: [],
      expandKey: [],
      mensProps: {
        children: "children",
        label: "name",
      },
    };
  },
  computed: {
    isEditFrom() {
      return this.fromMenus.catId == null
    }
  },
  methods: {
    // 批量删除
    batchDeleteHandle() {
      let selectNode = this.$refs.menusTree.getCheckedNodes();
      let _menuIds = [];
      selectNode.forEach((item) => {
        _menuIds.push(item.data.catId);
      });
      this.$confirm(`是否确定【${_menuIds}】删除?`, "删除提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.delteCategoryByIds(_menuIds, selectNode);
        })
        .catch(() => {
          this.$message({ type: "info", message: "删除取消" });
        });
    },
    // 拖拽完成统一更新数据
    batchSaveHandle() {
      this.updateChildNode();
    },
    // 获取节点拖拽后的结果
    nodeDrayEndHandle(currentNode, endNode, position, nodeEvent) {
      let pCid = 0;
      let level = 0;
      let sort = 0;
      let siblings = null;
      if (position == "befroe" || position == "after") {
        pCid =
          endNode.parent.data.catId == undefined
            ? 0
            : endNode.parent.data.catId;
        siblings = endNode.parent.childrenNodes;
      } else {
        pCid = endNode.data.catId;
        siblings = endNode.childrenNodes;
      }

      let index = 0;
      siblings.forEach((item) => {
        if (item.catId == currentNode.data.catId) {
          let catLevel = currentNode.level;
          if (item.level != catLevel) {
            catLevel = item.level;
          }
          this.updateNodes.push({
            catId: item.catid,
            sort: index,
            parentCid: pCid,
            catLevle: oldLevel,
          });
          this.updateChangeChildNodeLevelHandle(item);
        } else {
          this.updateNodes.push({
            catId: item.catid,
            sort: index,
            parentCid: pCid,
          });
        }
        index++;
      });
      this.tempCategory.parentCid = pCid;
    },
    // 更新子节点的层级
    updateChangeChildNodeLevelHandle(parenNode) {
      parenNode.childNodes.forEach((item) => {
        item.data.catLevel = item.level;
        this.updateNodes({ catId: item.catId, catLevel: item.level });
        this.updateChangeChildNodeLevelHandle(item);
      });
    },
    // 判断节点是否可以拖拽到某个节点
    allowDragHandle(currentNode, dropNode, type) {
      this.countModeLevel(currentNode);
      if (type == "inner") {
        return (
          Math.abs(this.maxLevel - currentNode.level + 1 + dropNode.level) <= 3
        );
      } else {
        return (
          Math.abs(
            this.maxLevel - currentNode.data.level + 1 + dropNode.parent.level
          ) <= 3
        );
      }
    },
    // 统计节点的最高层级
    countModeLevel(node) {
      if (node.childrenNodes.length != 0 && noe.childrenNodes != null) {
        node.childrenNodes.forEach((item) => {
          if (item.level > this.maxLevel) this.maxLevel = item.catLevel;
          this.countModeLevel(item);
        });
      }
    },
    // 图标选中
    iconActiveHandle(iconName) {
      this.fromMenus.icon = iconName;
    },
    // 重新获取分类数据
    reloadGetCategory() {
      this.showDialogHandle();
      this.getCategoryList();
      this.tempCategory.parentCid == undefined
        ? ""
        : (this.expandKey[0] = this.tempCategory.parentCid);
      this.maxLevel != 0 ? (this.maxLevel = 0) : "";
      this.updateNodes.length != 0 ? (this.updateNodes = []) : "";
    },
    // 关闭对话框的时候重置表单
    clearFromHandle(done) {
      this.resetFormHandle();
      done();
    },
    // 显示弹框
    showDialogHandle() {
      if (this.mens.length != 0) {
        this.addMenusVisible = !this.addMenusVisible;
      }
    },
    // 添加节点
    appendHandle(data) {
      this.tempCategory = data;
      this.title = "添加节点";
      this.showDialogHandle();
    },
    // 删除节点
    removeHandle(node, data) {
      let ids = [data.catId];
      this.$confirm(`是否确定【${node.label}】删除?`, "删除提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.delteCategoryByIds(ids, [node]);
        })
        .catch(() => {
          this.$message({ type: "info", message: "删除取消" });
        });
    },

    // 修改分类
    editHandle(parame) {
      this.title = "修改节点";
      this.getCategoryInfoById(parame)
        .then(({data}) => {
          console.log(data)
          let {
            name,
            catId,
            icon,
            productUnit,
            catLevel,
            sort,
          } = data.category;
          this.tempCategory = data.category;
          this.fromMenus.name = name;
          this.fromMenus.catId = catId;
          this.fromMenus.icon = icon;
          this.fromMenus.sort = sort;
          this.fromMenus.productUnit = productUnit;
        })
        .finally(() => {
          this.showDialogHandle();
        });
    },
    // 添加分类
    addMenusHandle(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          if (
            this.fromMenus.catId != null ||
            this.fromMenus.catId != "" ||
            this.fromMenus.catId != undefined
          ) {
            this._edit();
          } else {
            this._add();
          }
        } else {
          return false;
        }
      });
    },
    // add
    _add(data = null) {
      let _parentCid;
      let _catLevel;
      if (this.categoryType) {
        _parentCid = this.tempCategory.parentCid;
        _catLevel = this.tempCategory.catLevel;
      } else {
        _parentCid = this.tempCategory.catId;
        _catLevel = this.tempCategory.catLevel + 1;
      }
      this.fromMenus.parentCid = _parentCid == undefined ? 0 : _parentCid;
      this.fromMenus.catLevel = _catLevel == undefined ? 1 : _catLevel;
      this.saveCategory();
    },
    // edit
    _edit(data = null) {
      this.fromMenus.parentCid = null;
      this.fromMenus.catLevel = null;
      this.fromMenus.showStatus = null;
      this.updateCategory();
    },

    // 重置表单
    resetFormHandle(formName = "menusFrom") {
      this.$refs[formName].resetFields();
    },
    //===================http=========
    // 获取分类数据
    getCategoryList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.mens = data.categoryTree;
        console.log(data);
      });
    },
    // 删除分类
    delteCategoryByIds(menIds, node) {
      this.$http({
        url: this.$http.adornUrl("/product/category/delete"),
        method: "post",
        data: this.$http.adornData(menIds, false),
      }).then(({ data }) => {
        if (data.code == 0) {
          node.forEach((itemx) => {
            let _tempParent = itemx.parent;
            let _tempCihldNodeData =
              _tempParent.data.children || _tempParent.data;
            menIds.forEach((itemy) => {
              let _dIndex = _tempCihldNodeData.findIndex(
                (itemz) => (itemz.catId = itemy)
              );
              _tempCihldNodeData.splice(_dIndex, 1);
            });
          });

          // // 获取当前节点的父节点
          // const parent = node.parent;
          // // 获取所有子节点
          // const children = parent.data.children || parent.data;
          // menIds.forEach((itemx) => {
          //   let dIndex = children.findIndex((itemy) => itemy.catId == itemx);
          //   children.splice(dIndex, 1);
          // });
          this.$message({
            type: "success",
            message: "菜单删除成功",
          });
        } else {
          this.$message({
            type: "error",
            message: "删除出错",
          });
        }
      });
    },
    // 保存分类
    saveCategory(parame = null) {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.fromMenus, false),
      }).then(({ data }) => {
        if (data.code == 0) {
          this.$message({
            type: "success",
            message: "添加成功",
          });
          this.reloadGetCategory();
        }
      });
    },
    // 更新分类
    updateCategory(parame = null) {
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(this.fromMenus, false),
      }).then(({ data }) => {
        if (data.code == 0) {
          this.$message({
            type: "success",
            message: "修改成功",
          });
          this.reloadGetCategory();
        }
      });
    },
    // 更新拖拽后的字点的数据
    updateChildNode() {
      this.$http({
        url: this.$http.adornUrl("/product/category/updateSort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        if (data.code == 0) {
          this.$message({
            type: "success",
            message: "节点更新成功",
          });
          this.reloadGetCategory();
        }
      });
    },
    //更具{ catId} 获取对应分类的数据
    getCategoryInfoById(data) {
      return this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        params: this.$http.adornParams({}),
      });
    },
  },
  components: {},

  created() {
    this.getCategoryList();
    this.iconList = Icon.getNameList();
  },
};
</script>

<style lang="less" scoped>
</style>
