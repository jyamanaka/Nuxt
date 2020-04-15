<template>
  <div id="barChart">
    <div id="tooltip">
      <p>Month Year</p>
      <p>
        Water table:
        <span></span>
      </p>
    </div>
    <svg />
  </div>
</template>
<script>
import waterData from '~/assets/waterData.json'
import * as d3 from 'd3'
const filteredWaterData = waterData.filter(
  d => new Date(d.monthYear) >= new Date(1975, 0)
)
console.log('filtered', filteredWaterData)
export default {
  mounted() {
    this.buildBarChart()
  },
  methods: {
    buildBarChart: () => {
      const MARGINS = { left: 50, bottom: 50, right: 100, top: 0 }
      const h = 700
      const w = 3000
      const innerHeight = h - MARGINS.top - MARGINS.bottom
      const innerWidth = w - MARGINS.left - MARGINS.right
      d3.select('#barChart')
        .attr('width', w)
        .attr('height', h)
      const y = d3
        .scaleLinear()
        .range([innerHeight, 0])
        .domain([0, d3.max(filteredWaterData, d => d.mean_value)])
      const x = d3
        .scaleBand()
        .range([0, innerWidth])
        .domain(filteredWaterData.map(d => d.monthYear))
        .paddingInner(0)
        .align(0.5)
      const color = d3
        .scaleLinear()
        .range(['#2C3296', '#00FFEE'])
        .domain([0, 100])
      const svg = d3
        .select('#barChart svg')
        .attr('width', w)
        .attr('height', h)
        .append('g')
        .attr('transform', `translate(${MARGINS.left}, ${MARGINS.top})`)
      svg
        .append('g')
        .attr('class', 'axis')
        .attr('transform', `translate(0,${innerHeight})`)
        .call(
          d3
            .axisBottom(x)
            .tickValues(
              x.domain().filter(function(d, i) {
                return !(i % 50)
              })
            )
            .tickFormat(d => {
              return d3.timeFormat('%b-%d-%y')(new Date(d))
            })
        )
      const barGroups = svg
        .selectAll('.bar-group')
        .data(filteredWaterData)
        .enter()
        .append('g')
        .attr('class', 'bar-group')
        .attr('transform', d => `translate(${x(d.monthYear)},${y(0)})`)
        .each(function(d) {
          const heightBarGroup = innerHeight - y(d.mean_value)
          const numSquares = Math.ceil(heightBarGroup / x.bandwidth())
          const remainder = heightBarGroup % x.bandwidth()
          d3.select(this)
            .selectAll('.square')
            .data(d3.range(0, numSquares))
            .enter()
            .append('rect')
            .attr('class', 'square')
            .attr('y', (_d, i) => -1 * (1 + i) * x.bandwidth())
            .attr('width', x.bandwidth())
            .attr('height', (_d, i) =>
              i === numSquares.length - 1 ? remainder : x.bandwidth()
            )
            .attr('fill', (_d, i) => color(i))
        })
      barGroups.on('mouseenter', d => {
        const tooltip = d3
          .select('#barChart #tooltip')
          .style('visibility', 'visible')
        tooltip
          .transition()
          .style('left', () => `${x(d.monthYear) + MARGINS.left + 7.5}px`)
          .style('top', () => {
            console.log('d', d)
            return `${MARGINS.top + y(d.mean_value) - 20}px`
          })
        tooltip.select('p').html(d3.timeFormat('%B %Y')(new Date(d.monthYear)))
        tooltip.select('span').html(d.mean_value)
      })
    },
  },
}
</script>
<style>
#barChart .square {
  stroke: white;
  stroke-width: 0.5;
}
#barChart {
  position: relative;
}
#barChart #tooltip {
  visibility: hidden;
  text-align: left;
  font-size: 0.75rem;
  position: absolute;
  background: white;
  padding: 0.5rem;
  border: 1px solid #2c3296;
}
</style>
