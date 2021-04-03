# ActorSelector

### 人员选择组件
- 最近在做工作流设计器，封装了个审核人选择组件
- 支持部门、人员、角色、表单字段、自定义函数（需自己处理）等选择
- 界面参考了氚云的设计
- 本接口不调用api，提供一个函数变量用来提供数据，接口需调用方自己调用并把数据通过该变量返回

### 效果图

![Image text](https://github.com/suyana/ActorSelector/main/snapshot/1.png)
![Image text](https://github.com/suyana/ActorSelector/main/snapshot/2.png)


### 安装依赖
```
npm install
```

### 本地运行
```
npm run serve
```

### 示例
```
<div>
      <el-button type="text" @click="dialogVisible=true">{{ buttonName }}</el-button>
      <el-dialog title="选择" :visible.sync="dialogVisible" >
        <actor-selector :datas="actorInitData" :select-tags="actorSelectedData"
                        :dataProvider="dataProvider" @confirm="onActorConfirm" @cancel="dialogVisible=false" />
      </el-dialog>
</div>
```
### Props说明：

- datas
  
    类型：Object，分为dept、user、role、field等字段。
  
        dept：部门树形数据，默认包含2级数据，第3级往后数据通过接口返回
  
        user：无需返回数据，通过接口返回
  
        role：角色数据
  
        field：字段数据
  
- select-tags
    
    类型：Array，已选数据
  
- dataProvider
  
    数据提供函数，参数（type，value）
  
        type值为：dept、filterDept、user、filterUser，对应value为：
        
        value：部门id，部门名称，部门id，用户名称
  
        当展开一个部门时，type为dept，需返回该部门下的子部门数据。
    
        当搜索部门时，type为filterDept，需返回符合该名字的部门数据。
  
        当点击一个部门时，type为user，需返回该部门下的人员信息。
  
        当搜索人员时，type为filterUser，需返回符合该名字的用户数据。
  ```
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
  ```

- tabs 控制页签是否可见

  类型：Array，默认值：['dept','user','role','field','function']

- apiDelay api节流，400ms后才调用接口

  类型：Number，默认：400

- footer 是否显示底部按钮栏

  类型：Boolean，默认：true

- actorsNameMaxLength 所选项目名称合计最大长度（超过用...代替）

  类型：Number，默认：25
### 事件：
- confirm 确定按钮点击事件，返回已选tag，结构如下
    ```
  {
      actorsName:'',  //当前所选项的名称合计，逗号隔开
      actors:[
        {id:'',name:'',type:'user'},...
      ]
  }
    ```

- cancel 取消按钮点击事件

