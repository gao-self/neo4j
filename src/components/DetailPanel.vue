<template>
  <el-card class="box-card detail-panel" shadow="hover">
    <div slot="header" class="clearfix">
      <span>节点信息</span>
      <!-- <el-button style="float: right; padding: 3px 0" type="text">操作按钮</el-button> -->
    </div>
    <div class="text item">
      <table>
        <tbody>
          <tr v-for="(v,k) in currentNode">
            <td class="detail-info">{{k}}</td>
            <td class="detail-info">{{v}}</td>
          </tr>
        </tbody>
      </table>
      <el-form ref="form"  label-width="0px" v-show="ifShow">
        <el-form-item>
          <el-select v-model="currentType" placeholder="请选择查询关系">
            <el-option v-for="(type,i) in relationshipTypes" :label="type" :value="urls[i]" :key="type"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">查询关系</el-button>
        </el-form-item>
      </el-form>
    </div>
  </el-card>
</template>


<script>
 export default{
  name:"DetailPanel",
  data(){
    return{
      ifShow:false,
      currentNode:{},
      currentType:'',
      relationshipTypes:['ACTION_IN','DIRECTED','WROTE','PRODUCED','REVIEWED'],
      urls:['actedby','directed','wrote','produced','reviewed'],
    }
  },
  methods:{
      onSubmit(){
      var _this = this
      if(this.currentType === ""){
        this.$message.error('未选择任何类型');
      }

      this.axios.get("api/person/query/"+_this.currentType +"/"+ _this.currentNode.name)
      .then(function(response){
        if(response.status === 200){
          _this.$emit('update',response.data ,_this.currentNode,_this.currentType)
        }

      })
      .catch(function(error){
        console.log(error)
      })
    }
  },
}

</script>
<style scoped>
  .detail-panel{
    position: fixed;
    right:10px;
    top:40px;
    border-radius: 4px;
    width:200px;
    cursor:pointer;
  }

  .detail-info{
    text-align: left;
  }
</style>

<!-- onSubmit(){
      var _this = this
      console.log(this.currentType)
      if(this.currentType === ""){
        this.$message.error('未选择任何类型');
      }

      this.axios.get("api/person/actedby/" + _this.currentNode.name)
      .then(function(response){
        if(response.status === 200){
         for(var i=0;i<response.data.length;i++){
            let flag = true;
            for(var j=0;j<_this.testGraph.nodes.length;j++){
                if(_this.testGraph.nodes[j].id === response.data[i].id){
                    flag = false
                    break
                }
            }
            if(flag){
                _this.testGraph.nodes.push(response.data[i])
                _this.testGraph.links.push({
                    "source":d.id,
                    "target":response.data[i].id,
                    "value":5,
                    "relationship":"ACTED_IN"
                })
            }
         }
         console.log(_this.testGraph)

         _this.updateGraph(_this.testGraph);
        }

      })
      .catch(function(error){
        console.log(error)
      })
    } -->


