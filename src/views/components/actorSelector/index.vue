<template>
  <div>
    <div class="tag">
      <el-tag
          v-for="tag in tags"
          :key="tag.type+tag.id"
          closable effect="plain" type="info" @close="tagClose(tag)">
        {{tag.name}}
      </el-tag>
    </div>

    <el-tabs v-model="activeTab">
      <el-tab-pane label="部门" name="dept" v-if="tabs.indexOf('dept')>=0">
        <el-input class="filter" placeholder="搜索部门" v-model="filterDept" clearable @input="filterDeptChanged" >
          <i class="el-icon-search el-input__icon" slot="prefix"></i>
        </el-input>
        <div class="border">
          <div class="treeHeight">
            <el-tree
                ref="tDept2"
                v-show="filterDept!==''"
                node-key="id"
                :props="propsTree"
                show-checkbox
                :data="filterDepts"
                @check="handleTreeCheckChanged">
            </el-tree>
            <el-tree
                ref="tDept1"
              v-show="filterDept===''"
              node-key="id"
              :props="propsTree"
              show-checkbox
              :check-strictly="true"
              :load="loadDepts" lazy
              :default-expanded-keys="expandedDepts"
              @check="handleTreeCheckChanged">
          </el-tree>
          </div>
        </div>
      </el-tab-pane>

      <el-tab-pane label="人员" name="user" v-if="tabs.indexOf('user')>=0">
        <el-input class="filter" placeholder="搜索人员" v-model="filterUser" clearable @input="filterUserChanged" >
          <i class="el-icon-search el-input__icon" slot="prefix"></i>
        </el-input>
        <div class="border">
          <div class="treeHeight" v-show="filterUser!==''">
            <el-tree
                ref="tUser2"
                node-key="id"
                :props="propsTree"
                show-checkbox
                :data="filterUsers"
                @check="handleTreeCheckChanged">
            </el-tree>
          </div>
          <el-row v-show="filterUser===''">
            <el-col :span="12" style="border-right: 1px solid #ddd">
              <div class="treeHeight">
                <el-tree
                    node-key="id"
                    :props="propsTree"
                    :load="loadDepts" lazy
                    :default-expanded-keys="expandedDepts"
                    highlight-current
                    :expand-on-click-node="false"
                    @node-click="userDeptSelected">
                </el-tree>
              </div>
            </el-col>
            <el-col :span="12">
              <div class="treeHeight">
                <el-tree
                    ref="tUser1"
                    node-key="id"
                    :props="propsTree"
                    show-checkbox
                    :check-strictly="true"
                    :data="users"
                    @check="handleTreeCheckChanged">
                </el-tree>
              </div>
            </el-col>
          </el-row>
        </div>
      </el-tab-pane>

      <el-tab-pane label="角色" name="role" v-if="tabs.indexOf('role')>=0">
        <el-input class="filter" placeholder="搜索角色" v-model="filterRole" clearable >
          <i class="el-icon-search el-input__icon" slot="prefix"></i>
        </el-input>
        <div class="border">
          <div class="treeHeight">
            <el-tree
                ref="tRole"
                node-key="id"
                :props="propsTree"
                show-checkbox
                :data="filterRoles"
                @check="handleTreeCheckChanged">
            </el-tree>
          </div>
        </div>
      </el-tab-pane>

      <el-tab-pane label="字段" name="field" v-if="tabs.indexOf('field')>=0">
        <div class="border border-plus">
          <div class="treeHeight treeHeight-plus">
            <el-tree
                ref="tField"
                node-key="id"
                :props="propsTree"
                show-checkbox
                :data="datas.fields"
                @check="handleTreeCheckChanged">
            </el-tree>
          </div>
        </div>
      </el-tab-pane>

      <el-tab-pane label="函数" name="function" v-if="tabs.indexOf('function')>=0">
        <div class="border border-plus">
          <div class="treeHeight treeHeight-plus">
            <div v-for="func in functions" :key="func.id" class="funcBox">
              <div class="funcTitle">
                {{func.name}}
                <el-link type="primary" @click="funcAdd(func)">+添加</el-link>
              </div>
              <div class="funcNote">{{func.note}}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>

    <div v-if="footer" class="footer">
      <el-button @click="onCancel" type="default">取消</el-button>
      <el-button @click="onConfirm" type="primary">确定</el-button>
    </div>
  </div>
</template>

