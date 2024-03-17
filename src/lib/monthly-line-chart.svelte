<script>
    import { scaleLinear } from "d3-scale";
    import { rollups, sum } from "d3-array";
    import jsondata from "../python/data-json/2023-ridership.json";
    import "../assets/global-styles.css";
    /*
  export let variable;
  export let yTicks;
  export let colour;
  export let maxHeight;
  export let type;
*/
    let maxHeight = 400;
    let countType = "exit";
    let width = 100;
    let height = 60;
    let colour = "white"

    export let stationName;
    console.log(stationName)
    //filtre jsondata.json to the selected transitName in page.svelte
    function filterDesignation(jsondata) {
        return jsondata["Station"] === stationName;
    }
    var data = jsondata.filter(filterDesignation);

    $: height = maxHeight;

    var ticklist = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
        
    ];

    var daylist = [];

    for (let i = 0; i < ticklist.length; i++) {
        daylist.push(data[0][`${ticklist[i]}-${countType}`]);
    }

    console.log(daylist);

    var yTicks = [
        0, 5000, 10000, 20000, 30000, 40000, 50000, 60000, 70000, 150000,
    ];

    const xTicks = ticklist;
    const padding = { top: 20, right: 0, bottom: 35, left: 45 };

    function formatMobile(tick) {
        return "'" + tick.toString().slice(-2);
    }

    // converts thousands and million to K and M i.e. (1,000 ==> 1K , 1,000,000 ==> 1M)
    function thousandToK(tick) {
        var newtick;
        if (tick >= 1000 && tick < 1000000) {
            newtick = tick / 1000 + "K";
        } else if (tick > 1000000) {
            newtick = tick / 1000000 + "M";
        } else {
            newtick = tick;
        }
        return newtick;
    }
    $: xScale = scaleLinear()
        .domain([0, xTicks.length])
        .range([padding.left, width - padding.right]);

    $: yScale = scaleLinear()
        .domain([0, Math.max.apply(null, yTicks)])
        .range([height - padding.bottom, padding.top]);

    $: innerWidth = width - (padding.left + padding.right);

    $: barWidth = Math.max(Math.min(innerWidth / xTicks.length, 9), 56);

    let selected_datapoint = undefined;
    let selected_datapoint_i = undefined;

    let mouse_x, mouse_y;
    const setMousePosition = function (event) {
        mouse_x = event.clientX;
        mouse_y = event.clientY;
    };

    var barPadding = 10; // controls how much spacing the bars will be from the
</script>

<div id="barchart" class="chart" bind:clientWidth={width}>
    <svg width={xTicks.length * barWidth} {height}>
        <!-- Second x axis, only appears when the inner window width > 800 -->
        <!--  x axis - monthly-->
        <g class="axis x-axis">
            {#each daylist as stn, i}
                <!-- if the inner window width <=800 show years only-->
                <g class="tick" transform="translate({xScale(i)},{height})">
                    <text x={barWidth / 2 + 13} y="-20">{ticklist[i]}</text>
                </g>
            {/each}
        </g>

        <g>
            {#each daylist as stn, i}
                <!-- Controls the width of the bar graph, 
                  width: controls the spacing between the bars-->
                <rect
                    class="bar"
                    x={xScale(i) + barPadding}
                    y={yScale(stn)}
                    width={barWidth - 2}
                    height={yScale(0) - yScale(stn)}
                    on:mouseover={(event) => {
                        selected_datapoint = stn;
                        selected_datapoint_i = i;
                        // setMousePosition(event);
                    }}
                    on:mouseout={() => {
                        selected_datapoint = undefined;
                    }}
                    color={colour}
                />

                <!-- <rect class="tip"
                      x={xScale(i)+barPadding}
                      y={yScale(stn) + 0}
                      width={barWidth - 2}
                      height={8}
                  /> -->
            {/each}
        </g>

        <!-- y axis -->
        <g class="axis y-axis">
            {#each yTicks as tick}
                <g
                    class="tick tick-{tick}"
                    transform="translate(0, {yScale(tick)})"
                >
                    <line x2="100%" />
                    <text y="-4">{thousandToK(tick)} </text>
                </g>
            {/each}
        </g>
    </svg>
</div>


<div id="hoverLabel">
    <p>
        {#if selected_datapoint != undefined}
            {"  "}:
            <span id="lightBlue"
                >{selected_datapoint.toLocaleString()}</span
            >
        {/if}
    </p>
</div>

<style>
    .chart {
        width: 100%;
        max-width: 100%;
        min-width: 300px;
        margin: 0 auto;
    }
    #hoverLabel {
        height: 30px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    #hoverLabel p {
        color: var(--brandWhite);
        font-family: RobotoRegular;
        font-size: 12px;
        text-align: center;
    }
    #lightBlue {
        color: var(--brandLightBlue);
    }

    svg {
        position: relative;
        width: 100%;
    }

    .tick {
        font-family: RobotoRegular;
        font-size: 0.725em;
        font-weight: 200;
    }

    .tick line {
        stroke: var(--brandGray);
        stroke-width: 1px;
        opacity: 0.1;
    }
    .tick text {
        fill: var(--brandGray);
        text-anchor: start;
        font-size: 12px;
    }

    .tick.tick-0 line {
        stroke-dasharray: 0;
    }

    .x-axis .tick text {
        text-anchor: middle;
        font-size: 12px;
        text-align: right;
    }

    .x-label {
        text-anchor: middle;
        transform: translate(-10px, 0px) rotate(-90deg);
    }
    .x-label.tick text {
        font-size: 20px; /* Adjust the font size as desired */
        fill: #000000; /* Adjust the font color as desired */
    }

    .bar {
        stroke: var(--brandDarkGreen);
        stroke-width: 1px;
        stroke-opacity: 1;
        fill: var(--brandWhite);
        cursor: pointer;
    }
    .bar:hover {
        fill: var(--brandLightBlue);
        opacity: 1;
    }
    .barLight {
        stroke: var(--brandDarkGreen);
        stroke-width: 1px;
        stroke-opacity: 1;
        fill: var(--brandWhite);
    }
    .barLight:hover {
        opacity: 0.4;
        cursor: pointer;
    }

    .point {
        cursor: pointer;
    }
    .point:hover {
        fill: var(--brandLightBlue);
        opacity: 1;
    }
    .tip {
        stroke: var(--brandDarkGreen);
        stroke-width: 1px;
        fill: var(--brandYellow);
    }
    .year-tick {
        stroke-width: 2px;
        z-index: 6;
    }
</style>
