<script>
    import * as d3 from 'd3';

    export let calfire_clean;
    
    const width = 928;
    const height = 600;
    const marginTop = 20;
    const marginRight = 30;
    const marginBottom = 30;
    const marginLeft = 40;

    let gx;
    let gy;
    let svg;

    /* 
    $: x=d3
    .scaleUtc()
    .domain(d3.extent(calfire_clean, (d) => d.Longitude))
    .range([marginLeft, width - marginRight])

    $: y = d3
    .scaleLinear()
    .domain(d3.extent(calfire_clean, (d) => d.Latitude))
    .range([height - marginBottom, marginTop])
    */


</script>


<div class="loc-plot">
    <svg
        bind:this={svg}
        width={width}
        height={height}
        viewBox="0 0 {width} {height}"
        style="max-width: 100%; height: auto;">

        <g stroke="#000" stroke-opcaity="0.2">
            {#each Object.values(calfire_clean) as data}
                <circle
                    cx={data.Longitude}
                    cy={data.Latitude}
                    r=5
                />
            {/each}
        </g>
        
    </svg>
</div>