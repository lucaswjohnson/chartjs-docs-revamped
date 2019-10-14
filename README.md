# chartjs-docs-revamped

I've been working with Chart.js, and have had way too much confusion reading and trying to figure out the docs, so here's some things I've learned.

**Remove Overflowing Grid Lines**

This removes overflowing grid lines on the x and y axes that is there by default
```
Chart.plugins.register({
  beforeDraw: (chart) => {
    const ctx = chart.chart.ctx

    const x_axis = chart.scales['x-axis-0']
    const topY = chart.scales['y-axis-0'].top
    const bottomY = chart.scales['y-axis-0'].bottom

    const y_axis = chart.scales['y-axis-0']
    const rightX = chart.scales['x-axis-0'].right
    const leftX = chart.scales['x-axis-0'].left

    x_axis.options.gridLines.display = false
    y_axis.options.gridLines.display = false

    x_axis.ticks.forEach((label, index) => {
      const x = x_axis.getPixelForValue(label)
      ctx.save()
      ctx.beginPath()
      ctx.moveTo(x, topY)
      ctx.lineTo(x, bottomY)
      ctx.lineWidth = 1
      ctx.strokeStyle = x_axis.options.gridLines.color
      ctx.stroke()
      ctx.restore()
    })

    y_axis.ticks.forEach((label, index) => {
      const y = y_axis.getPixelForValue(label)
      ctx.save()
      ctx.beginPath()
      ctx.moveTo(leftX, y)
      ctx.lineTo(rightX, y)
      ctx.lineWidth = 1
      ctx.strokeStyle = y_axis.options.gridLines.color
      ctx.stroke()
      ctx.restore()
    })
  }
})
```
