<script>
    import { scaleLinear } from "d3-scale"; // for scaling data to graph the data points
    import jsondata from "../python/data-json/2023-ridership.json"; // import data
    import { line, curveNatural } from "d3";
    import "../assets/global-styles.css";
   /*
    export let mon; //get the selected variable name for 2021
    export let tue; //get the selected variable name for 2021
    export let colour96; // set lineColour[transitName] for 1996
    export let colour21; // set lineColour[transitName] for 2021
    export let transitName;
    */
    export let transitName
    export let ridershiptype

    let avg = `Average-${ridershiptype}`
    let sat = `Saturday-${ridershiptype}`
    let sun = `Sunday-${ridershiptype}`
    console.log(sun)
    
    const lineColour = {
      "淡水信義線": "#e3002c",
      "松山新店線": "#008659",
      "中和新蘆線": "#f8b61c",
      "板南線": "#0070bd",
      "環狀線": "#ffdb00",
      "新北投支線": "#fd92a3",
      "小碧潭支線": "#cfdb00",
      "文湖線": "#c48c31"
    };
  
    /* ======= SETTING UP DATA ==========*/
  
    //filtre data.json to the selected transitName in page.svelte
    function filterDesignation(jsondata) {
      return jsondata["Route"] === transitName;
    }
    // store filtered jsondata in variable called data
    var data = jsondata.filter(filterDesignation);
  
    /* ======= SETTING UP CHARTING AREAS ==========*/
    const padding = { top: 20, right: 25, bottom: 10, left: 150 };
    let width = 600; // set the base width
    let height = data.length * 5 + padding.top + padding.bottom; // set the base width
  
    // adjust the width of the graph based on viewing size
    $: innerWidth = width - (padding.left + padding.right);
    $: innerHeight = height - (padding.top + padding.bottom);
  
    /* ======= SETTING UP X AND Y AXIS ========== */
  
    // ------ SETTING UP Y AXIS ------------------------
    // create yTicks
    const yTicks = data.map(function (obj) {
      return obj["Code"];
    });
  
    // ------ SETTING UP X AXIS ------------------------
    // round the numbers for creating xTicks,
    // create xticks using the newMax values
    var xTicks = [0, 5000, 10000, 20000, 30000, 40000, 50000, 60000, 70000, 150000];
  
    // converts thousands and million to K and M i.e. (1,000 ==> 1K , 1,000,000 ==> 1M)
    function thousandToK(tick) {
      var newtick;
      if (tick < 400) {
        newtick = tick;
      } else if (tick >= 1000 && tick < 1000000) {
        newtick = tick / 1000 + "K";
      } else if (tick > 1000000) {
        newtick = tick / 1000000 + "M";
      } else {
        newtick = tick;
      }
      return newtick;
    }
  
    // ------SETTING UP BARS FOR THE BAR CHART ------------------------
    var barPadding = 1.5; // controls how much spacing the bars will be from the
    // control barwidth based on viewing window width
    $: barWidth = Math.max(Math.min(innerHeight / yTicks.length, 25), 6);
  
    /* ======= SCALING DATA POINTS ========== */
    $: xScale = scaleLinear()
      .domain([0, Math.max.apply(null, xTicks)])
      .range([padding.left, innerWidth + padding.left - padding.right]);
  
    $: yScale = scaleLinear()
      .domain([0, yTicks.length])
      .range([padding.top + 10, height - padding.bottom - 2 * barWidth]); // added 0.2*height to accomodate station text label.
    
    /* ==== SCALING THE LINES === */
    /*
    $: line_gen96 = line()
      .curve(curveNatural)
      .x((d) => xScale(d[mon]))
      .y((d, i) => yScale(i) + barPadding + padding.top)(data);
  
    $: line_gen21 = line()
      .curve(curveNatural)
      .x((d) => xScale(d[sun]))
      .y((d, i) => yScale(i) + barPadding + padding.top)(data);
    */
    /* ======= SET UP DATA LABELLING ========== */
    let selected_datapoint = undefined;
    let selected_datapoint_i = undefined;
  
  
    /* MOUSEOVER EVENT */
    let mouse_x, mouse_y;
  
    const setMousePosition = function (event) {
      mouse_x = event.clientX;
      mouse_y = event.clientY;
    };
  
    var axis = padding.left / 2.5;
  </script>
  
  <div
    id="barchart"
    class="chart"
    bind:clientHeight={height}
    bind:clientWidth={width}
  >
    <svg
      height={(data.length + 2) * (barWidth + barPadding) +
        padding.top +
        padding.bottom}
      {width}
    >
      <!-- TEXT LABEL FOR THE STATION NAMES-->
      <g>
        {#each data as point, i}
            <g class="station-tick">
              <text
                x={padding.left - 22}
                y={yScale(i) + barPadding + padding.top + 5}
                text-anchor="end"
                font-size="14"
              >
                {point["Station"]}
              </text>
            </g>
        {/each}
      </g>
      <!--
      <path d={line_gen96} stroke={"white"} stroke-dasharray="6 3" />
      <path d={line_gen21} stroke={"white"} />-->
      <!-- CREATE THE LINES ON THE LINE GRAPH-->
  
      {#each data as point, i}
        <!-- CREATE THE DOTS ON THE LINE GRAPH-->
        <!-- {#if point[mon] !== null}
          <circle
            class="point"
            r={2}
            cx={xScale(point[mon])}
            cy={yScale(i) + barPadding + padding.top}
            fill={colour96}
          />
          <circle
            class="point"
            r={2}
            cx={xScale(point[tue])}
            cy={yScale(i) + barPadding + padding.top}
            fill={colour21}
          />
        {/if} -->
      {/each}
  
      <!-- CITY AVERAGE VALUES-->
      <!-- <g>
        <line class = "city-average"
          x1={xScale(cityAverage[1][varName])}
          y1={padding.top + 5}
          x2={xScale(cityAverage[1][varName])}
          y2={height - padding.bottom - 1.5 * barWidth}
          stroke={"grey"}
          stroke-width="1"
        />
        <text class = "city-average-text"
          x={xScale(cityAverage[1][varName])+10}
          y={padding.top + 50}
        >2021 City Average: {cityAverage[1][varName]}</text>
      </g> -->
  
      <!-- X AND Y AXIS-->
      <!-- x axis -->
      <line
        x1={padding.left}
        y1={padding.top + 4}
        x2={padding.left + innerWidth}
        y2={padding.top + 4}
        stroke-width={1}
        stroke={"#fff"}
      />
      {#if innerWidth > 300}
        {#each xTicks as tick, i}
          <text class="text" x={xScale(tick)} y={padding.top} text-anchor="middle"
            >{thousandToK(tick)}
          </text>
          <line
            x1={xScale(tick)}
            y1={padding.top + 4}
            x2={xScale(tick)}
            y2={padding.top + 12}
            stroke-width={1}
            stroke={"#fff"}
          />
          <line
            x1={xScale(tick)}
            y1={padding.top + 30}
            x2={xScale(tick)}
            y2={height - padding.bottom - 2 * barWidth}
            stroke-width={1}
            stroke={"#4d4d4d"}
          />
        {/each}
      {/if}
      <!-- y axis -->
      <line
        class="axis y-axis"
        x1={padding.left - 12}
        y1={padding.top + 30}
        x2={padding.left - 12}
        y2={height - padding.bottom - 2 * barWidth}
        stroke-width={6}
        stroke={lineColour[transitName]}
        z-index="3"
      />
  
      <!-- CREATE STATION TICKS AND LABELS-->
  
      
        <!-- CREATE THE BARS ON THE LINE GRAPH-->
        <!-- <rect
          class="barLight"
                  fill={"white"}
          x={padding.left}
          y={yScale(i) + barWidth / 2 - 2 * barPadding}
          width={xScale(Math.max(point[tue])) - padding.left}
          height={barWidth}
          opacity="0.1"
        />
        <rect
          class="barLight"
                  
          x={padding.left}
          y={yScale(i) + barWidth / 2 - 2 * barPadding}
          width={xScale(Math.max(point[tue])) - padding.left}
          height={barWidth}
          opacity="0"
          on:mouseover={(event) => {
            selected_datapoint = point;
            setMousePosition(event);
            document.querySelector(`#var-text-${i}`).style.opacity = 1;
            document.querySelector(`#var-hover-${i}`).style.opacity = 1;
            selected_datapoint_i = i;
          }}
          on:mouseout={() => {
            // Reset the opacity of var-text when the mouse leaves the bar
            document.querySelector(`#var-text-${i}`).style.opacity = 0;
            document.querySelector(`#var-hover-${i}`).style.opacity = 0;
          }}
        /> -->
        <!-- LABELLING THE DATA WILL SHOW ON HOVERING THE BARS-->
        <!-- <rect
          id={`var-hover-${i}`}
          class="var-hover"
          x={padding.left + innerWidth}
          y={yScale(i) + barWidth / 2 - 2 * barPadding}
          width={500}
          height={barPadding + 3 * barWidth}
          opacity="0"
        /> -->
        
  
      <!-- Line that marks the location of the station-->
      <g class="station-lines">
        {#each data as point, i}
            <line
              class="station-sep-lines"
              x1={padding.left + innerWidth}
              y1={yScale(i) + barPadding + padding.top}
              x2={padding.left}
              y2={yScale(i) + barPadding + padding.top}
              stroke-width={1}
              stroke="#fff"
              stroke-dasharray="5 3"
              opacity="0.1"
            />
            <circle
              class="point"
              r={6}
              cx={padding.left - 12}
              cy={yScale(i) + barPadding + padding.top}
              stroke-width="4px"
              stroke={lineColour[transitName]}
              fill="#FFFFFF"
            />
  
        {/each}
      </g>
      <g>
        {#each data as point, i}
            {#if i > 0 && point[avg] !== null && data[i - 1][avg] !== null}
                <line
                    x1={xScale(data[i - 1][avg])}
                    y1={yScale(i - 1) + padding.top}
                    x2={xScale(point[avg])}
                    y2={yScale(i) + padding.top}
                    stroke={lineColour[transitName]}
                    stroke-width="3"
                />
            {/if}

            {#if point[avg] !== null}
                <circle
                    class="point"
                    r={innerWidth / 200}
                    cx={xScale(point[avg])}
                    cy={yScale(i) + padding.top}
                    fill={lineColour[transitName]}
                    on:mouseover={(event) => {
                        selected_datapoint = point;
                        selected_datapoint_i = i;
                        setMousePosition(event);
                    }}
                    on:mouseout={() => {
                        selected_datapoint = undefined;
                    }}
                />
                <rect
                    class="barLight"
                    x={xScale(0)}
                    y={yScale(i) + padding.top/2}
                    height={barWidth - 2}
                    width={width - padding.left}
                    stroke={"white"}
                    opacity="0.1"
                    on:mouseover={(event) => {
                        selected_datapoint = point;
                        //setMousePosition(event);
                        selected_datapoint_i = i;
                        document.querySelector(`#var-text-${i}`).style.opacity = 1;
                    }}
                    on:mouseout={(event) => {
                      selected_datapoint = point;
                      //setMousePosition(event);
                      selected_datapoint_i = i;
                      document.querySelector(`#var-text-${i}`).style.opacity = 0;
                  }}
                />
            {/if}
        {/each}
        {#each data as point, i}
        <text
          id={`var-text-${i}`}
          class="var-text"
          x={padding.left + innerWidth}
          y={yScale(i) + barPadding + padding.top + 5}
          >
          <tspan
            x={[xScale(Math.max(point[avg])) + 20]}
            dy={-1}
            fill={"white"}
            >{Math.round(point[avg])}</tspan
          >

        </text>
      {/each}
    </g>
    </svg>
  </div>
  
  <style>
    /* CHART */
    .chart {
      /* width: 80%; */
      /* left: 0%; */
      max-width: 1000px;
      min-width: 500px;
    }
  
    svg {
      position: relative;
      /* width: 90%; */
      /* left: 5%; */
      z-index: 2;
    }
  
    .city-average {
      stroke-width: 5px;
      z-index: 1;
      stroke: white;
      opacity: 0.5;
    }
    .city-average-text {
      font-size: 15px;
      font-weight: bold;
      fill: #ffffff;
    }
  
    /* XY axis */
    .text {
      fill: white;
      font-size: 16px;
      background-color: lightgrey;
      font-weight: bold;
    }
  
    /* AXIS */
    .station-lines {
      stroke-width: 5px;
    }
  
    .station-tick {
      text-anchor: right;
      word-wrap: break-word;
      fill: var(--brandGray);
      font-size: 17px;
      text-align: right;
      max-width: 15px; /* Adjust the value as needed */
      /* Optional: Enable text wrapping */
      white-space: pre-line; /* Allow line breaks */
    }
    .station-box {
      width: 200px;
      height: 300px;
      background-color: red;
    }
  
    /* LINES GRAPH */
    .point {
      cursor: pointer;
    }
    .point:hover {
      fill: var(--brandLightBlue);
      opacity: 1;
    }
  
    /* BAR GRAPH */
    .barLight {
      opacity: 0;
      z-index: -5;
    }
  
    .barLight:hover {
      opacity: 1;
      fill-opacity: 0;
      cursor: pointer;
      z-index: 6;
    }
  
    /* HOVERING */
    #hoverLabel {
      height: 30px;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  
    .var-text {
      opacity: 0;
      padding-top: 5px;
      padding-left: 14px;
      font-size: 12px;
      font-weight: bold;
    }
    .var-hover {
      fill: black;
    }
    path {
      fill: transparent;
      stroke-width: 2;
      stroke-linejoin: round;
      z-index: 40;
    }
  </style>