<script>
export default {
  name: "ActorSelector",
  data() {
    return {
      activeTab:'dept',
      timer:null,
      propsTree: {
        label: 'name',
        children: 'children',
        isLeaf: 'isLeaf'
      },
      tags:[],
      expandedDepts:['dept1'],
      filterDept:'',//过滤值
      filterDepts:[],//过滤数据源
      filterUser:'',//过滤值
      filterUsers:[],//过滤数据源,
      users:[],//树形列表数据源
      filterRole:'',//过滤值
      functions:[
        {id:"findDeptManager",name:"部门经理函数",note:"获取对应部门的所有经理。"},
        {id:"findRole",name:"查找角色函数",note:"获取“角色”中属于指定“部门”的所有成员。"},
      ],
      checkedDeptKeys:[],
      checkedUserKeys:[],
      checkedRoleKeys:[],
      checkedFieldKeys:[],
      checkedFunctionKeys:[],
    }
  },
  computed:{
    filterRoles(){
      if(this.filterRole==='')return this.datas.roles;
      return this.datas.roles.filter(k=>{
        return k.name.match(this.filterRole)
      })
    },
  },
  props:{
    datas: {//初始数据来源
      type:Object,
      default:()=>{
        return {}
      }
    },
    selectTags:{//已选tags
      type:Array,
    },
    dataProvider:{//数据提供函数
      type:Function,
      default:null
    },
    tabs:{//控制页签是否可见
      type:Array,
      default:()=>{
        return ['dept','user','role','field','function']
      }
    },
    apiDelay:{//api节流，400ms后才调用接口
      type:Number,
      default:400
    },
    footer:{//是否显示底部按钮栏
      type:Boolean,
      default:true
    },
    actorsNameMaxLength:{//所选tag名称累计最大长度
      type:Number,
      default:25
    }
  },
  watch:{
    // filterDept:{
    //   handler(val,oldVal){
    //     if(val!==''&& oldVal===''){
    //       this.setTreeCheckKeys(true);
    //     }
    //   }
    // },
    selectTags:{
      handler(val){
        if(val && val.length>0){
          this.tags=val;
          this.refreshCheckedKeys();
          this.setTreeCheckKeys(true,true,true,true);
        }
      },
      immediate:true
    }
  },
  methods:{
    getCheckedKeysArray(type){
      let array;
      if(type==='dept'){
        array=this.checkedDeptKeys;
      }else if(type==='user'){
        array=this.checkedUserKeys;
      }else if(type==='role'){
        array=this.checkedRoleKeys;
      }else if(type==='function'){
        array=this.checkedFunctionKeys;
      }else{
        array=this.checkedFieldKeys;
      }
      return array;
    },
    addCheckedKeys(item){
      let array=this.getCheckedKeysArray(item.type);
      if(array){
        array.push(item.id);
      }
    },
    removeCheckedKeys(item){
      let array=this.getCheckedKeysArray(item.type);
      if(array){
        let index=array.indexOf(item.id);
        array.splice(index,1);
      }
    },
    refreshCheckedKeys(){
      const that=this;
      this.checkedDeptKeys=[];
      this.checkedUserKeys=[];
      this.checkedRoleKeys=[];
      this.checkedFieldKeys=[];
      this.checkedFunctionKeys=[];
      this.tags.forEach(function(item){
        that.addCheckedKeys(item);
      });
    },
    setTreeCheckKeys(dept,user,role,field){
      this.$nextTick(function(){
        if(dept) {
          this.$refs.tDept1.setCheckedKeys(this.checkedDeptKeys);
          this.$refs.tDept2.setCheckedKeys(this.checkedDeptKeys);
        }
        if(user) {
          this.$refs.tUser1.setCheckedKeys(this.checkedUserKeys);
          this.$refs.tUser2.setCheckedKeys(this.checkedUserKeys);
        }
        if(role) {
          this.$refs.tRole.setCheckedKeys(this.checkedRoleKeys);
        }
        if(field) {
          this.$refs.tField.setCheckedKeys(this.checkedFieldKeys);
        }
      });
    },
    getActors(){
      let name="";
      for (let i = 0; i < this.tags.length; i++) {
        name+=this.tags[i].name+",";
      }
      const ml=this.actorsNameMaxLength-3
      if(name.length>ml)name=name.substring(0,ml)+'...';
      else name=name.substring(0,name.length-1);
      let data={
        actorsName:name,
        actors:this.tags
      }
      // console.error(JSON.stringify(data));
      return data;
    },
    filterDeptChanged(){
      clearTimeout(this.timer);
      const that=this;
      this.timer=setTimeout(function(){
        if(that.dataProvider){
          if(that.filterDept==='')return;
          that.filterDepts=that.dataProvider('filterDept',that.filterDept);
          that.setTreeCheckKeys(true);
        }
      },this.apiDelay);
    },
    loadDepts(node, resolve) {
      if (node.level ===0) {
        return resolve(this.datas.depts);
      }else if(node.level===1){
        return resolve(node.data.children);
      }
      let returnData=[];
      if(this.dataProvider){
        returnData=this.dataProvider('dept',node.data.id);
      }
      return resolve(returnData);
    },
    filterUserChanged(){
      clearTimeout(this.timer);
      const that=this;
      this.timer=setTimeout(function(){
        if(that.dataProvider){
          if(that.filterUser==='')return;
          that.filterUsers=that.dataProvider('filterUser',that.filterUser);
          that.setTreeCheckKeys(false,true);
        }
      },that.apiDelay);
    },
    userDeptSelected(data){
      if(this.dataProvider){
        this.users=this.dataProvider('user',data.id);
        this.setTreeCheckKeys(false,true);
      }
    },
    handleTreeCheckChanged(data) {
      const arr=this.getCheckedKeysArray(data.type);
      const exists=arr.indexOf(data.id)>=0;
      if(!exists) {
        this.addTags(data.type,data.id,data.name,true);
      }else{
        this.removeTags(data.type,data.id);
      }
    },
    addTags(type,id,name){
      let tag = {id: id,name: name,type: type};
      this.tags.push(tag);
      this.addCheckedKeys(tag);
    },
    removeTags(type,id){
      for (let i = 0; i < this.tags.length; i++) {
        if (this.tags[i].id === id && this.tags[i].type===type) {
          this.removeCheckedKeys(this.tags[i]);
          this.tags.splice(i, 1);
          break;
        }
      }
    },
    funcAdd(func){
      let tag={
        id:func.id,
        name:func.name,
        type:'func'
      }
      let exists=false;
      for (let i = 0; i < this.tags.length; i++) {
        if (this.tags[i].id === tag.id && this.tags[i].type===tag.type) {
          exists=true;
          break;
        }
      }
      if(exists){
        this.$message({
          message:'不能重复添加',
          type: 'warning',
          duration:2000
        });
        return;
      }
      this.tags.push(tag);
    },
    tagClose(tag){
      this.removeTags(tag.type,tag.id);
      this.setTreeCheckKeys(
          tag.type==='dept',
          tag.type==='user',
          tag.type==='role',
          tag.type==='field'||tag.type==='variable'
      );
    },
    onConfirm(){
      this.$emit('confirm',this.getActors());
    },
    onCancel(){
      this.$emit('cancel');
    }
  },
}
</script>

