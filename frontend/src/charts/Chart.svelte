<script lang="ts">
  import { writable } from "svelte/store";

  import {
    barChartMode,
    chartToggledCurrencies,
    hierarchyChartMode,
    lineChartMode,
    showCharts,
    treemapCurrency,
  } from "../stores/chart";

  import BarChart from "./BarChart.svelte";
  import ChartLegend from "./ChartLegend.svelte";
  import HierarchyContainer from "./HierarchyContainer.svelte";
  import LineChart from "./LineChart.svelte";
  import ModeSwitch from "./ModeSwitch.svelte";
  import ScatterPlot from "./ScatterPlot.svelte";

  import type { FavaChart } from ".";

  /**
   * The chart to render.
   */
  export let chart: FavaChart;

  /**
   * Width of the chart.
   */
  let width: number;

  const treemap_currencies = writable<string[]>([]);
  const legend = writable<[string, string | null][]>([]);
</script>

<div class="flex-row">
  {#if $showCharts}
    {#if chart.type === "barchart" || chart.type === "linechart"}
      <ChartLegend legend={$legend} toggled={chartToggledCurrencies} />
    {/if}
    {#if chart.type === "hierarchy" && $hierarchyChartMode === "treemap"}
      <ChartLegend
        legend={$treemap_currencies.map((c) => [c, null])}
        active={treemapCurrency}
      />
    {/if}
    <span class="spacer" />
    {#if chart.type === "hierarchy"}
      <ModeSwitch store={hierarchyChartMode} />
    {:else if chart.type === "linechart"}
      <ModeSwitch store={lineChartMode} />
    {:else if chart.type === "barchart" && chart.hasStackedData}
      <ModeSwitch store={barChartMode} />
    {/if}
  {:else}<span class="spacer" />{/if}
  <slot />
  <button
    type="button"
    on:click={() => {
      showCharts.update((v) => !v);
    }}
    class:closed={!$showCharts}
    class="toggle-chart"
  />
</div>
<div hidden={!$showCharts} bind:clientWidth={width}>
  {#if width}
    {#if chart.type === "barchart"}
      <BarChart {chart} {width} {legend} />
    {:else if chart.type === "hierarchy"}
      <HierarchyContainer {chart} {width} {treemap_currencies} />
    {:else if chart.type === "linechart"}
      <LineChart {chart} {width} {legend} />
    {:else if chart.type === "scatterplot"}
      <ScatterPlot {chart} {width} />
    {/if}
  {/if}
</div>

<style>
  .toggle-chart {
    height: 22px;
    padding: 2px 6px;
    margin: 0;
  }

  .toggle-chart::before {
    display: block;
    content: "";
    border: 0;
    border-top: 13px solid var(--background);
    border-right: 9px solid transparent;
    border-left: 9px solid transparent;
  }

  .toggle-chart.closed::before {
    border: 0;
    border-right: 9px solid transparent;
    border-bottom: 13px solid var(--background);
    border-left: 9px solid transparent;
  }
</style>
