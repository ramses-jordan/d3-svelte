<script>
  import * as d3 from "d3";
  import raw_data from "./capacity_status.json";
  import DataContainer from "@snlab/florence-datacontainer";
  import {
    Graphic,
    Section,
    RectangleLayer,
    Label,
    XAxis,
    DiscreteLegend,
    YAxis,
    y2s,
    Rectangle,
    y2,
  } from "@snlab/florence";

  const data = new DataContainer(raw_data);
  const regions = data
    .groupBy("Region")
    .summarise({ total_kwh: { "Capacity (MW)": "sum" } });

  const getStatusBreakdown = (region) => {
    const regionStatuses = data
      .filter((row) => row["Region"] === region)
      .groupBy("Status")
      .summarise({ total_kwh: { "Capacity (MW)": "sum" } });

    return {
      region,
      statuses: regionStatuses.column("Status"),
      total_kwh: regionStatuses.column("total_kwh"),
      colors: [
        "#FFB100",
        "#A3BB98",
        "#1C315E",
        "#E6E2C3",
        "#FBC252",
        "#FF6961",
        "#F0ECCF",
        "#88A47C",
      ],
    };
  };
  let statusBreakdowns = []
  regions.column('Region').forEach(region => {
	statusBreakdowns.push(getStatusBreakdown(region))
  });

  const previousY = (list, i) =>
    list.slice(0, i).reduce((acc, v) => acc + v, 0);

  let tooltipData = {
    visible: false,
    xPos: 0,
    yPos: 0,
    total_kwh: 0,
  };

  const showRegionTooltip = (event) => {
    const { index, hitBbox } = event;
    const { maxX, minY, maxY } = hitBbox;
    tooltipData.visible = true;
    tooltipData.xPos = maxX + 40;
    tooltipData.yPos = minY - (minY - maxY) / 2;
    tooltipData.total_kwh = regions.row({ index })["total_kwh"];
    document.body.style.cursor = "pointer";
  };
  const hideRegionTooltip = () => {
    tooltipData.visible = false;
    document.body.style.cursor = "default";
  };
</script>

<div>
  <p
    id="tooltip"
    style="visibility: {tooltipData.visible
      ? 'visible'
      : 'hidden'}; top:{tooltipData.yPos}px; left: {tooltipData.xPos}px;"
  >
    {tooltipData.total_kwh} ADD UNIT OF MEASUREMENT
  </p>
  <Graphic width={925} height={825}>
    <Section
      x1={0.15}
      x2={0.95}
      y1={0.05}
      y2={0.95}
      scaleX={d3
        .scaleLinear()
        .domain([0, Math.max(...regions.domain("total_kwh"))])
        .nice()}
      scaleY={d3.scaleBand().domain(regions.domain("Region")).padding(0.1)}
      flipY
    >
      <RectangleLayer
        x1={Array(regions.column("total_kwh").length).fill(0)}
        x2={regions.column("total_kwh")}
        y2={regions.column("Region")}
        y1={y2s(regions.column("Region"))}
        fill='#227C70'
        onMouseover={showRegionTooltip}
        onMouseout={hideRegionTooltip}
      />
      {#each statusBreakdowns as breakdown }
        {#each breakdown.statuses as status, i}
          <p>{breakdown.region}</p>
          <Rectangle
            y1={breakdown.region}
            y2={y2(breakdown.region)}
            x1={previousY(breakdown.total_kwh, i)}
            x2={previousY(breakdown.total_kwh, i) +
              breakdown.total_kwh[i]}
            fill={breakdown.colors[i]}
          />
        {/each}
      {/each}
      <XAxis title="Idk what you said :(" />
      <YAxis title="Regions" />
    </Section>
  </Graphic>
</div>

<style>
  div {
    font-family: Helvetica, Arial;
  }
  #tooltip {
    position: absolute;
    padding: 0;
    margin: 0;
    z-index: 2;
  }
</style>
