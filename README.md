# Chart.js Docs Revamped

I've been working with Chart.js, and have had way too much confusion reading and trying to figure out the docs, so here's some things I've learned.

---

## A Note on Chart globals (defaults)

To keep your code neat, I would recommend creating a separate file that you import into the files that contain Charts that you want to have those global defaults. If you have many globals and plugins, this should help reduce bloat in your files and make them more DRY.
```
// chartConfig.js
import Chart from 'chart.js'

// Chart globals
Chart.defaults.scale.ticks.beginAtZero = true
Chart.defaults.global.tooltips.yAlign = 'bottom'
...

// Chart plugins
Chart.plugins.register({
...
```
You can learn more about Chart.js globals [here.](https://www.chartjs.org/docs/latest/configuration/#global-configuration)

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

## Further customizable tooltip positioning

If you're using the default tooltip (not a custom HTML one) but want to be able to customize your tooltip positioning beyond the default options 'nearest' and 'average', you can create your own function to calculate the tooltip position.

The basic customization is documented [here](https://www.chartjs.org/docs/latest/configuration/tooltip.html#position-modes), but if you want to cusomtize the 'nearest' and 'average' positioning, or simply have access to the positioning data while using these position modes, you can use one of the custom functions below that do the same thing, but give you access to the code as it's executed.

Remember to put `options.tooltips.position = 'custom'` to make them work!

**nearest**
```
// Custom 'nearest' tooltip positioning. This is does the same thing as options.tooltips.position = 'nearest'
Chart.Tooltip.positioners.custom = function(elements, eventPosition) {
  var x = eventPosition.x;
  var y = eventPosition.y;

  var minDistance = Number.POSITIVE_INFINITY;
  var i, len, nearestElement;

  for (i = 0, len = elements.length; i < len; ++i) {
    var el = elements[i];
    if (el && el.hasValue()) {
      var center = el.getCenterPoint();
      // Note that Math.hypot is ES2015, so if that's not supported, calculate the distance using another method
      // https://stackoverflow.com/questions/20916953/get-distance-between-two-points-in-canvas
      var d = Math.hypot(center.x - eventPosition.x, center.y - eventPosition.y)

      if (d < minDistance) {
        minDistance = d;
        nearestElement = el;
      }
    }
  }

  if (nearestElement) {
    var tp = nearestElement.tooltipPosition();
    x = tp.x;
    y = tp.y;
  }

  return {
    x: x,
    y: y
  };
}
```

**average**
```
// Custom 'average' tooltip positioning. This is does the same thing as options.tooltips.position = 'average'
Chart.Tooltip.positioners.custom = function(elements) {
    if (!elements.length) {
        return false;
    }

    var i, len;
    var x = 0;
    var y = 0;
    var count = 0;

    for (i = 0, len = elements.length; i < len; ++i) {
        var el = elements[i];
        if (el && el.hasValue()) {
            var pos = el.tooltipPosition();
            x += pos.x;
            y += pos.y;
            ++count;
        }
    }

    return {
        x: x / count,
        y: y / count
    };
}
```
I got these functions from the Chart.js source code [here](https://codeclimate.com/github/chartjs/Chart.js/src/core/core.tooltip.js/source)

## Distance legends from the graph

If you'd like to distance the legends from the graph, you can use the code below, or there might be a better, less verbose solution [here.](https://stackoverflow.com/questions/42585861/chart-js-increase-spacing-between-legend-and-chart)

Just change the `this.height + 40` part to change the distance.
```
// Chart functions and plugin to distance legends from graphs
Chart.NewLegend = Chart.Legend.extend({
  afterFit: function () {
    this.height = this.height + 40 // change this to change the distance between the graph and the legends
  }
})

const createNewLegendAndAttach = (chartInstance, legendOpts) => {
  const legend = new Chart.NewLegend({
    ctx: chartInstance.chart.ctx,
    options: legendOpts,
    chart: chartInstance
  })

  if (chartInstance.legend) {
    Chart.layoutService.removeBox(chartInstance, chartInstance.legend)
    delete chartInstance.newLegend
  }

  chartInstance.newLegend = legend
  Chart.layoutService.addBox(chartInstance, legend)
}

// Register the legend plugin
Chart.plugins.register({
  beforeInit: (chartInstance) => {
    const legendOpts = chartInstance.options.legend

    if (legendOpts) {
      createNewLegendAndAttach(chartInstance, legendOpts)
    }
  },
  beforeUpdate: (chartInstance) => {
    let legendOpts = chartInstance.options.legend;

    if (legendOpts) {
      legendOpts = Chart.helpers.configMerge(Chart.defaults.global.legend, legendOpts)

      if (chartInstance.newLegend) {
        chartInstance.newLegend.options = legendOpts
      } else {
        createNewLegendAndAttach(chartInstance, legendOpts)
      }
    } else {
      Chart.layoutService.removeBox(chartInstance, chartInstance.newLegend)
      delete chartInstance.newLegend
    }
  },
  afterEvent: (chartInstance, e) => {
    const legend = chartInstance.newLegend
    if (legend) {
      legend.handleEvent(e)
    }
  }
})
```
