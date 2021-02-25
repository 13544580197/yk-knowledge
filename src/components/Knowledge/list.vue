<template>
<div class="app-container">

    <div v-if="knowType == 1 || knowType == 4 || knowType == 5">
      <div class="fLeft contentClass">
        <!-- left -->
        <listTable
          v-if="knowType == 1 || knowType == 4 || knowType == 5"
          :knowType="knowType"
          :listLoading.sync="listLoading"
          :list="list"
          :entitiesOption="entitiesOption"
          :funcitonOption="funcitonOption"
          :describe="describe"
          :addSuccess.sync="addSuccess"
          @getChildrenHandle="getChildrenHandle"
          @getList="getList"
          @addHandle="addHandle"
          @editHandle="editHandle"
          @deleteHandle="deleteHandle"
          @getReflector="getReflector"
        ></listTable>

      </div>
      <div v-if="curParentId != 0" class="fRight contentClass">
        <!-- right -->
        <listChildTable v-if="(knowType == 1 || knowType == 4 || knowType == 5) && curParentId != 0"
        :knowType="knowType"
         :listLoading.sync="listLoading"
        :isChildren.sync="isChildren"
        :entitiesParamOption.sync="entitiesParamOption"
        :list="itemList"
        @getList="getList"
        @addHandle="addHandle"
        @editHandle="editHandle"
        @deleteHandle="deleteHandle"
        ></listChildTable>

      </div>
    </div>

    <div v-else>
        <listTable
        v-if="knowType == 0 || knowType == 2"
        :knowType="knowType"
        :listLoading.sync="listLoading"
        :list="list"
        :entitiesOption="entitiesOption"
        :funcitonOption="funcitonOption"
        :describe="describe"
        @getList="getList"
        @addHandle="addHandle"
        @editHandle="editHandle"
        @deleteHandle="deleteHandle"
        ></listTable>

    </div>

  </div>
</template>

<script>
// import { fetchShopList } from "@/api/shop";
// import { fetchActivity, createActivity, updateActivity } from '@/api/activity'
import {
  engineConstantsList, engineConstants, updateEngineConstants, createEngineConstants, delEngineConstants,
  engineVariablesList, engineVariables, updateEngineVariables, createEngineVariables, delEngineVariables,
  engineEntitiesList, engineEntities, updateEngineEntities, createEngineEntities, delEngineEntities,
  engineEntitiesFieldsList, engineEntitiesFields, updateEngineEntitiesFields, createEngineEntitiesFields, delEngineEntitiesFields,
  engineEventList, engineEvent, updateEngineEvent, createEngineEvent, delEngineEvent,
  engineEventParamList, engineEventParam, updateEngineEventParam, createEngineEventParam, delEngineEventParam,
  engineActionList, engineAction, updateEngineAction, createEngineAction, delEngineAction,
  engineActionParamList, engineActionParam, updateEngineActionParam, createEngineActionParam, delEngineActionParam,
  engineEntitiesReflector,
  } from '@/api/knowledge'

import { displayShopFilter,displayNameFilter } from "@/filters";
import listTable from './listTable'
import listChildTable from './listChildTable'
// import { dict } from "@/store";