<style scoped>
  .border,.border-plus{
    border: 1px solid #ddd;
    border-top: 0;
  }
  .border-plus{
    border-top: 1px solid #ddd;
  }
  .treeHeight{
    height: calc(100vh*0.30);
    min-height: 200px;
    overflow: auto;
  }
  .treeHeight-plus{
    height: calc(100vh*0.30 + 31px) !important;
  }
  .tag{
    border: 1px solid #ddd;
    padding: 10px;
    text-align: left;
    height: 100px;
    overflow: auto;
  }
  .funcBox{
    border-bottom: 1px solid #ddd;
    text-align: left;
    padding: 10px;
  }
  .funcTitle{
    font-weight: bold;
    padding: 5px;
  }
  .funcNote{
    font-size: 11px;
    padding: 5px;
  }
  .el-tag + .el-tag{
    margin-left: 10px;
  }
  .footer{
    text-align: right;
    margin-top: 10px;
  }
</style>
<style>
  .filter>.el-input__inner{
    border-radius: 0 !important;
  }

  /*以下为滚动条样式*/
  ::-webkit-scrollbar {
    /*滚动条整体样式*/
    width: 6px;
    /*高宽分别对应横竖滚动条的尺寸*/
    height               : 6px;
    scrollbar-arrow-color: red;

  }

  ::-webkit-scrollbar-thumb {
    /*滚动条里面小方块*/
    border-radius        : 5px;
    box-shadow           : inset 0 0 5px rgba(0, 0, 0, 0.2);
    background           : rgba(0, 0, 0, 0.2);
    scrollbar-arrow-color: red;
  }

  ::-webkit-scrollbar-track {
    /*滚动条里面轨道*/
    box-shadow   : inset 0 0 5px rgba(0, 0, 0, 0.2);
    border-radius: 0;
    background   : rgba(0, 0, 0, 0.1);
  }
</style>
