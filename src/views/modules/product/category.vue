<template>
  <div>
    <el-tree :props="props" :data="data"  show-checkbox @check-change="handleCheckChange"></el-tree>
  </div>
</template>



<script>
export default {
  data() {
    return {
        data:[],
      props: {
        label: "name",
        children: "children"
      },
      count: 1
    };
  },
  created() {
      this.getCategory()
  },
  methods: {
    getCategory(){
        this.$http({
              url: this.$http.adornUrl(`/product/category/list/tree/`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(data=>{
                {
                this.data =data.data.data
                console.log(data.length);
                
            }
            })
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