<template>
<div class="app-container">
    <div class="filter-container addClass">
      <el-form :model="addQuery" class="demo-form-inline">
        <el-form-item label="名称">
          <el-input  style="width: 150px" v-model="addQuery.name" placeholder="请输入名称"></el-input>
        </el-form-item>
        <!-- <el-form-item v-if="knowType == 1 " label="值的数据类型"> -->
        <el-form-item  label="值的数据类型" v-if="showDataType">
        <el-select clearable v-model="addQuery.dataType" placeholder="请选择数据类型" class="filter-item searchUI" style="width: 180px">
          <el-option v-for="(item, index) in dataTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
        </el-select>
        </el-form-item>

        <el-form-item label="值" v-if="knowType == 1 && entitiesParamOption.length>0">
          <el-select id="entitiesParamSelect"  v-model="addQuery.value" filterable placeholder="请选择字段" @blur="selectBlur" @change="selectChange">
                <el-option
                    v-for="(item,index) in entitiesParamOption"
                    :key="index"
                    :label="item.column_name+' '+item.column_comment"
                    :value="item.column_name">
                 </el-option>
            </el-select>
        </el-form-item>
        <el-form-item label="值" v-else>
          <!-- <el-input style="width: 150px" v-model="addQuery.value" placeholder="请输入值"></el-input> -->
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
    <!-- <div class="filter-container" v-if="knowType == 1"> -->
    <div class="filter-container">
    <!-- <el-select clearable v-model="listQuery.systemID" placeholder="请选择系统" class="filter-item searchUI" style="width: 150px">
      <el-option v-for="(item, index) in systemOption" :key="index" :label="item.name" :value="item.id"></el-option>
    </el-select> -->
    <el-select v-if="knowType != 1" clearable v-model="listQuery.dataType" placeholder="请选择数据类型" class="filter-item searchUI" style="width: 120px">
      <el-option v-for="(item, index) in dataTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
    </el-select>
    <el-input @keyup.enter.native="getList" style="width: 120px;" class="filter-item" placeholder="名称" v-model="listQuery.name" clearable>
    </el-input>
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
    >
    <el-table-column align="center" label="ID" min-width="40">
      <template slot-scope="scope">
        <span>{{scope.row.id}}</span>
      </template>
    </el-table-column>

    <el-table-column align="center" label="父级ID" min-width="80">
      <template slot-scope="scope">
        <el-input v-if="scope.row.isEdit && knowType != 1" class="edit-input" size="small" type="number" v-model.number="scope.row.parentID" style="width=2rem;">            </el-input>
        <span v-else>{{scope.row.parentID}}</span>
      </template>
    </el-table-column>
    <el-table-column v-if="knowType != 1" align="center" label="数据类型">
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
    <el-table-column v-if="knowType != 1" align="center" label="值类型">
      <template slot-scope="scope">
        <el-select v-if="scope.row.isEdit" clearable v-model="scope.row.valueType" placeholder="请选择值类型" class="filter-item searchUI" style="width: 150px">
          <el-option v-for="(item, index) in ValueTypeOption" :key="index" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <span v-else>{{scope.row.valueType | displayNameFilter([{id: 0, name: '主类型'}, {id: 1, name: '实体Field'}, {id: 2, name: '动作参数'}])}}</span>
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
import { displayNameFilter } from "@/filters";
// const defaultPostForm = {
//   sid: undefined,
//   name: undefined,
//   id: undefined,
//   status:0, //状态
// };
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
    entitiesParamOption: {
      type: Array,
      default: []
    },
    listLoading:{          //加载进度调
      type: Boolean,
      default: false
    },
    // isEdit:{          //是否可以编辑
    //   type: Boolean,
    //   default: false
    // },
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
        value: undefined
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
      dataTypeStringList: [
        ['int','smallint','tinyint','decimal','bigint','numeric','mediumint','float'],
        ['varchar','longtext','mediumtext','text','tinytext','varbinary'],
        ['boolean','bool'],
        [],
        ['date','datetime','year'],
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

      // listLoading: false,
      showDataType: true,

    }
  },
  watch: {
    knowType(val) {
      console.log('===knowType==',val);
    },
    entitiesParamOption(val) {
      console.log('===entitiesParamOption====', val);
      if (val.length == 0){
        this.showDataType = true
        // this.entitiesParamOption = []
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
      console.log('===addHandleChildren==table=',this.addQuery);
      this.$emit("addHandle", this.addQuery, true);
    },
    cancelEdit(row) {
      console.log('==cancelEdit==',row)
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
      this.$emit("editHandle", row, true);
    },
    editHandle(row){
      row.isEdit = !row.isEdit
      this.tableKey++;
      console.log('===editHandle=table=',row);
    },
    deleteHandle(row){
      console.log('===deleteHandle==table=',row);
      this.$emit("deleteHandle", row, true);
    },
    getList() {
      console.log('come getListChildren====',this.isChildren+'==val==true')
      console.log('come getListChildren==this.listQuery==',this.listQuery)
      this.$emit("getList", this.listQuery, true);
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
    //输入字段值
    selectBlur(e) {
        if (e.target.value) {
         // 不在表单的时候不需要判断，也不需要isname字段
            this.addQuery.value = e.target.value;
            this.showDataType = true;
            console.log('==inputed===',this.addQuery)

        } else {
            this.showDataType = false;
            // console.log('==selected===',this.addQuery)
        }
    },
    //如果是选择下拉字段
    selectChange(vID){
      console.log('==selectChange===',vID)
      let obj = {}
      obj = this.entitiesParamOption.find(item => {
        return item.column_name === vID;
      });
      // console.log('obj===',obj)
      this.dataTypeStringList.forEach((item,index) =>{
          var key= item.findIndex(value=>{
                if(value == obj.data_type){
                    return true
                }
            })
          // console.log('===key====',index+'====='+key)
          if (key !== -1){
            this.addQuery.value = obj.column_name;
            this.addQuery.dataType = index;
            return true
          }
          // console.log('===addQuery====',this.addQuery)
      })

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

  /* .fLeft{
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
  .contentClass1 {
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
