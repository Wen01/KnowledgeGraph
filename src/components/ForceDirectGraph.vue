<template>
  <div ref="container" />
</template>

<script>
import * as d3 from 'd3'
export default {
  data() {
    return {
      graphData: {
        nodes: [
          { id: 1, name: 'A' },
          { id: 2, name: 'B' },
          { id: 3, name: 'C' },
          { id: 4, name: 'D' },
          { id: 5, name: 'E' },
          { id: 6, name: 'F' }
        ],
        edges: [
          { source: 1, target: 2, relation: 'AB', value: 100 },
          { source: 1, target: 3, relation: 'AC', value: 50 },
          { source: 1, target: 4, relation: 'AD', value: 10 },
          { source: 1, target: 5, relation: 'AE', value: 1000 },
          { source: 1, target: 6, relation: 'AF', value: 10000 }
        ],
      },
      color: ['#40E0D0', '#FFD700']
    }
  },
  mounted() {
    this.drawDirectGraph()
  },
  methods: {
    drawDirectGraph() {
      const width = 1400
      const height = 600
      d3.select('#svgId').remove()
      this.svg = d3
        .select(this.$refs.container)
        .append('svg')
        .attr('id', 'svgId')
        .attr('viewbox', [0, 0, width, height])
        .attr('width', width)
        .attr('height', height)
        .style('background-color', 'transparent');

      // 设置力导向图
      this.simulation = this.setupSimulation(width, height)

      this.simulation.nodes(this.graphData.nodes).on('tick')
      this.simulation.force('link').links(this.graphData.edges).distance(150)

      this.graphElements = this.drawGraphElements(this.svg, this.color)
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
        .text(d => d.name)

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
      let needAdd = true
      for (const link of this.graphData.edges) {
        if (link.source.id === clickNode.id) {
          needAdd = false
          break
        }
      }
      if (needAdd) this.addNodes(clickNode)
      else this.removeNodes(clickNode)
    },
    // 新增节点
    addNodes(currNode) {
      if (currNode.id !== 6) {
        return
      }
      const newData = {
        nodes: [
          { id: 7, name: 'G' },
          { id: 8, name: 'H' }
        ],
        edges: [
          { source: 6, target: 7, relation: 'FG' },
          { source: 6, target: 8, relation: 'FH' }
        ]
      }
      // 更新节点和边数据
      this.graphData.nodes.push(...newData.nodes)
      this.graphData.edges.push(...newData.edges)

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
        .attr('viewbox', [0, 0, 1400, 600])
        .attr('width', 1400)
        .attr('height', 600)
        .style('background-color', 'transparent');

      // 重新绘制整个图形
      this.graphElements = this.drawGraphElements(this.svg, this.color);

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
        .attr('viewbox', [0, 0, 1400, 600])
        .attr('width', 1400)
        .attr('height', 600)
        .style('background-color', 'transparent');

      // 重新绘制整个图形
      this.graphElements = this.drawGraphElements(this.svg, this.color);

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