<template>
  <div class="container">

  </div>
</template>


<script>
import * as d3 from 'd3'
export default {
  name: 'HelloWorld',
  data () {
    return {
      links:[],
      nodes:[],
      nodeNameText:[],
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
  mounted(){
    this.getGraphData()
  },
  methods:{
    getGraphData(){
      var _this = this;
      this.axios.get("api/person/Tom Cruise" )
      .then(function(response){ 
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
            .force("link", d3.forceLink(links).id(d => d.id).distance(100))
            .force("collide",d3.forceCollide().radius(()=>30))
            .force("charge", d3.forceManyBody().strength(-30))
            .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));

      //画布
        // const svg = d3.create("svg")
        //     .attr("viewBox", [0, 0, width, height]);
        const svg = d3.select(".container")
        .append("svg")
        .attr("viewBox", [0, 0, _this.width, _this.height])
        .call(d3.zoom().on("zoom",function(){
          g.attr("transform",d3.event.transform)
        }))
      //link属性
        const g = svg.append("g")

        _this.links = g.append("g")
            .attr("stroke", "#999")
            .attr("stroke-opacity", 0.6)
          .selectAll("line")
          .data(links)
          .join("line")
            .attr("stroke-width", d => Math.sqrt(d.value))
            .attr("class","link");


      //node属性
        _this.nodes = g.append("g")
          .selectAll("circle")
          .data(nodes)
          .join("circle")
            .attr("r", 30)
            .attr("class","node")
            .attr("fill", _this.color)
            .on("click",_this.queryTest)
            .call(_this.drag(_this.simulation));



        _this.nodes.append("title")
            .text(d => d.id);

        _this.nodeNameText = g.append("g")
        .selectAll("text")
        .data(nodes)
        .join("text")
        .text(function(d){
            return d.id;
        })
        .attr("dx",function(){
            return this.getBoundingClientRect().width/2*(-1)
        })
        .attr("dy",50)
        .attr("class","nodeName")
        
        

        _this.simulation.on("tick", () => {
          _this.links
              .attr("x1", d => d.source.x)
              .attr("y1", d => d.source.y)
              .attr("x2", d => d.target.x)
              .attr("y2", d => d.target.y);

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

        //关系
        _this.links = _this.links
        .data(links)
        .enter()
        .append("line")
        .attr("stroke","#999")
        .attr("stroke-opacity",0.6)
        .attr("stroke-width",d=>Math.sqrt(d.value))
        .merge(_this.links)
        .attr("class","links");
        //node属性
        _this.nodes = _this.nodes
          .data(nodes)
          .enter()
          .append("circle")
          .attr("r", 30)
          .attr("class","node")
          .attr("fill", _this.color)
          .merge(_this.nodes)
          .on("click",_this.queryTest)
          .call(_this.drag(_this.simulation));

        _this.nodes.append("title")
            .text(d => d.id);

        //名字
        _this.nodeNameText = _this.nodeNameText
        .data(nodes)
        .enter()
        .append("text")
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
        _this.simulation.alpha(1).restart()

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
          if (!d3.event.active) simulation.alphaTarget(0);
          d.fx = null;
          d.fy = null;
        }

        return d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended);
    },
    queryTest(d){
        var _this = this;
        console.log(d);
      this.axios.get("api/person/actedby/" + d.id)
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
                    "value":5
                })
            }
         }

         _this.updateGraph(_this.testGraph);
        }

      })
      .catch(function(error){
        console.log(error)
      })
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
