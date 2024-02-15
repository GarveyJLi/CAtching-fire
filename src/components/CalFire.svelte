<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import { createEventDispatcher } from 'svelte';
    export let tempData;

    const width = 1200;
    const height = 700;
    const marginTop = 20;
    const marginRight = 10;
    const marginBottom = 10;
    const marginLeft = 295;
    const extendFactor_x = 0.4; // Factor by which to extend the axes beyond the data range
    const extendFactor_y = 0.1;
    let gx;
    let gy;
    let svg;
    let marker_container;
    let circle_markers;

    const dispatch = createEventDispatcher();

    // Calculate extended domain for x-axis
    let xExtent = d3.extent(tempData, (d) => d.Longitude);
    let xRange = [marginLeft, width - marginRight];
    let x = d3.scaleLinear()
              .domain([xExtent[0] - (xExtent[1] - xExtent[0]) * extendFactor_x,
                       xExtent[1] + (xExtent[1] - xExtent[0]) * extendFactor_x])
              .range(xRange);

    // Calculate extended domain for y-axis
    let yExtent = d3.extent(tempData, (d) => d.Latitude);
    let yRange = [height - marginBottom, marginTop];
    let y = d3.scaleLinear()
              .domain([yExtent[0] - (yExtent[1] - yExtent[0]) * extendFactor_y,
                       yExtent[1] + (yExtent[1] - yExtent[0]) * extendFactor_y - 1])
              .range(yRange);

    // Update the x and y axes
    $: d3.select(gx).call(d3.axisBottom(x));
    $: d3.select(gy).call(d3.axisLeft(y));



    function handleZoom(e) {
    d3.select('g.chart')
        .attr('transform', e.transform);
    }

    let zoom = d3.zoom()
        .on('zoom', handleZoom);

    function initZoom() {
        d3.select('svg')
            .call(zoom);
    }




</script>

<div class="fire-plot">
    <svg
        bind:this={svg}
        style="position: absolute; top: 70%; left: 50%; transform: translate(-50%, -50%);"
        width={width + marginLeft + marginRight}
        height={height + marginTop + marginBottom}
    >
        <!--x axis-->
        <g bind:this={gx} transform={`translate(0, ${height - marginBottom})`} />

        <!--y axis-->
        <g bind:this={gy} transform={`translate(${marginLeft}, 0)`} />

        <g stroke="#000" stroke-opacity="0.2">
            {#each tempData as d, i}
                <!-- svelte-ignore a11y-mouse-events-have-key-events -->
                <!-- svelte-ignore a11y-no-static-element-interactions -->
                <circle
                    key={i}
                    cx={x(d.Longitude)}
                    cy={y(d.Latitude)}
                    fill='red'
                    r="2.5"
                    on:mouseover={() => console.log('Hover')}
                    on:mouseout={() => console.log('CYA')}
                />
            {/each}
        </g>

    </svg>
</div>


