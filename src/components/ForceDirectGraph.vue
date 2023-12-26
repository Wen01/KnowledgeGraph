<template>
  <div ref="container" />
</template>

<script>
import * as d3 from 'd3'
export default {
  props: {
    searchWord: String
  },
  data() {
    return {
      graphData: {
        nodes: [],
        edges: []
      },
      color: ['#40E0D0', '#FFD700'],
      width: '',
      height: ''
    }
  },
  watch: {
    searchWord(newVal) {
      console.log('change')
      if (newVal === 'https://api.ownthink.com/kg/knowledge') {
        // 导入json
        this.importData()
        this.drawDirectGraph()
        window.addEventListener('resize', this.handleResize)
      } else {
        this.graphData.nodes = []
        this.graphData.edges = []
        this.drawDirectGraph()
      }
    },
  },
  mounted() {
    this.width = window.innerWidth * 0.8
    this.height = window.innerHeight
  },
  methods: {
    handleResize() {
      this.width = window.innerWidth * 0.8
      this.height = window.innerHeight
      // 更新画布尺寸
      this.svg.attr('width', this.width).attr('height', this.height)
      // 更新力导向图模拟
      this.simulation.force('center', d3.forceCenter(this.width / 2, this.height / 2));
      this.simulation.restart();
    },
    importData() {
      const graph1 = require('../static/json/graph1.json')
      this.constructData(graph1)
    },
    constructData(jsonData) {
      if (!this.graphData.nodes.find(item => item.id === jsonData.id)) {
        this.graphData.nodes.push({ id: jsonData.id, name: jsonData.name })
      }
      const source = jsonData.id
      for (const node of jsonData.nodes) {
        this.graphData.nodes.push({ id: node.id, name: node.name })
        this.graphData.edges.push({ source: source, target: node.id, relation: node.relation })
      }
    },
    drawDirectGraph() {
      d3.select('#svgId').remove()
      this.svg = d3
        .select(this.$refs.container)
        .append('svg')
        .attr('id', 'svgId')
        .attr('viewbox', [0, 0, this.width, this.height])
        .attr('width', this.width)
        .attr('height', this.height)
        .style('background-color', 'transparent')
        .style('position', 'absolute')
        .style('left', window.innerWidth * 0.2 + 'px');

      // 设置力导向图
      this.simulation = this.setupSimulation(this.width, this.height)

      this.simulation.nodes(this.graphData.nodes).on('tick')
      this.simulation.force('link').links(this.graphData.edges).distance(150)
      // 绘制各个元素
      this.graphElements = this.drawGraphElements(this.svg, this.color)
      // 添加拖拽
      const drag = this.setupDrag(this.simulation)
      this.graphElements.node.call(drag)

      this.simulation.on('tick', () => {
        this.updateGraphElements(this.graphElements)
      })
    },
    setupSimulation(width, height) {
      return d3
        .forceSimulation()
        .force('link', d3.forceLink().id(d => d.id))
        .force('charge', d3.forceManyBody().strength(-90)) // 引力
        .force('collide', d3.forceCollide().radius(() => 90)) // 碰撞力
        .force('center', d3.forceCenter(width / 2, height / 2))
    },
    // 绘制各个元素
    drawGraphElements(svg) {
      const link = svg
        .selectAll('.link')
        .data(this.graphData.edges)
        .enter()
        .append('line')
        .style('stroke-width', 1)
        .style('stroke', '#F5F5F5')
        .style('opacity', 0.6)

      const node = svg
        .selectAll('.node')
        .data(this.graphData.nodes)
        .enter()
        .append('circle')
        .attr('r', 16)
        .style('fill', d => this.hasChildNodes(d) ? '#40E0D0' : '#FFFF66')
        .on('click', this.handleNodeClick)

      const edges_text = svg
        .selectAll('.linetext')
        .data(this.graphData.edges)
        .enter()
        .append('text')
        .attr('class', 'linetext')
        .text(d => d.relation)
        .style('stroke', 'white')
        .style('font-size', 12)

      const texts = svg
        .selectAll('.forceText')
        .data(this.graphData.nodes)
        .enter()
        .append('text')
        .style('stroke', '#F5F5F5')
        .style('font-size', '12px')
        .attr('text-anchor', 'middle')
        .attr('dy', 30)
        .text(d =>  d.name.length > 10 ? d.name.substring(0, 10) + '...' : d.name)

      return { link, node, edges_text, texts }
    },
    // 添加拖拽效果
    setupDrag(simulation) {
      const drag = d3
        .drag()
        .on('start', dragStarted)
        .on('drag', dragged)
        .on('end', dragEnded)

      return drag

      function dragStarted(event, d) {
        if(!event.active){//event.active 属性对判断并发的拖拽手势序列中的 start 事件和 end 事件: 在拖拽手势开始时为0，在拖拽结束最后一个手势事件时为0
          //这里就是drag的过程中
          simulation.alphaTarget(0.8).restart()//设置衰减系数，对节点位置移动过程的模拟，数值越高移动越快，数值范围[0，1]
        }
        d.fx = d.x
        d.fy = d.y
      }
      function dragged(event, d) {
        d.fx = event.x
        d.fy = event.y
      }
      function dragEnded(event, d) {
        if (!event.active) {
          simulation.alphaTarget(0)
        }
        d.fx = null
        d.fy = null
      }
    },
    // 更新图形元素
    updateGraphElements(graphElements) {
      graphElements.link.attr('x1', d => d.source.x)
        .attr('y1', d => d.source.y)
        .attr('x2', d => d.target.x)
        .attr('y2', d => d.target.y)
      graphElements.node.attr('cx', d => d.x).attr('cy', d => d.y);
      graphElements.edges_text.attr('x', d => (d.source.x + d.target.x) / 2)
        .attr('y', d => (d.source.y + d.target.y) / 2)
      graphElements.texts.attr('x', d => d.x)
        .attr('y', d => d.y)
    },
    handleNodeClick(event, clickNode) {
      let haveChildren = this.graphData.edges.some(edge => edge.source.id === clickNode.id)
      if (!haveChildren) this.addNodes(clickNode)
      else this.removeNodes(clickNode)
    },
    // 新增节点
    addNodes(currNode) {
      const nodeId = currNode.id
      const jsonData = require(`../static/json/graph${nodeId}.json`)
      // 更新节点和边数据
      this.constructData(jsonData)
      // 重新设置力导向图的模拟和图形元素
      this.simulation.nodes(this.graphData.nodes)
      this.simulation.force('link').links(this.graphData.edges).distance(150)
      // 重新启动力导向图模拟
      this.simulation.alpha(1).restart()
      // 移除旧的 SVG
      d3.select('#svgId').remove();

      // 重新创建 SVG
      this.svg = d3
        .select(this.$refs.container)
        .append('svg')
        .attr('id', 'svgId')
        .attr('viewbox', [0, 0, this.width, this.height])
        .attr('width', this.width)
        .attr('height', this.height)
        .style('background-color', 'transparent')
        .style('position', 'absolute')
        .style('left', window.innerWidth * 0.2 + 'px');

      // 重新绘制整个图形
      this.graphElements = this.drawGraphElements(this.svg, this.color)
      const drag = this.setupDrag(this.simulation)
      this.graphElements.node.call(drag)

      // 更新图形元素
      this.updateGraphElements(this.graphElements);
    },
    // 删除节点
    removeNodes(currNode) {
      const targetId = currNode.id
      const removeId = []
      for (let i = 0; i < this.graphData.edges.length;) {
        if (this.graphData.edges[i].source.id === targetId) {
          removeId.push(this.graphData.edges[i].target.id)
          this.graphData.edges.splice(i, 1)
        }
        else {
          i++
        }
      }
      for (let j = 0; j < removeId.length; j++) {
        for (let k = 0; k < this.graphData.nodes.length;) {
          if (this.graphData.nodes[k].id === removeId[j]) {
            this.graphData.nodes.splice(k, 1)
            break
          }
          k++
        }
      }
      // 重新设置力导向图的模拟和图形元素
      this.simulation.nodes(this.graphData.nodes)
      this.simulation.force('link').links(this.graphData.edges).distance(150)

      // 重新启动力导向图模拟
      this.simulation.alpha(1).restart()

      // 移除旧的 SVG
      d3.select('#svgId').remove();

      // 重新创建 SVG
      this.svg = d3
        .select(this.$refs.container)
        .append('svg')
        .attr('id', 'svgId')
        .attr('viewbox', [0, 0, this.width, this.height])
        .attr('width', this.width)
        .attr('height', this.height)
        .style('background-color', 'transparent')
        .style('position', 'absolute')
        .style('left', window.innerWidth * 0.2 + 'px');

      // 重新绘制整个图形
      this.graphElements = this.drawGraphElements(this.svg, this.color)
      const drag = this.setupDrag(this.simulation)
      this.graphElements.node.call(drag)

      // 更新图形元素
      this.updateGraphElements(this.graphElements);
    },
    hasChildNodes(node) {
      return this.graphData.edges.some(edge => edge.source.id === node.id)
    }
  }
}
</script>

<style>
</style>