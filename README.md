# Chart.js Docs Revamped

I've been working with Chart.js, and have had way too much confusion reading and trying to figure out the docs, so here's some things I've learned.

---

## Remove Overflowing Grid Lines

This removes overflowing grid lines on the x and y axes that is there by default

(my own revision of [this Stack Overflow answer](https://stackoverflow.com/a/45592824/11786802))
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

## Distance the legends from the graph

If you find that the legends are too close to the graph, you can move them, but it's not very intuitive. Luckily it has been figured out by [jordanwillis](https://stackoverflow.com/users/7581592/jordanwillis) on [Stack Overflow](https://stackoverflow.com/a/42589310/11786802)

## Remove dataset toggle when clicking legend

This probably isn't something that needs to be mentioned, since it's not complex or unintuitive, but I figured I would add it here since it's something I needed to use.
```
Chart.defaults.global.legend.onClick = () => false
```

## Change default legend shapes

If you want the default legend rectangles to look like the points on your graphs, you can enable usePointStyle to have it mirror the shape of the points on your graph:

```
Chart.defaults.global.legend.labels.usePointStyle = true
```

There's a bunch of shapes you can choose from, all listed [here.](https://www.chartjs.org/docs/latest/configuration/elements.html#point-styles)

## Customizing the tooltip (and caret) positioning

The docs relating to customizing tooltips are confusing in my opinion, but luckily chaning the tooltip and caret positioning is quite easy with the default tooltip.

`yAlign` affects the position of the caret, meaning that `'bottom'` will put the caret below the tooltip. This will disable the automatic positioning of the tooltip, so it could potentially cause issues when getting near the edge of your graph.

`xAlign` affects the position of the caret on the x axis, relative to its `yAlign` positioning. This can be set without `yAlign` to force the caret positioning on the left or right, which will disable the automatic positioning as well.
```
Chart.defaults.global.tooltips.yAlign = 'bottom' // bottom, top
Chart.defaults.global.tooltips.xAlign = 'center' // left, right, center
```

If you're looking to center the tooltip in the middle of a bar graph, I suggest you take a look at [this Stack Overflow answer](https://stackoverflow.com/questions/45415925/position-tooltip-in-center-of-bar).