const defaultPostForm = {
  sid: undefined,
  name: undefined,
  id: undefined,
  status:0, //状态
};
export default {
  name: "knowledge-list",
  components: { listTable, listChildTable },
  props: {
    knowType: {             //类型 0变量 1实体 2常量 3事件 4动作
      type: Number,
      default: 0
    },
    systemID: {             //系统id 0 御客餐饮  1供应链
      type: Number,
      default: 0
    },

  },
  data() {
    return {
      list: [],                 // 知识库列表
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
        {id: 3, name: 'Date'},
        {id: 4, name: 'Object'},
        ],

      // variablesList:[
      //   {id: 21, parentID:0, dataType:5, name:'活动主体1', valueType:1, value: 'activity.name', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 22, parentID:1, dataType:1, name:'御客餐饮1', valueType:0, value: 0, createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 23, parentID:1, dataType:1, name:'御客外面1', valueType:0, value: 1, createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'}
      // ],
      // entitiesList:[
      //   {id: 11, parentID:0, dataType:undefined, name:'顾客', valueType:undefined, value: 'cust', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 12, parentID:0, dataType:undefined, name:'店铺', valueType:undefined, value: 'shop', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'}
      // ],
      // entitiesItemList:[
      //   {id: 31, parentID:11, dataType:0, name:'顾客等级', valueType:1, value: 'level', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 32, parentID:11, dataType:0, name:'顾客性别', valueType:1, value: 'sex', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 33, parentID:12, dataType:1, name:'店铺名称', valueType:1, value: 'name', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'}
      // ],
      // constantsList:[
      //   {id: 2, parentID:0, dataType:1, name:'加', valueType:0, value: '+', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 3, parentID:0, dataType:1, name:'减', valueType:0, value: '-', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 5, parentID:0, dataType:1, name:'大于', valueType:0, value: '>', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'},
      //   {id: 6, parentID:0, dataType:1, name:'大于或等于', valueType:0, value: '>=', createDT: 1585497600, createDTStr: '2021-02-04 09:00', opUserName:'芳'}
      // ],

      isChildren: false, //是否需要请求下一级
      curParentId: 0, //当前选中实体id
      curParent: {}, //当前选中实体
      itemList:[], //当前实体字段、事件参数、动作参数
      entitiesOption:[], //动作实体列表
      funcitonOption: [], //动作方法
      entitiesParamOption: [], //对应数据模型的实体包含字段列表
      // entitiesParamOption: [{name:'name',datatype:'int', discr:'名称'},{name:'address',datatype:'string', discr:'地址'}], //对应数据模型的实体包含字段列表
      describe:undefined,  //方法说明
      addSuccess: false, //是否添加成功

      listLoading: false,
    }
  },
  watch: {
    knowType(val) {
      console.log('===knowType==',val);
    }
  },
  created() {

    // dict('shop', 1, 0, 0).then(response => {
    //   // console.log('=dict=',this.$store.getters.session)
    //   this.shopOptions = displayShopFilter(this.$store.getters.session.sids, response)
    // })
    this.getList()
  },
  computed: {

    // knowType: function () {
    //   console.log('===knowType==',this.knowType);
    //   // return this.knowType
    // }
  },
  methods: {
    // getList(query) {
    getList(query, isChildren) {
      console.log('come here====')
      if (query){
        this.listQuery = query
      }
      //for test
      if (this.knowType == 0){
        // this.list = this.variablesList
        engineVariablesList(this.listQuery).then(response => {
          this.total = response.data.total
          // this.list = response.data.list
          this.list = response.data.list.map(item => {
            item.isEdit = false
            item.originalParentID = item.parentID
            item.originalDataType = item.dataType
            item.originalName = item.name
            item.originalValueType = item.valueType
            item.originalValue = item.value
            // console.log('=item===',item)
            return item
          })
        })
      }
      if (this.knowType == 1){
        this.isChildren = isChildren
        console.log('===实体列表==',this.listQuery)
        console.log('===实体列表=isChildren=',this.isChildren+'==isChildren=='+this.curParentId)

        if (this.isChildren){
          engineEntitiesFieldsList(this.listQuery, this.curParentId).then(response => {
            this.total = response.data.total
            // this.list = response.data.list
            this.itemList = response.data.list.map(item => {
              item.isEdit = false
              item.originalParentID = item.parentID
              item.originalDataType = item.dataType
              item.originalName = item.name
              item.originalValueType = item.valueType
              item.originalValue = item.value
              // console.log('=item===',item)
              return item
            })
          })
        }else{
          engineEntitiesList(this.listQuery).then(response => {
            this.total = response.data.total
            // this.list = response.data.list
            this.list = response.data.list.map(item => {
              item.isEdit = false
              item.originalParentID = item.parentID
              item.originalDataType = item.dataType
              item.originalName = item.name
              item.originalValueType = item.valueType
              item.originalValue = item.value
              // console.log('=item===',item)
              return item
            })
          })
        }

      }
      if (this.knowType == 2){
        // this.list = this.constantsList
        engineConstantsList(this.listQuery).then(response => {
          this.total = response.data.total
          // this.list = response.data.list
          this.list = response.data.list.map(item => {
            item.isEdit = false
            item.originalParentID = item.parentID
            item.originalDataType = item.dataType
            item.originalName = item.name
            item.originalValueType = item.valueType
            item.originalValue = item.value
            // console.log('=item===',item)
            return item
          })
        })
      }

      if (this.knowType == 4){
        this.isChildren = isChildren
        console.log('===事件列表==',this.listQuery)
        console.log('===事件列表=isChildren=',this.isChildren+'==isChildren=='+this.curParentId)

        if (this.isChildren){
          engineEventParamList(this.listQuery, this.curParentId).then(response => {
            this.total = response.data.total
            // this.list = response.data.list
            this.itemList = response.data.list.map(item => {
              item.isEdit = false
              item.originalParentID = item.parentID
              item.originalDataType = item.dataType
              item.originalName = item.name
              item.originalValueType = item.valueType
              item.originalValue = item.value
              // console.log('=item===',item)
              return item
            })
          })
        }else{
          engineEventList(this.listQuery).then(response => {
            this.total = response.data.total
            // this.list = response.data.list
            this.list = response.data.list.map(item => {
              item.isEdit = false
              item.originalParentID = item.parentID
              item.originalDataType = item.dataType
              item.originalName = item.name
              item.originalValueType = item.valueType
              item.originalValue = item.value
              // console.log('=item===',item)
              return item
            })
          })
        }

      }
      if (this.knowType == 5){
        this.isChildren = isChildren
        console.log('===动作列表==',this.listQuery)
        console.log('===动作列表=isChildren=',this.isChildren+'==isChildren=='+this.curParentId)

        if (this.isChildren){
          engineActionParamList(this.listQuery, this.curParentId).then(response => {
            this.total = response.data.total
            // this.list = response.data.list
            this.itemList = response.data.list.map(item => {
              item.isEdit = false
              item.originalParentID = item.parentID
              item.originalDataType = item.dataType
              item.originalName = item.name
              item.originalValueType = item.valueType
              item.originalValue = item.value
              // console.log('=item===',item)
              return item
            })
          })
        }else{
          engineActionList(this.listQuery).then(response => {
            this.total = response.data.total
            // this.list = response.data.list
            this.list = response.data.list.map(item => {
              item.isEdit = false
              item.originalParentID = item.parentID
              item.originalDataType = item.dataType
              item.originalName = item.name
              item.originalValueType = item.valueType
              item.originalValue = item.value
              // console.log('=item===',item)
              return item
            })
          })
          // this.getReflector({type: 2})
          //获取实体
          engineEntitiesList({limit:1000}).then(response => {
            this.entitiesOption = response.data.list
          })
        }

      }

      this.listLoading = false
      // this.total = this.list.length
      console.log('knowType:',this.knowType)
      console.log('getlist:',this.list)
    },

    //获取实体相关数据
    getReflector(query, type){
      if (type == 'getFunction'){
        engineEntitiesReflector(query).then(response => {
          this.funcitonOption = response.data.list
        })
      }
      if (type == 'getParam'){
        engineEntitiesReflector(query).then(response => {
          this.describe = response.data.describe
        })
      }
      //获取实体对用模型的字段
      if (type == 'getEntitiesParam'){
        this.entitiesParamOption = []
        // entitiesParamOption
        engineEntitiesReflector(query).then(response => {
          this.entitiesParamOption = response.data.column_list
          // this.entitiesParamOption = response.data.list
          console.log('==entitiesParamOption===',this.entitiesParamOption);
        })
      }
    },

    getChildrenHandle(row){
      console.log('=getChildrenHandle==',row)
      this.curParentId = row.id
      this.curParent = row
      this.isChildren = true
      // this.itemList = this.entitiesItemList
      // this.getList()
      this.listQuery.name = undefined
      this.getList(this.listQuery, this.isChildren)
      if (this.knowType == 1){
        let classPath = this.curParent.namespace+'\\'+this.curParent.value
        console.log('====classPath==',classPath);
        this.getReflector( {type: 6, classPath: classPath}, 'getEntitiesParam')
      }

    },
    addHandle(query,isChildren){
      console.log('=addHandle=list=',query)
      console.log('=addHandle=knowType=',this.knowType)
      if (this.knowType == 0){
        createEngineVariables(query).then(response => {
            this.$message({
              type: 'success',
              message: response.msg
            })
            // this.addSuccess = true
            this.getList()
        }).catch(() => {})
      }
      if (this.knowType == 1){
        this.isChildren = isChildren
        console.log('=addHandle=curParentId=',this.curParentId+'==isChildren=='+this.isChildren)
        if (this.isChildren){
          createEngineEntitiesFields(query, this.curParentId).then(response => {
              this.$message({
                type: 'success',
                message: response.msg
              })
              // this.addSuccess = true
              this.getList(this.listQuery, isChildren)
          }).catch(() => {})
        }else{
          createEngineEntities(query).then(response => {
              this.$message({
                type: 'success',
                message: response.msg
              })
              this.addSuccess = true
              this.describe = undefined
              this.getList()
          }).catch(() => {})
        }
      }
      if (this.knowType == 2){
        createEngineConstants(query).then(response => {
            this.$message({
              type: 'success',
              message: response.msg
            })
            // this.addSuccess = true
            this.getList()
        }).catch(() => {})
      }
      if (this.knowType == 4){
        this.isChildren = isChildren
        console.log('=addHandle=curParentId=',this.curParentId+'==isChildren=='+this.isChildren)
        if (this.isChildren){
          createEngineEventParam(query, this.curParentId).then(response => {
              this.$message({
                type: 'success',
                message: response.msg
              })
              // this.addSuccess = true
              this.getList(this.listQuery, isChildren)
          }).catch(() => {})
        }else{
          createEngineEvent(query).then(response => {
              this.$message({
                type: 'success',
                message: response.msg
              })
              this.addSuccess = true
              this.describe = undefined
              this.getList()
          }).catch(() => {})
        }
      }

      if (this.knowType == 5){
        this.isChildren = isChildren
        console.log('=addHandle=curParentId=',this.curParentId+'==isChildren=='+this.isChildren)
        if (this.isChildren){
          createEngineActionParam(query, this.curParentId).then(response => {
              this.$message({
                type: 'success',
                message: response.msg
              })
              // this.addSuccess = true
              this.getList(this.listQuery, isChildren)
          }).catch(() => {})
        }else{
          createEngineAction(query).then(response => {
              this.$message({
                type: 'success',
                message: response.msg
              })
              this.addSuccess = true
              this.describe = undefined
              this.getList()
          }).catch(() => {})
        }
      }


    },
    editHandle(row, isChildren){
      console.log('===editHandle=',row);
      let id = row.id
      if (this.knowType == 0){
        updateEngineVariables(id, row).then(response => {
            this.$message({
              // showClose: true,
              type: 'success',
              message: response.msg
            })
            // this.getList()
        }).catch(() => {})
      }
      if (this.knowType == 1){
        if (isChildren){
          // updateEngineEntitiesFields(id, row).then(response => {
          updateEngineEntitiesFields(this.curParentId,id, row).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              // this.getList()
          }).catch(() => {})
        }else{
          updateEngineEntities(id, row).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              // this.getList()
          }).catch(() => {})
        }

      }
      if (this.knowType == 2){
        updateEngineConstants(id, row).then(response => {
            this.$message({
              // showClose: true,
              type: 'success',
              message: response.msg
            })
            row.isEdit = false
            // this.getList()
        }).catch(() => {})
      }
      if (this.knowType == 4){
        if (isChildren){
          // updateEngineEntitiesFields(id, row).then(response => {
          updateEngineEventParam(this.curParentId,id, row).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              // this.getList()
          }).catch(() => {})
        }else{
          updateEngineEvent(id, row).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              // this.getList()
          }).catch(() => {})
        }
      }
      if (this.knowType == 5){
        if (isChildren){
          // updateEngineEntitiesFields(id, row).then(response => {
          updateEngineActionParam(this.curParentId,id, row).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              // this.getList()
          }).catch(() => {})
        }else{
          updateEngineAction(id, row).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              // this.getList()
          }).catch(() => {})
        }
      }

    },
    deleteHandle(row, isChildren){
      console.log('===deleteHandle=',row);
      let id = row.id
      if (this.knowType == 0){
        delEngineVariables(id).then(response => {
            this.$message({
              // showClose: true,
              type: 'success',
              message: response.msg
            })
            this.getList()
        }).catch(() => {})
      }
      if (this.knowType == 1){
        if (isChildren){
          delEngineEntitiesFields(this.curParentId, id).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              this.getList(this.listQuery, true)
          }).catch(() => {})
        }else{
          delEngineEntities(id).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              this.getList()
          }).catch(() => {})
        }

      }
      if (this.knowType == 2){
        delEngineConstants(id).then(response => {
            this.$message({
              // showClose: true,
              type: 'success',
              message: response.msg
            })
            this.getList()
        }).catch(() => {})
      }
      if (this.knowType == 4){
        if (isChildren){
          delEngineEventParam(this.curParentId, id).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              this.getList(this.listQuery, true)
          }).catch(() => {})
        }else{
          delEngineEvent(id).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              this.getList()
          }).catch(() => {})
        }
      }
      if (this.knowType == 5){
        if (isChildren){
          delEngineActionParam(this.curParentId, id).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              this.getList(this.listQuery, true)
          }).catch(() => {})
        }else{
          delEngineAction(id).then(response => {
              this.$message({
                // showClose: true,
                type: 'success',
                message: response.msg
              })
              this.getList()
          }).catch(() => {})
        }
      }
    },
  }
}
</script>

<style scoped>
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

  /* padding: .75rem .5rem .75rem .75rem; */
  height: 650px;
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
  /* border: medium inset #FF0000; */
  /* border: medium solid #409eff; */
  border: medium solid #dadfe5;
  border-radius:5px;
  padding: 10px;
}

</style>
