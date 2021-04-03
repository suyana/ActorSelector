<template>
  <div>
    <div>
      <el-button type="text" @click="dialogVisible=true">{{ buttonName }}</el-button>
      <el-dialog title="选择" :visible.sync="dialogVisible" >
        <actor-selector :datas="actorInitData" :select-tags="actorSelectedData"
                        :dataProvider="dataProvider" @confirm="onActorConfirm" @cancel="dialogVisible=false" />
      </el-dialog>
    </div>

  </div>

</template>
<script>
import actorSelector from './actorSelector/index'
export default {
  name: "Home",
  components:{actorSelector},
  data(){
    return {
      buttonName:'选择',
      dialogVisible:false,
      actorInitData:{
        depts:[
            {id:"dept1",name:"总公司",type:'dept',isLeaf:false,children:[
                {id:"dept2",name:"部门A",type:'dept',isLeaf:false,children:[
                  {id:"dept5",name:"部门H",type:'dept'}
                ]},
              {id:"dept3",name:"部门B",type:'dept',isLeaf:true,children:[]},
              {id:"dept4",name:"部门C",type:'dept',isLeaf:true,children:[]},
            ]
          }
        ],
        roles:[
          {id:"role1",name:"角色1",type:'role'},
          {id:"role2",name:"角色2",type:'role'},
          {id:"role3",name:"角色3",type:'role'},
          {id:"role4",name:"角色4",type:'role'},
        ],
        fields:[
          {id:"OwnId",name:"创建人",type:'field'},
          {id:"OwnDeptId",name:"所属部门",type:'field'},
          {id:"CreatedOwnId",name:"部门",type:'field'},
          {id:"Contact",name:"联系人",type:'variable'},
        ],
      },
      dataProvider:null,
      actorSelectedData:[
        {"id":"dept2","name":"部门A","type":"dept"},
        {"id":"userdept21","name":"user1","type":"user"},
        {"id":"OwnDeptId","name":"所属部门","type":"field"},
        {"id":"Contact","name":"联系人","type":"variable"}
      ]
    }
  },
  methods:{
    onActorConfirm(data){
      this.buttonName=data.actorsName;
      if(this.buttonName==='')this.buttonName='选择';
      this.actorSelectedData=data.actors;
      this.dialogVisible=false;
      console.error(JSON.stringify(data))
    }
  },
  mounted() {
    this.dataProvider=function(type,value){
      console.warn('get data'+type+','+value)
      if(type==='dept'){
        return [
          {id:value+'1',name:"部门"+value+'1',type:type,isLeaf:true,children:[]}
        ]
      }if(type==='filterDept' || type==='filterUser' || type==='user'){
        let type2=type;
        if(type==='filterDept')type2='dept';
        else if(type==='filterUser')type2='user';
        return [
          {id:type+value+'1',name:type+'1',type:type2,isLeaf:true,children:[]},
          {id:type+value+'2',name:type+"2",type:type2,isLeaf:true,children:[]},
          {id:type+value+'3',name:type+"3",type:type2,isLeaf:true,children:[]}
        ]
      }
    }
  }
}
</script>

<style scoped>

</style>
