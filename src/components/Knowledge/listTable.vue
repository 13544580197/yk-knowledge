<template>
<div class="app-container">
    <div class="filter-container addClass">
      <el-form :model="addQuery" class="demo-form-inline">
        <el-form-item label="名称">
          <el-input  style="width: 150px" v-model="addQuery.name" placeholder="请输入名称"></el-input>
        </el-form-item>
        <el-form-item label="命名空间" v-if="knowType == 1">
          <el-input  style="width: 150px" v-model="addQuery.namespace" placeholder="请输入内容"></el-input>
        </el-form-item>

        <el-form-item v-if="knowType == 0 || knowType == 2" label="值的数据类型">
        <!-- <el-select v-if="knowType != 1" clearable v-model="addQuery.valueType" placeholder="请选择值类型" class="filter-item searchUI" style="width: 150px">
          <el-option v-for="(item, index) in ValueTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
        </el-select> -->
        <el-select clearable v-model="addQuery.dataType" placeholder="请选择数据类型" class="filter-item searchUI" style="width: 180px">
          <el-option v-for="(item, index) in dataTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
        </el-select>
        </el-form-item>
        <el-form-item label="值" v-if="knowType == 0 || knowType == 1 || knowType == 2">
          <el-input v-if="addQuery.dataType == undefined || addQuery.dataType == 0 || addQuery.dataType == 1" style="width: 150px" v-model="addQuery.value" placeholder="请输入值"></el-input>
          <el-select v-if="addQuery.dataType == 2" v-model="addQuery.value" placeholder="请选择值" class="filter-item searchUI" style="width: 120px">
            <el-option v-for="(item, index) in [{id: '0', name: '否'}, {id: '1', name: '是'}]" :key="index" :label="item.name" :value="item.id"></el-option>
          </el-select>
          <el-date-picker
                v-if="addQuery.dataType == 4"
                v-model="addQuery.value"
                align="right"
                type="date"
                placeholder="选择日期"
                value-format="yyyy-MM-dd"
                :picker-options="pickerOptions1">
              </el-date-picker>
           <el-input
                v-if="addQuery.dataType == 5"
                type="textarea"
                :rows="2"
                placeholder="请输入内容"
                v-model="addQuery.value">
              </el-input>
        </el-form-item>
        <el-form-item v-if="knowType == 5" label="实体">
          <el-select v-model="entities" @change="changeEntitiesHandle" placeholder="请选择实体" class="filter-item searchUI" style="width: 150px">
            <el-option v-for="(item, index) in entitiesOption" :key="index" :label="item.name" :value="item.namespace+'\\'+item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item v-if="knowType == 5 && entities" label="方法">
          <el-select v-model="functionName" @change="changeFunctionHandle" placeholder="请选择方法" class="filter-item searchUI" style="width: 150px">
            <el-option v-for="(item, index) in funcitonOption" :key="index" :label="item.name" :value="item.name"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item v-if="knowType == 5 && describe" label="方法说明:" style="width:100%">

          <el-input type="textarea" v-model="describe" disabled autosize style="width: 18rem;"></el-input>
        </el-form-item>
        <el-form-item v-if="knowType == 5" label="添加动作参数"  :rules="[{required: true, fullField: '动作参数', trigger: 'blur'}]">
          <el-button
            type="primary"
            icon="el-icon-circle-plus"
            @click="addNewParam()"
            >
          </el-button>
          <el-row v-for="(param, index) in paramsArr" :key="index">
            <el-form-item label="名称">
              <el-input  style="width: 150px" v-model="param.name" placeholder="请输入参数名称"></el-input>
            </el-form-item>
            <el-form-item label="值的数据类型">
            <el-select clearable v-model="param.dataType" placeholder="请选择数据类型" class="filter-item searchUI" style="width: 150px">
              <el-option v-for="(item, index) in dataTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
            </el-select>
            </el-form-item>
            <el-form-item label="值" >
              <el-input v-if="param.dataType == undefined || param.dataType == 0 || param.dataType == 1" style="width: 150px" v-model="param.value" placeholder="请输入值"></el-input>
              <el-select v-if="param.dataType == 2" v-model="param.value" placeholder="请选择值" class="filter-item searchUI" style="width: 120px">
                <el-option v-for="(item, index) in [{id: '0', name: '否'}, {id: '1', name: '是'}]" :key="index" :label="item.name" :value="item.id"></el-option>
              </el-select>
              <el-date-picker
                    v-if="param.dataType == 4"
                    v-model="param.value"
                    align="right"
                    type="date"
                    placeholder="选择日期"
                    value-format="yyyy-MM-dd"
                    :picker-options="pickerOptions1">
                  </el-date-picker>
               <el-input
                    v-if="param.dataType == 5"
                    type="textarea"
                    :rows="2"
                    placeholder="请输入内容"
                    v-model="param.value">
                  </el-input>
            </el-form-item>
            </el-row>
        </el-form-item>
        <el-form-item>
          <el-button
            class="filter-item"
            style="margin-left: 10px;"
            type="primary"
            icon="el-icon-edit"
            @click="addHandle()"
            >添加</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div style="clear: both;"></div>
    <div class="filter-container">
    <!-- <el-select clearable v-model="listQuery.systemID" placeholder="请选择系统" class="filter-item searchUI" style="width: 150px">
      <el-option v-for="(item, index) in systemOption" :key="index" :label="item.name" :value="item.id"></el-option>
    </el-select> -->
    <el-select v-if="knowType != 1" clearable v-model="listQuery.dataType" placeholder="请选择数据类型" class="filter-item searchUI" style="width: 120px">
      <el-option v-for="(item, index) in dataTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
    </el-select>
    <el-input @keyup.enter.native="getList" style="width: 120px;" class="filter-item" placeholder="名称" v-model="listQuery.name" clearable>
    </el-input>
    <!-- <el-input v-if="knowType == 2" @keyup.enter.native="getList" style="width: 150px;" class="filter-item" placeholder="实例名称" v-model="listQuery.entityName"> </el-input>-->

    <el-button
      class="filter-item"
      type="primary"
      icon="el-icon-search"
      @click="getList()"
      >
      搜索
    </el-button>
  </div>

  <!-- v-loading="listLoading" -->
  <el-table
    v-loading="listLoading"
    :key="tableKey"
    :data="list"
    element-loading-text="给我一点时间"
    border
    fit
    highlight-current-row
    style="width: 100%"
    @row-click="getChildrenHandle"
    >
    <el-table-column align="center" label="ID" min-width="40">
      <template slot-scope="scope">
        <span>{{scope.row.id}}</span>
      </template>
    </el-table-column>

    <!-- <el-table-column align="center" label="父级ID" min-width="80" v-if="!(knowType == 5 && !isChildren)"> -->
    <!-- <el-table-column align="center" label="父级ID" min-width="80" v-if="!(knowType == 5 && !isChildren)">
      <template slot-scope="scope">
        <el-input v-if="scope.row.isEdit && knowType != 1" class="edit-input" size="small" type="number" v-model.number="scope.row.parentID" style="width=2rem;">            </el-input>
        <span v-else>{{scope.row.parentID}}</span>
      </template>
    </el-table-column> -->
    <el-table-column v-if="knowType == 0 || knowType == 2" align="center" label="数据类型">
      <template slot-scope="scope">
        <el-select v-if="scope.row.isEdit" clearable v-model="scope.row.dataType" placeholder="请选择数据类型" class="filter-item searchUI" style="width: 150px">
          <el-option v-for="(item, index) in dataTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <span v-else>{{scope.row.dataType | displayNameFilter([{id: 0, name: 'Integer'}, {id: 1, name: 'String'}, {id: 2, name: 'Boolean'}, {id: 4, name: 'Date'}, {id: 5, name: 'Object'}])}}</span>

      </template>

    </el-table-column>
    <el-table-column align="center" label="名称">
      <template slot-scope="scope">
        <el-input v-if="scope.row.isEdit" class="edit-input" size="small" type="text" v-model.number="scope.row.name" style="width=2rem;"></el-input>
        <span v-else>{{scope.row.name}}</span>
      </template>
    </el-table-column>
    <el-table-column align="center" label="命名空间" v-if="knowType == 1">
      <template slot-scope="scope">
        <el-input v-if="scope.row.isEdit" class="edit-input" size="small" type="text" v-model.number="scope.row.namespace" style="width=2rem;"></el-input>
        <span v-else>{{scope.row.namespace}}</span>
      </template>
    </el-table-column>
    <el-table-column v-if="knowType == 0 || knowType == 2" align="center" label="值类型">
      <template slot-scope="scope">
        <!-- <el-select v-if="scope.row.isEdit" clearable v-model="scope.row.valueType" placeholder="请选择值类型" class="filter-item searchUI" style="width: 150px">
          <el-option v-for="(item, index) in ValueTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <span v-else>{{scope.row.valueType | displayNameFilter([{id: 0, name: '主类型'}, {id: 1, name: '实体Field'}, {id: 2, name: '动作参数'}])}}</span> -->
        <span >{{scope.row.valueType | displayNameFilter([{id: 0, name: '主类型'}, {id: 1, name: '实体Field'}, {id: 2, name: '动作参数'}])}}</span>
      </template>

    </el-table-column>
    <el-table-column align="center" label="值">
      <template slot-scope="scope">
        <el-input v-if="scope.row.isEdit" class="edit-input" size="small" type="text" v-model.number="scope.row.value" style="width=2rem;"></el-input>
        <span v-else>{{scope.row.value}}</span>
      </template>
    </el-table-column>
    <!-- <el-table-column align="center" label="创建时间">
      <template slot-scope="scope">
        <span>{{scope.row.createDTStr}}</span>
      </template>
    </el-table-column>
    <el-table-column align="center" label="操作员">
      <template slot-scope="scope">
        <span>{{scope.row.opUserName}}</span>
      </template>
    </el-table-column> -->

    <el-table-column align="center" label="操作" min-width="80">
      <template slot-scope="scope">
        <div v-if="scope.row.isEdit">
          <el-button size="small" icon="el-icon-refresh" type="warning" @click.native.stop="cancelEdit(scope.row)">取消</el-button>
          <el-button type="success" @click.native.stop="confirmEdit(scope.row)" size="small" icon="el-icon-circle-check-outline">确认</el-button>
        </div>
        <div v-else>
        <el-button
          type="success"
          size="mini"
          @click.native.stop="editHandle(scope.row)">
          修改
        </el-button>
        <el-button style="background:red;color:#fff" @click.native.stop="deleteHandle(scope.row)" size="mini">删除</el-button>
        </div>
      </template>
    </el-table-column>
  </el-table>
  <div class="pagination-container">
    <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="listQuery.p" :page-sizes="[]" :page-size="listQuery.limit" layout="total" :total="total">
    </el-pagination>
  </div>

  </div>
