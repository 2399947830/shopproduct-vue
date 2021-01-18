<template>
  <div>
    <h1>商品属性列表</h1>
    <!-- 商品属性 展示列表-->
    <div>
      <div class="container">
        <div class="handle-box">
          <el-button
            type="primary"
            icon="el-icon-circle-plus"
            class="handle-del mr10"
            @click="addAttr"
          >属性新增
          </el-button>
          <el-input v-model="query.name" placeholder="属性名称" class="handle-input mr10"></el-input>
          <el-button type="primary" icon="el-icon-search" @click="handleSearch">查询</el-button>
        </div>
        <el-table
          :data="tableData"
          border
          class="table"
          header-cell-class-name="table-header"
        >
          <el-table-column
            prop="id"
            label="编号"
            align="center">
          </el-table-column>

          <el-table-column
            prop="name"
            label="英文名称"
            align="center">
          </el-table-column>

          <el-table-column
            prop="nameCH"
            label="中文名称"
            align="center">
          </el-table-column>

          <el-table-column
            prop="type"
            label="类型框"
            :formatter="formatType"
            align="center">
          </el-table-column>

          <el-table-column
            prop="isSKU"
            label="是否是SKU"
            :formatter="formatIsSKU"
            align="center">
          </el-table-column>

          <el-table-column
            prop="isdel"
            label="是否展示"
            :formatter="formatIsdel"
            align="center">
          </el-table-column>

          <el-table-column
            prop="createDate"
            label="创建时间"
            align="center"
          ></el-table-column>

          <el-table-column
            prop="updateDate"
            label="修改时间"
            align="center"
          ></el-table-column>

          <el-table-column
            prop="author"
            label="操作人"
            align="center"
          ></el-table-column>

          <el-table-column label="操作" width="180" align="center">
            <template slot-scope="scope">
              <el-button type="text" icon="el-icon-edit"  @click="handleEdit( scope.row)">编辑</el-button>
              <el-button type="text" icon="el-icon-edit" v-if="scope.row.type!=3" @click="queryAttrValue( scope.row)">查看</el-button>
            </template>
          </el-table-column>
        </el-table>
        <!--分页-->
        <div class="pagination">
          <el-pagination
            background
            layout="total, sizes, prev, pager, next, jumper"
            :current-page="query.current"
            :page-size="query.size"
            :page-sizes="pageSizes"
            :total="pageTotal"
            @current-change="handlePageChange"
            @size-change="handleSizeChange"
          ></el-pagination>
        </div>
      </div>


      <!-- 新增弹出框 -->
      <el-dialog :visible.sync="editVisible" width="50%">
        <el-form ref="attrForm" :model="attrForm" label-width="140px">
          <el-form-item label="英文名称" prop="name">
            <el-input v-model="attrForm.name"></el-input>
          </el-form-item>
          <el-form-item label="中文名称" prop="nameCH">
            <el-input v-model="attrForm.nameCH"></el-input>
          </el-form-item>

          <el-form-item label="分类">
            <el-select v-model="attrForm.typeId" placeholder="请选择分类">
              <el-option label="请选择" :value="-1"></el-option>
              <el-option v-for="item in types" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="属性类型">
            <el-radio-group v-model="attrForm.type">
              <el-radio label="0">下拉框</el-radio>
              <el-radio label="1">单选框</el-radio>
              <el-radio label="2">复选框</el-radio>
              <el-radio label="3">输入框</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="是否SKU">
            <el-switch v-model="attrForm.isSKUb"></el-switch>
          </el-form-item>

        </el-form>
        <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible=false">取 消</el-button>
                <el-button type="primary" @click="saveAttr">确 定</el-button>
            </span>
      </el-dialog>

      <!-- 编辑弹出框 -->
      <el-dialog :visible.sync="addVisible" width="50%">
        <el-form ref="updateForm" :model="updateForm" label-width="140px">
          <el-form-item label="英文名称" prop="name">
            <el-input v-model="updateForm.name"></el-input>
          </el-form-item>
          <el-form-item label="中文名称" prop="nameCH">
            <el-input v-model="updateForm.nameCH"></el-input>
          </el-form-item>

          <el-form-item label="分类">
            <el-select v-model="updateForm.typeId" placeholder="请选择分类">
              <el-option label="请选择" :value="-1"></el-option>
              <el-option v-for="item in types" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="属性类型">
            <el-radio-group v-model="updateForm.type">
              <el-radio :label="0">下拉框</el-radio>
              <el-radio :label="1">单选框</el-radio>
              <el-radio :label="2">复选框</el-radio>
              <el-radio :label="3">输入框</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="是否SKU">
            <el-radio-group v-model="updateForm.isSKU">
              <el-radio :label="1">是</el-radio>
              <el-radio :label="2">否</el-radio>
            </el-radio-group>
          </el-form-item>

        </el-form>
        <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible=false">取 消</el-button>
                <el-button type="primary" @click="updateAttr">确 定</el-button>
            </span>
      </el-dialog>


      <!-- 查看属性值弹出框 -->
      <el-dialog :title="changeTitle" :visible.sync="queryVisible" width="50%">
        <el-button type="primary">新增属性值</el-button>
        <el-table
          :data="attrValueTableData"
          border
          class="table"
          header-cell-class-name="table-header"
        >
          <el-table-column
          prop="id"
          label="ID"
          align="center"
          ></el-table-column>

          <el-table-column
          prop="value"
          label="值的英文名称"
          align="center"
          ></el-table-column>

          <el-table-column
          prop="valueCH"
          label="值名称"
          align="center"
          ></el-table-column>

          <el-table-column
            prop="attrId"
            label="属性"
            align="center"
          ></el-table-column>

          <el-table-column
          prop="isdel"
          label="是否显示"
          :formatter="formatValueIsdel"
          align="center"
          ></el-table-column>

          <el-table-column label="操作" width="180" align="center">
            <template slot-scope="scope">
              <el-button type="text" icon="el-icon-edit"  @click="handleAttrValueEdit( scope.row)">编辑</el-button>
              <el-button type="text" icon="el-icon-edit"  @click="delAttrValue( scope.row)">删除</el-button>
            </template>
          </el-table-column>

        </el-table>

      </el-dialog>

      <!-- 属性值表的编辑弹框 -->
      <el-dialog title="维护属性值的表信息" :visible.sync="updateVisible" width="50%">
        <el-form ref="updateAttrValueForm" :model="updateAttrValueForm" label-width="140px">
          <el-form-item label="英文名称" prop="value">
            <el-input v-model="updateAttrValueForm.value"></el-input>
          </el-form-item>

          <el-form-item label="中文名称" prop="valueCH">
            <el-input v-model="updateAttrValueForm.valueCH"></el-input>
          </el-form-item>

          <el-form-item label="是否展示" prop="isdel">
            <el-radio-group v-model="updateAttrValueForm.isdel">
            <el-radio :label="1">展示</el-radio>
            <el-radio :label="0">不展示</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
                <el-button @click="updateVisible=false">取 消</el-button>
                <el-button type="primary" @click="updateAttrValue">确 定</el-button>
            </span>
      </el-dialog>

    </div>
  </div>
