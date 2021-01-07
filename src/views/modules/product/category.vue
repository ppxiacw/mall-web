<template>
  <div>
    <el-tree
      :props="props"
      node-key="catId"
      :data="data"
      show-checkbox
      :default-expanded-keys="expandedKeys"
      @check-change="handleCheckChange"
      :expand-on-click-node="false"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button type="text" v-if="node.level!=3" size="mini" @click="() => showDialog(data)">Append</el-button>
          <el-button
            type="text"
            v-if="node.childNodes.length == 0"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog title="收货地址" :visible.sync="dialogFormVisible">
      <el-form :model="category">
        <el-form-item label="活动名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="append">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>



<script>
export default {
  data() {
    return {
      data: [],
      category: { name: "", parentId: 0, catLevel: 0, showStatus: 1, sort: 0 },
      expandedKeys: [],
      dialogFormVisible:false,
      props: {
        label: "name",
        children: "children"
      },
      count: 1
    };
  },
  created() {
    this.getCategory();
  },
  methods: {
    showDialog(data){
      console.log(data);
      
      this.dialogFormVisible = true
         this.category.catLevel = +data.catLevel+1
      this.category.parentCid = data.catId
    },
    append() {
      if (this.category.name.trim()=='') {
        this.$message.error('不能为空')
        return
      }
      this.category.name = this.category.name
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
       this.$message.success('添加chengg')
       this.dialogFormVisible=false
       this.getCategory()
       this.expandedKeys = [this.category.parentCid];
      });
    },

    remove(node, data) {
      this.$confirm(`是否删除【${data.name}】？', '确认信息`, {
        distinguishCancelAndClose: true,
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          let ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              type: "info",
              message: "删除成功"
            });
            this.getCategory();
            this.expandedKeys = [node.parent.data.catId];
          });
        })
        .catch(action => {
          this.$message({
            type: "info",
            message:
              action === "cancel" ? "放弃保存并离开页面" : "停留在当前页面"
          });
        });
    },
    getCategory() {
      this.$http({
        url: this.$http.adornUrl(`/product/category/list/tree/`),
        method: "get",
        params: this.$http.adornParams()
      }).then(data => {
        {
          this.data = data.data.data;
          console.log(data.length);
        }
      });
    },
    handleCheckChange(data, checked, indeterminate) {
      console.log(data, checked, indeterminate);
    },
    handleNodeClick(data) {
      console.log(data);
    },
    loadNode(node, resolve) {
      if (node.level === 0) {
        return resolve([{ name: "region1" }, { name: "region2" }]);
      }
      if (node.level > 3) return resolve([]);

      var hasChild;
      if (node.data.name === "region1") {
        hasChild = true;
      } else if (node.data.name === "region2") {
        hasChild = false;
      } else {
        hasChild = Math.random() > 0.5;
      }

      setTimeout(() => {
        var data;
        if (hasChild) {
          data = [
            {
              name: "zone" + this.count++
            },
            {
              name: "zone" + this.count++
            }
          ];
        } else {
          data = [];
        }

        resolve(data);
      }, 500);
    }
  }
};
</script>