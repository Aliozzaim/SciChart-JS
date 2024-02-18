<template>
  <div style="font-size: 40px; margin-bottom: 250px">
    <a
      style="margin-bottom: 25px; display: block"
      href="https://www.aliozzaim.com/"
      >Aliozzaim.com</a
    >
    <div class="chart-container">
      <div
        id="scichart-root"
        ref="chartRoot"
        style="width: 1500px; height: 800px; margin: auto"
      ></div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue"
import {
  SciChartSurface,
  NumericAxis,
  FastLineRenderableSeries,
  FastColumnRenderableSeries,
  XyDataSeries,
  EAxisAlignment,
  NumberRange,
  MouseWheelZoomModifier,
  ZoomPanModifier,
  ZoomExtentsModifier,
  DateTimeNumericAxis,
} from "scichart"
import jsonData from "../components/data.json"

async function initSciChart(chartRoot) {
  const data = jsonData

  const { sciChartSurface, wasmContext } = await SciChartSurface.create(
    chartRoot
  )

  // Get the height of the chart dynamically
  const chartHeight = chartRoot.clientHeight

  // Find the newest data timestamp
  const newestTimestamp = Math.max(...data.map((item) => item.dt * 1000))

  // Calculate the timestamp for 3 hours ago
  const threeHoursAgo = newestTimestamp - 3 * 60 * 60 * 1000

  // Filter the data for timestamps within the last 3 hours
  const filteredData = data.filter((item) => item.dt * 1000 >= threeHoursAgo)

  // Extract timestamps, prices, and amounts from filtered data
  const timestamps = filteredData.map((item) => item.dt)
  const prices = filteredData.map((item) => item.price)
  const amounts = filteredData.map((item) => item.amount)

  // Create DateTimeNumericAxis
  const minDate = new Date(Math.min(...timestamps) * 1000)
  const maxDate = new Date(Math.max(...timestamps) * 1000)
  const xAxis = new DateTimeNumericAxis(wasmContext, {
    axisTitle: "Time",
    axisTitleStyle: {
      fontSize: 40,
      fontFamily: "sans",
      color: "#ffffff",
      fontWeight: "bold",
    },
    visibleRange: new NumberRange(
      minDate.getTime() / 1000,
      maxDate.getTime() / 1000
    ),
    labelStyle: {
      fontSize: 15,
      color: "#ffffff",
      fontFamily: "sans",
    },
  })
  sciChartSurface.xAxes.add(xAxis)

  const yAxisPrice = new NumericAxis(wasmContext)
  yAxisPrice.axisTitle = "Price"
  yAxisPrice.labelStyle = {
    fontSize: 20,
    color: "#FF0000",
    fontWeight: "bold",
    fontFamily: "sans",
  }
  yAxisPrice.axisTitleStyle = {
    fontSize: 40,
    fontFamily: "sans",
    color: "#FF0000",
    fontWeight: "bold",
  }
  yAxisPrice.axisAlignment = EAxisAlignment.Left
  yAxisPrice.growBy = new NumberRange(0.1, 0.1)
  sciChartSurface.yAxes.add(yAxisPrice)

  const maxColumnHeight = chartHeight * 50

  // Create secondary Y axis for Amount

  const yAxisAmount = new NumericAxis(wasmContext)
  ;(yAxisAmount.axisTitle = "Amount"),
    (yAxisAmount.axisTitleStyle = {
      fontSize: 40,
      fontFamily: "sans",
      color: "#0000FF",
      fontWeight: "bold",
    }),
    (yAxisAmount.labelStyle = {
      fontSize: 20,
      color: "#0000FF",
      fontWeight: "bold",
      fontFamily: "sans",
    }),
    (yAxisAmount.axisAlignment = EAxisAlignment.Right)
  yAxisAmount.growBy = new NumberRange(0, 1)
  yAxisAmount.visibleRange = new NumberRange(0, maxColumnHeight)
  yAxisAmount.id = "yAxisAmount"
  sciChartSurface.yAxes.add(yAxisAmount)

  const priceDataSeries = new XyDataSeries(wasmContext, {
    xValues: timestamps,
    yValues: prices,
  })
  const lineSeries = new FastLineRenderableSeries(wasmContext, {
    stroke: "red",
    strokeThickness: 3,

    dataSeries: priceDataSeries,
  })
  sciChartSurface.renderableSeries.add(lineSeries)

  const amountDataSeries = new XyDataSeries(wasmContext, {
    xValues: timestamps,
    yValues: amounts.map((amount) => Math.min(amount, maxColumnHeight)),
  })
  const columnSeries = new FastColumnRenderableSeries(wasmContext, {
    fill: "blue",
    dataSeries: amountDataSeries,

    yAxisId: "yAxisAmount",
  })
  sciChartSurface.renderableSeries.add(columnSeries)

  sciChartSurface.chartModifiers.add(
    new MouseWheelZoomModifier(),
    new ZoomPanModifier(),
    new ZoomExtentsModifier()
  )

  return sciChartSurface
}

export default defineComponent({
  mounted() {
    initSciChart(this.$refs.chartRoot)
  },
})
</script>

<style scoped>
.chart-container {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
  touch-action: manipulation;
}
</style>
