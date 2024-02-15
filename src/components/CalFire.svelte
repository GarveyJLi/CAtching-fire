<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import { createEventDispatcher } from 'svelte';
    export let tempData;

    const width = 900;
    const height = 500;
    const marginTop = 10;
    const marginRight = 10;
    const marginBottom = 10;
    const marginLeft = 10;

    let gx;
    let gy;
    let svg;
    let marker_container;
    let circle_markers;

    const dispatch = createEventDispatcher();

    $: x=d3
    .scaleLinear() 
    .domain(d3.extent(tempData, (d) => d.Longitude)) 
    .range([marginLeft * 20, width - marginRight * 20])

    $: y=d3
    .scaleLinear()
    .domain(d3.extent(tempData, (d) => d.Latitude))
    .range([height - marginBottom, marginTop])

    $: d3.select(gx).call(d3.axisBottom(x));

    $: d3.select(gy).call(d3.axisLeft(y));

    
</script>


<div class="fire-plot">
    <svg
        bind:this={svg}
        style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);"
        width={width + marginLeft + marginRight}
        height={height + marginTop + marginBottom}
    >


        <!--x axis-->
        <g bind:this={gx} transform="translate(0, {height - marginBottom})"/>

        <!--y axis-->
        <g bind:this={gy} transform="translate({marginLeft + 40}, 0)"/>

        <g stroke="#000" stroke-opacity="0.2">
            {#each tempData as d, i}
              <circle key={i} cx={x(d.Longitude)} cy={y(d.Latitude)} fill='red' r="2.5"/>
            {/each}
        </g>

        
        <g transform={`translate(${marginLeft} ,${marginTop})`}>
            <g class="grid x-grid" id="xGrid" transform={`translate(0,${height})`}></g>
            <g class="grid y-grid" id="yGrid"></g>
            <g class="x-axis" id="xAxis"></g>
            <g class="y-axis" id="yAxis"></g>
            <g id="marker_container"></g>
        </g>
        

        

    </svg>
</div>
