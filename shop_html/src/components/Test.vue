<template>
    <div>
      <h1>hello world</h1>

      <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>


      <h1> 懒加载 tree</h1>
      <el-tree  :props="defaultProps2"
                :load="loadNode"
                show-checkbox lazy >

      </el-tree>

    </div>
</template>

<script>
    export default {
        name: "Test",
      data() {
        return {
          data: [{
            label: '一级 1',
            children: [{
              label: '二级 1-1',
              children: [{
                label: '三级 1-1-1'
              }]
            }]
          }, {
            label: '一级 2',
            children: [{
              label: '二级 2-1',
              children: [{
                label: '三级 2-1-1'
              }]
            }, {
              label: '二级 2-2',
              children: [{
                label: '三级 2-2-1'
              }]
            }]
          }, {
            label: '一级 3',
            children: [{
              label: '二级 3-1',
              children: [{
                label: '三级 3-1-1'
              }]
            }, {
              label: '二级 3-2',
              children: [{
                label: '三级 3-2-1'
              }]
            }]
          }],
          defaultProps: {
            children: 'children',
            label: 'label'
          },defaultProps2: {
            children: 'children',
            label: 'name'
          }
        };
      },
      methods: {
        handleNodeClick(data) {
          console.log(data);
        } ,loadNode(node, resolve) {
          if (node.level === 0) {
            return resolve([{ name: 'region1' }, { name: 'region2' }]);
          }
          if (node.level > 3) return resolve([]);

          var hasChild;
          if (node.data.name === 'region1') {
            hasChild = true;
          } else if (node.data.name === 'region2') {
            hasChild = false;
          } else {
            hasChild = Math.random() > 0.5;
          }

          setTimeout(() => {
            var data;
            if (hasChild) {
              data = [{
                name: 'zone' + this.count++
              }, {
                name: 'zone' + this.count++
              }];
            } else {
              data = [];
            }

            resolve(data);
          }, 500);
        }
      }
    }
</script>

<style scoped>

</style>