</template>

<script>
    export default {
        name: "Attr",
        data() {
            return {
              tableData:[],
              attrValueTableData:[],
              query: {
                name: '',
                size: 5,
                current: 1
              },
              pageSizes: [5, 10, 15, 20],
              pageTotal: 0,
              editVisible:false,
              addVisible:false,
              queryVisible:false,
              updateVisible:false,
              attrForm:{
                typeId:-1,
              },
              updateForm:{
                name:"",
                nameCH:"",
                typeId:null,
                type:"",
                isSKU:null
              },
              types:[
                {"id":7,name:"分类/电子产品/手机"},
                {"id":10,name:"分类/电子产品/电脑"},
                {"id":4,name:"分类/服装/上衣"},
                {"id":5,name:"分类/服装/裤子"}
                ],
              changeTitle:"",
              updateAttrValueForm:{
                value:"",
                valueCH:"",
                isdel:""
              }
            }
        },
        methods: {
          queryAttr:function(){
          this.$ajax.get("http://localhost:8082/api/brand/queryAllData?currPage="+this.query.current+"&size="+this.query.size+"&name="+this.query.name).then(result=>{
              console.log(result);
              this.tableData=result.data.data.list;
              this.pageTotal=result.data.data.count;
            }).catch(
              error=>console.log(error)
            )
          },
          //新增属性
          addAttr:function () {
            this.editVisible=true;
          },
          //提交新增内容
          saveAttr:function(){
            this.attrForm.isSKU=this.attrForm.isSKUb?1:0;
            this.$ajax.post("http://localhost:8082/api/brand/addBrand",this.$qs.stringify(this.attrForm)).then(result=>{
            this.editVisible=false;
            })
          },
          //条件查询
          handleSearch:function () {
            this.queryAttr();
          },
          //编辑
          handleEdit:function (row) {
            //alert(JSON.stringify(id.id));
            this.addVisible=true;
            this.$ajax.get("http://localhost:8082/api/brand/queryBrandById?id="+row.id).then(result=>{
              this.updateForm=result.data.data;
            }).catch(
              error=>console.log(error)
            )

          },
          //页数改变
          handlePageChange:function (val) {
            this.$set(this.query,'current',val)
            this.queryAttr();
          },
          //长度改变
          handleSizeChange:function () {
            this.$set(this.query, 'current', 1);
            this.$set(this.query, 'size', val);
            this.getData();
          },
          formatIsdel:function (a,b,c,d) {
              if (c == 1){
                return "展示"
              }else {
                return "不展示"
              }
          },
          formatIsSKU:function (a,b,c,d) {
              if (c == 1){
                return "是"
              }else{
                return "否"
              }
          },
          formatType:function (a,b,c,d) {
              if (c == 0){
                return "下拉框"
              }
              if (c == 1){
                return "单选框"
              }
              if (c == 2){
                return "复选框"
              }
              if (c == 3){
                return "输入框"
              }
          },
          //修改
          updateAttr:function () {
            this.$ajax.post("http://localhost:8082/api/brand/updateBrand",this.$qs.stringify(this.updateForm)).then(result=>{
            this.addVisible=false;
            }).catch(
              error=>console.log("属性修改报错")
            )
          },
          //弹框里面的编辑按钮
          handleAttrValueEdit:function (row) {
            this.updateVisible=true;
            this.$ajax.get("http://localhost:8082/api/attr/value/queryAttrValueById?id="+row.id).then(result=>{
              this.updateAttrValueForm=result.data.data;
              console.log(result);

            }).catch(
              error=>console.log("属性值回显报错")
            )

          },
          //主页面上 查看按钮
          queryAttrValue:function (row) {
            this.queryVisible=true;
            this.changeTitle=row.name+"属性值的选项信息";
            this.$ajax.get("http://localhost:8082/api/attr/value/queryAllData?attrId="+row.id).then(result=>{
              console.log(result.data);
              this.attrValueTableData = result.data.data.list;
            }).catch(
              error=>console.log(error)
            )
          },
          //属性值的删除
          delAttrValue:function (row) {
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              this.$ajax.post("http://localhost:8082/api/attr/value/deleteAttrValue?id="+row.id);
              this.$message({
                type: 'success',
                message: '删除成功!'
              });
            }).catch(() => {
              this.$message({
                type: 'info',
                message: '已取消删除'
              });
            });
          },
          //attrid转换是否显示
          formatValueIsdel:function (a,b,c,d) {
            if (c == 0){
              return "不显示";
            }
            if (c == 1){
              return "显示";
            }
          },
          updateAttrValue:function () {
            this.$ajax.post("http://localhost:8082/api/attr/value/updateAttrValue",this.$qs.stringify(this.updateAttrValueForm)).then(result=>{
            this.updateVisible=false;
            }).catch(
              error=>console.log("提交属性值的修改列表报错")
            )
          }
        },
        created() {
          this.queryAttr();
        }

    }
</script>

<style scoped>

</style>
