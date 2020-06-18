<template>
  <div>
     <div class="container"></div>
     <detail-panel ref="detailPanel" @update="getQueryResult"></detail-panel>
  </div>
 
</template>


<script>
import * as d3 from 'd3'
import DetailPanel from "./DetailPanel";
export default {
  name: 'Canvas',
  components:{DetailPanel},
  data () {
    return {
      links:[],
      nodes:[],
      nodeNameText:[],
      linksName:[],
      simulation:null,
      width:800,
      height:500,
      colorList:["#ff1c12","#de5991","#759AA0"],
      testGraph:{
  "nodes": [],
  "links":[
     // {"source": "gao", "target": "John Hurt", "value": 5},
          ]
      },
    }
  },
  created(){
    this.getGraphData()
  },
  methods:{   
    getGraphData(){
      var _this = this;
      this.axios.get("api/person/Tom Hanks" )
      .then(function(response){ 
        console.log(response);
          _this.testGraph["nodes"] = response.data;
          _this.initGraph(_this.testGraph);
      })
      .catch(function(error){
        console.log(error)
      })
    },

    initGraph(data){
        var _this = this;
      //转换为对象 
        const links = data.links.map(d => Object.create(d));
        const nodes = data.nodes.map(d => Object.create(d));

      //模拟仿真
        _this.simulation = d3.forceSimulation(nodes)
            .force("link", d3.forceLink(links).id(d => d.id).distance(200))
            .force("collide",d3.forceCollide().radius(()=>30)) //碰撞半径
            .force("charge", d3.forceManyBody().strength(-10))  //-斥力，+吸引力
            .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));

      //画布
        // const svg = d3.create("svg")
        //     .attr("viewBox", [0, 0, width, height]);
        const svg = d3.select(".container")
        .append("svg")
        .attr("viewBox", [0, 0, _this.width, _this.height]) //范围
        .call(d3.zoom().on("zoom",function(){   //放缩
          g.attr("transform",d3.event.transform)
        }))
    //箭头
    const positiveMarker = svg.append("marker")
        .attr("id","positiveMarker")
        .attr("orient","auto") //自动调整方向
        .attr("stroke-width",2) //箭头的粗细
        .attr("markerUnits","strokeWidth")  //匹配粗细
        .attr("markerUnits","userSpaceOnUse")
        .attr("viewBox","0 -5 10 10") //箭头所在可视范围
        .attr("refX",35) //偏移
        .attr("refY",0)
        .attr("markerWidth",12)
        .attr("markerHeight",12)
        .append("path")
        .attr("d","M 0 -5 L 10 0 L 0 5")
        .attr("fill",'#999')
        .attr("stroke-opacity",0.6);

     const negativeMarker = svg.append("marker")
        .attr("id","negativeMarker")
        .attr("orient","auto") //自动调整方向
        .attr("stroke-width",2) //箭头的粗细
        .attr("markerUnits","strokeWidth")  //匹配粗细
        .attr("markerUnits","userSpaceOnUse")
        .attr("viewBox","0 -5 10 10") //箭头所在可视范围
        .attr("refX",-25) //偏移
        .attr("refY",0)
        .attr("markerWidth",12)
        .attr("markerHeight",12)
        .append("path")
        .attr("d","M 10 -5 L 0 0 L 10 5")
        .attr("fill",'#999')
        .attr("stroke-opacity",0.6);
      //link属性
        const g = svg.append("g")

        _this.links = g.append("g")
            .attr("stroke", "#999")
            .attr("stroke-opacity", 0.6)
            .attr("marker-end","url(#direction)")
            .selectAll("path")
            .data(links,function(d){
             if(typeof(d.source) === 'object'){
              return d.source.id+"_"+d.relationship+"_"+d.target.id
            }
            else{
              return d.source+"_"+d.relationship+"_"+d.target
            }
          })
            .join("path")
            .attr("stroke-width", d => Math.sqrt(d.value))
            .attr("class","link")
            .attr("id",function(d){
               if(typeof(d.source) === 'object'){
                  return d.source.id+"_"+d.relationship+"_"+d.target.id
                 }
                else{
                   return d.source+"_"+d.relationship+"_"+d.target
                 }
            })
        //linksName
        _this.linksName = g.append("g")
        .selectAll("text")
        .data(links,function(d){
           if(typeof(d.source) === 'object'){
              return d.source.id+"_"+d.relationship+"_"+d.target.id
            }
            else{
              return d.source+"_"+d.relationship+"_"+d.target
            }

        })
        .join("text")
        .style('text-anchor','middle')
        .style('fill','white')
        .style('font-size','10px')
        .style('font-weight','bold')
        .append('textPath')
        .attr(
            'xlink:href', d=>"#"+d.source+"_"+d.relationship+"_"+d.target
        )
        .attr('startOffset','50%')
        .text(function(d){
            return d.relationship
        });


      //node属性
        _this.nodes = g.append("g")
          .selectAll("circle")
          .data(nodes,d=>d.id)
          .join("circle")
            .attr("r", 30)
            .attr("class","node")
            .attr("fill", _this.color)
            .on("click",_this.select)
            .call(_this.drag(_this.simulation));



        _this.nodes.append("title")
            .text(d => d.id);

        //nodeName
        _this.nodeNameText = g.append("g")
        .selectAll("text")
        .data(nodes)
        .join("text")
        .text(d=>d.id)
        .attr("dx",function(){
            return this.getBoundingClientRect().width/2*(-1)
        })
        .attr("dy",50)
        .attr("class","nodeName")

        

        

        _this.simulation.on("tick", () => {
          // _this.links.attr("d",d=>"M" + d.source.x + " " + d.source.y + "L" +  d.target.x + " " + d.target.y);
          //path

          //line
              // .attr("x1", d => d.source.x)
              // .attr("y1", d => d.source.y)
              // .attr("x2", d => d.target.x)
              // .attr("y2", d => d.target.y);
          _this.links
          .attr("d",function(d){
            if(d.source.x<d.target.x){
                return "M" + d.source.x +" " + d.source.y + "L" +d.target.x + " " + d.target.y
            }else{
                return "M" + d.target.x +" " + d.target.y + "L" +d.source.x + " " + d.source.y
            }
          })
          .attr("marker-end",function(d){
                if(d.source.x<d.target.x){
                    return "url(#positiveMarker)"
                }
                else{
                    return null
                }
          })
          .attr("marker-start",function(d){
            if(d.source.x<d.target.x){
                return null
            }
            else{
                return "url(#negativeMarker)"
            }
          })

          _this.nodes
              .attr("cx", d => d.x)
              .attr("cy", d => d.y);

        _this.nodeNameText
        .attr("x",d => d.x)
        .attr("y",d => d.y)
        });
    },

    updateGraph(data){
        var _this = this;
        // const links = data.links.map(d=>Object.create(d));
        // const nodes = data.nodes.map(d=>Object.create(d));
        const links = data.links;
        const nodes = data.nodes;

        //删除关系
        // _this.links = _this.links
        // .data(links,function(d){
        //    if(typeof(d.source) === 'object'){
        //       return d.source.id+"_"+d.relationship+"_"+d.target.id
        //     }
        //     else{
        //       return d.source+"_"+d.relationship+"_"+d.target
        //     }
        // })
        // .exit()
        // .remove()


        //关系
        _this.links = _this.links
        .data(links,function(d){
           if(typeof(d.source) === 'object'){
              return d.source.id+"_"+d.relationship+"_"+d.target.id
            }
            else{
              return d.source+"_"+d.relationship+"_"+d.target
            }
        })
        .join("path")
        .attr("stroke","#999")
        .attr("stroke-opacity",0.6)
        .attr("stroke-width",d=>Math.sqrt(d.value))
        .attr("marker-end","url(#direction)")
        .merge(_this.links)
        // .attr("id",d=>d.source+"_"+d.relationship+"_"+d.target)
        .attr("id",function(d){
            if(typeof(d.source) === 'object'){
              return d.source.id+"_"+d.relationship+"_"+d.target.id
            }
            else{
              return d.source+"_"+d.relationship+"_"+d.target
            }
        })
        .attr("class","links");

        //删除名字
        //  _this.linksName
        // .data(links,function(d){
        //    if(typeof(d.source) === 'object'){
        //       return d.source.id+"_"+d.relationship+"_"+d.target.id
        //     }
        //     else{
        //       return d.source+"_"+d.relationship+"_"+d.target
        //     }
        // })
        // .exit()
        // .remove()

         //linksName
        _this.linksName = _this.linksName
       .data(links,function(d){
           if(typeof(d.source) === 'object'){
              return d.source.id+"_"+d.relationship+"_"+d.target.id
            }
            else{
              return d.source+"_"+d.relationship+"_"+d.target
            }
        })
        .join("text")
        .style('text-anchor','middle')
        .style('fill','white')
        .style('font-size','10px')
        .style('font-weight','bold')
        .append('textPath')
        .attr(
          'xlink:href', function(d){
            if(typeof(d.source) === 'object'){
              return "#" + d.source.id+"_"+d.relationship+"_"+d.target.id
            }
            else{
              return "#" +d.source+"_"+d.relationship+"_"+d.target
            }
          }
          )
        .attr('startOffset','50%')
        .merge(_this.linksName)
        .text(function(d){
            return d.relationship
        });
        //node属性
        _this.nodes = _this.nodes
          .data(nodes)
          .join("circle")
          .attr("r", 30)
          .attr("class","node")
          .attr("fill", _this.color)
          .merge(_this.nodes)
          .on("click",_this.select)
          .call(_this.drag(_this.simulation));

        _this.nodes.append("title")
            .text(d => d.id);

        //名字
        _this.nodeNameText = _this.nodeNameText
        .data(nodes)
        .join("text")
        .merge(_this.nodeNameText)
        .text(function(d){
            return d.id;
        })
        .attr("dx",function(){
            return this.getBoundingClientRect().width/2*(-1)
        })
        .attr("dy",50)
        .attr("class","nodeName")

        

        _this.simulation.nodes(nodes)
        _this.simulation.force("link").links(links)
        _this.simulation.alpha(0.5).restart()  

    },

    color(d){
       // const scale = d3.scaleOrdinal(d3.schemeCategory10);
       // return scale(d.group);
       return this.colorList[d.group]
    },
    drag(simulation){
       function dragstarted(d) {
          if (!d3.event.active) simulation.alphaTarget(0.3).restart();
          d.fx = d.x;
          d.fy = d.y;
        }

        function dragged(d) {
          d.fx = d3.event.x;
          d.fy = d3.event.y;
        }

        function dragended(d) {
          if (!d3.event.active) simulation.alphaTarget(0.5);
          d.fx = null;  //注释后，拉着就停了
          d.fy = null;
        }

        return d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended);
    },
    select(d){
        var _this = this
        let data = {}
        for(var i in d.obj){
          let ifArray = d.obj[i] instanceof Array
          if(!ifArray){
            data[i] = d.obj[i]
          }
        }
        _this.$refs.detailPanel.currentNode = data
        _this.$refs.detailPanel.ifShow = true
    },
    getQueryResult(result,currentNode,currentType){
          for(var i=0;i<result.length;i++){  //result:查询得到的节点组
            let flag = true;
            let tempLinks = {
                "source":currentNode.name,
                "target":result[i].id,
                "value":5,
                "relationship":currentType
            }
            for(var j=0;j<this.testGraph.nodes.length;j++){
                if(this.testGraph.nodes[j].id === result[i].id){
                    flag = false
                    break
                }
            }
            if(flag){
                this.testGraph.nodes.push(result[i])
            }
            else{
              console.log("已存在的节点")
              console.log(result[i])
            }
            this.testGraph.links.push({
                    "source":currentNode.name,
                    "target":result[i].id,
                    "value":5,
                    "relationship":currentType
            })
         }
         for(var i=this.testGraph.links.length-1;i>=0;i--){
          if(this.testGraph.links[i].source.id === currentNode.name && this.testGraph.links[i].relationship !== currentType){
            let ifRemove = true;
            for(var k=0;k<result.length;k++){
              if(result[k].id === this.testGraph.links[i].target.id){
                ifRemove = false
                break
              }
            }
            if(ifRemove){
              for(var j=this.testGraph.nodes.length-1;j>=0;j--){
                if(this.testGraph.nodes[j].id === this.testGraph.links[i].target.id){
                  this.testGraph.nodes.splice(j,1)
                }
              }
            }



            this.testGraph.links.splice(i,1)
          }
         }
         this.updateGraph(this.testGraph)
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  body{
    margin: 0px;
  }
.container{
  width:800px;
  height: 500px;
  border:1px solid #2c3e50;
  border-radius:8px;
  margin-top: 40px;
  margin-left:auto;
  margin-right:auto;
  background: #154360 repeating-linear-gradient(30deg,
  hsla(0,0%,100%,.1),hsla(0,0%,100%,.1)15px,
  transparent 0,transparent 30px);
}
.node{
  stroke:#fff;
  stroke-width: 1;
  cursor: pointer;
}

.node:hover{
    stroke-width:5;
}
 
.nodeName{
    fill: white;
}
</style>
