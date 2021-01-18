<template>
  <div>
      <div>
        <h1>商品分类管理</h1>
        <el-button type="success" @click="dialogFormVisible=true">新增</el-button>
        <el-tree
          :data="data"
          :props="defaultProps"
          accordion
        >
        </el-tree>
      </div>

    <!-- 表 单 -->
    <div >
      <el-dialog title="新增类型" :visible.sync="dialogFormVisible">
        <el-form :model="testForm">
          <el-form-item label="父节点类型" :label-width="formLabelWidth">
            <el-select v-model="testForm.name" placeholder="请选择节点">
                  <el-option
                    v-for="item in selectNode"
                    :key="item.id"
                    :value="item.value"
                    :label="item.id"
                  ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="节点名称" :label-width="formLabelWidth">
            <el-input  v-model="testForm.addNode" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
        </div>
      </el-dialog>
    </div>
    <!-- 树 -->
    <div class="block">
      <el-tree
        :data="data"
        show-checkbox
        node-key="id"
        default-expand-all
        :expand-on-click-node="false">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => dialogFormVisible=true">
            新增
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            删除
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() =>toUpdate(node,data)"
          >
            修改
          </el-button>
        </span>
      </span>
      </el-tree>
    </div>

  </div>
</template>

<script>
    export default {
        name: "Type",
      data(){
          return{
            data:[],//tree 的数据
            ajaxData:[],// 远程请求的数据
            jsonStr:"", //递归拼接处理
            defaultProps:{},
            dialogFormVisible:false,
            formLabelWidth: '120px',
            testForm:{
              productType:"",
              addNode:""
            },
            selectNode:[]
          }
      }
      ,methods:{
          chandleData:function(){ //ajaxData  处理成 data   //转换数据
            //找到顶层节点的数据
            for (let i = 0; i <this.ajaxData.length ; i++) {
                if(this.ajaxData[i].pid==0){
                  this.diguiNode(this.ajaxData[i]);
                  break;
                }
            }
            console.log(this.jsonStr);
            //将字符串 转为json对象
            this.data.push(JSON.parse(this.jsonStr));
          },  //  id  name  pid        id  name children []
          diguiNode:function (node) {
              // 判断是否为父节点
               var bf=this.isParent(node);
               if(bf==true){
                 //{"id":1,"label":"分类",}
                 //{"id":1,"label":"分类","children":[]}
                 this.jsonStr+='{"id":'+node.id+',"label":"'+node.name+'","children":[';
                 //拼接子节点
                 //查找此节点的子节点
                  let count=0
                 for (let i = 0; i <this.ajaxData.length ; i++) {
                   //判断是否为当前节点的子节点
                   if(node.id==this.ajaxData[i].pid){
                     count++;
                      this.diguiNode(this.ajaxData[i]);
                      this.jsonStr+=",";
                   }
                 }
                //处理多余的,号
                 if(count!=0){
                   this.jsonStr=this.jsonStr.substr(0,this.jsonStr.length-1);
                 }
                 //拼完整
                  this.jsonStr+=']}';
               }else{
                  this.jsonStr+='{"id":'+node.id+',"label":"'+node.name+'"}';
               }
          },
          isParent:function(node){ // 判断是否为父节点  pid 有没有指向当前id
            for (let i = 0; i <this.ajaxData.length ; i++) {
              if(node.id==this.ajaxData[i].pid){
                return true;
              }
            }
            console.log(node);
            return false;
          },
        //新增节点
        append(data) {
          const newChild = { id: id++, label: 'testtest', children: [] };
          if (!data.children) {
            this.$set(data, 'children', []);
          }
          data.children.push(newChild);
        },
        //删除节点
        remove(node, data) {
          const parent = node.parent;
          const children = parent.data.children || parent.data;
          const index = children.findIndex(d => d.id === data.id);
          children.splice(index, 1);
        },
        querySelectData:function () {
            this.$ajax.get("\"http://192.168.1.251:8080/api/type/getData").then(result=>{
            }).catch(
              err=>console.log(err)
            );
        }

      },
      created:function(){
          //远程请求数据
          this.$ajax.get("http://192.168.1.251:8080/api/type/getData").then(res=>{
            this.ajaxData=res.data.data;  // 把请求的数据  赋给全局
            this.chandleData();
          }).catch(err=>console.log(err));
          this.querySelectData();
      }
    }
</script>

<style scoped>

</style>