</template>

<script>
import {displayNameFilter } from "@/filters";
// const defaultPostForm = {
//   sid: undefined,
//   name: undefined,
//   id: undefined,
//   status:0, //状态
// };
const defaultForm = {
  // systemID: undefined,
  parentID: undefined,
  dataType: undefined,  //值的数据类型
  name: undefined,
  value: undefined,
  namespace: undefined, //命名空间
  params: undefined, //动作的参数
};
export default {
  name: "knowledge-listTable",
  // components: { ActivityDetail },
  props: {
    knowType: {             //类型 0变量 1实体 2常量 3事件 4动作
      type: Number,
      default: 0
    },
    systemID: {             //系统id 0 御客餐饮  1供应链
      type: Number,
      default: 0
    },
    isChildren:{          //是否是子集（实体的字段）
      type: Boolean,
      default: false
    },
    list: {
      type: Array,
      default: []
    },
    entitiesOption: {  //动作实体下拉列表
      type: Array,
      default: []
    },
    funcitonOption: {  //动作实体下的方法
      type: Array,
      default: []
    },
    describe: {  //动作实体下方法的参数说明
      type: String,
      default: undefined
    },
    listLoading:{          //加载进度调
      type: Boolean,
      default: false
    },
    addSuccess:{          //添加成功
      type: Boolean,
      default: false
    },
  },
  mounted() {
      this.listQuery.systemID = this.systemID
      this.addQuery.systemID = this.systemID
  },
  data() {
    return {
      // list: [],                 // 知识库列表
      listQuery: {              // 知识库查询对象
        sid: undefined,
        limit: 2,
        getAll: 1,
        p: 1,
        systemID: undefined,
        dataType: undefined,
        name: undefined,
        entityName: undefined
      },
      addQuery: {              // 知识库添加对象
        systemID: undefined,
        parentID: undefined,
        dataType: undefined,  //值的数据类型
        name: undefined,
        value: undefined,
        namespace: undefined, //命名空间
        params: undefined, //动作的参数
      },
      total: undefined,
      tableKey: 0,
      systemOption: [
        {id: 0, name: '御客餐饮'},
        {id: 1, name: '御客外卖'},
        ],
      dataTypeOption: [
        {id: 0, name: 'Integer'},
        {id: 1, name: 'String'},
        {id: 2, name: 'Boolean'},
        {id: 3, name: 'Array'},
        {id: 4, name: 'Date'},
        {id: 5, name: 'Object'},
        ],
      ValueTypeOption: [ //值类型 0 主类型(变量, 实体, 常量, 事件, 动作等) 1 实体field 2 动作参数
        {id: 0, name: '主类型'},
        {id: 1, name: '实体field'},
        {id: 2, name: '动作参数'}
        ],
      pickerOptions1: {
            disabledDate(time) {
              return time.getTime() > Date.now();
            },
            shortcuts: [{
              text: '今天',
              onClick(picker) {
                picker.$emit('pick', new Date());
              }
            }, {
              text: '昨天',
              onClick(picker) {
                const date = new Date();
                date.setTime(date.getTime() - 3600 * 1000 * 24);
                picker.$emit('pick', date);
              }
            }, {
              text: '一周前',
              onClick(picker) {
                const date = new Date();
                date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
                picker.$emit('pick', date);
              }
            }]
      },

      paramsArr: [], //动作参数

      entities: undefined,  //动作的实体
      functionName: undefined,  //动作实体的方法
    }
  },
  watch: {
    knowType(val) {
      console.log('===knowType==',val);
    },
    addSuccess(val) {
      console.log('===addSuccess==',val);
      if (val){
         this.addQuery = Object.assign({}, defaultForm)
         this.entities = undefined
         this.functionName = undefined
         // this.describe = undefined
         this.paramsArr = []

      }
    }
  },
  created() {

  },
  computed: {
    // knowType: function () {
    //   console.log('===knowType==',this.knowType);
    //   // return this.knowType
    // }
  },
  methods: {
    addHandle(){
      console.log('===addHandle==table=',this.addQuery);
      if (this.knowType == 5){
        this.addQuery.value = this.entities+':'+this.functionName
      }
      this.addSuccess = false
      this.$emit("addHandle", this.addQuery);
    },
    cancelEdit(row) {
      console.log('==cancelEdit==',row)
      // row.Pid = row.originalId
      // row.Pname = row.originalName
      // row.FieldName = row.originalFieldName
      // row.Pdesc = row.originalDesc

      row.parentID = row.originalParentID
      row.dataType = row.originalDataType
      row.name = row.originalName
      row.valueType = row.originalValueType
      row.value = row.originalValue
      row.isEdit = false
      this.$message({
        message: '已经恢复上一次编辑',
        type: 'warning'
      })
    },
    confirmEdit(row) {
      row.isEdit = false
      this.tableKey++;
      console.log('===row.isEdit=333=',row.isEdit);
      this.$emit("editHandle", row);
    },
    editHandle(row){
      row.isEdit = !row.isEdit
      this.tableKey++;
      console.log('===editHandle=table=',row);
      // if(!row.isEdit){
      //   row.isEdit = true
      //   this.tableKey++;
      //   console.log('===row.isEdit=',row.isEdit);
      // }else{
      //   row.isEdit = false
      //   this.tableKey++;
      //   console.log('===row.isEdit=333=',row.isEdit);
      //   this.$emit("editHandle", row);

      // }

    },
    deleteHandle(row){
      console.log('===deleteHandle==table=',row);
      this.$emit("deleteHandle", row);
    },
    getListChildren() {
      console.log('come getListChildren====',this.isChildren+'==val==true')
      console.log('come getListChildren==this.listQuery==',this.listQuery)
      this.$emit("getList", this.listQuery, true);
    },
    getList() {
      console.log('come here====',this.isChildren)
      console.log('come here==this.listQuery==',this.listQuery)
      this.$emit("getList", this.listQuery);
    },
    handleSizeChange(val) {
    },
    handleCurrentChange(val) {
    },
    //点选父级
    getChildrenHandle(row, event, column){
      console.log('=getChildrenHandle=table=',row)
      if (row.isEdit){
        return
      }
      this.$emit("getChildrenHandle", row);
    },
    //选择动作实体，获取方法
    changeEntitiesHandle(){
      this.$emit("getReflector", {type: 2, classPath: this.entities}, 'getFunction');
    },
    //选择动作实体的方法，获取参数说明
    changeFunctionHandle(){
      this.$emit("getReflector", {type: 5, classPath: this.entities, methodName:this.functionName}, 'getParam');
    },

    // 添加参数
    addNewParam() {
      this.paramsArr.push({name: undefined, dataType:undefined, value: undefined})
      this.addQuery.params = this.paramsArr
    },
  }
}
</script>
<style>
  .addClass .el-form-item{
    float: left;
  }
  .addClass .el-form-item .el-form-item__label{
    float: left;
  }
  .addClass .el-form-item .el-form-item__content{
    float: left;
    margin-right: 20px;
  }

  .addClass .el-form-item .el-form-item{
    margin-bottom: 10px;
  }
/*
  .fLeft{
    float: left;
    height: 100%;
    width:50%;
    overflow-y: scroll;
    }
  .fRight{
      float: right;
      width:49%;
      height: 100%;
      margin-left:1%;
      overflow-y: scroll;
    }
  .contentClass {

    height: 680px;
    display: block;
    box-sizing: border-box;
    overflow-x: hidden;
    color: #323233;
    font-size: .875rem;
    line-height: 1.25rem;
    background-color: #fafafa;
    border-left: .1875rem solid transparent;
    user-select: none;
    text-align: left !important;
    border: medium solid #409eff;
    border-radius:5px;
    padding: 10px;
  } */
  </style>
<style scoped>



</style>
