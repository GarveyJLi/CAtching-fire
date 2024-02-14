<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import { createEventDispatcher } from 'svelte';

    const width = 900;
    const height = 500;
    const marginTop = 10;
    const marginRight = 10;
    const marginBottom = 10;
    const marginLeft = 10;

    let gx;
    let gy;

    let fireData = [];

    let fire_data = [];
    let circle_markers;
    let trip_data = [];

    const dispatch = createEventDispatcher();

    let svg;
    let marker_container;

    

    function create_markers(fire_data) {
        marker_container = d3.select(svg)
            .append('g')
            .attr('id', 'marker_container');

        circle_markers = marker_container
            .selectAll("circle")
            .data(fire_data)
            .enter()
            .append("circle")
            .attr("r", 5)
            .style("fill", "#808080")
            .attr("stroke", "#808080")
            .attr("stroke-width", 1)
            .attr("fill-opacity", 0.4)
    }

    function position_markers(xScale, yScale) {
		circle_markers
			.attr("cx", function (d) {
				return project_x(xScale, d);
			})
			.attr("cy", function (d) {
				return project_y(yScale, d);
			});
	}

    function project_x(xScale, d) {
        return xScale(d.Longitude) + marginLeft;
    }

    function project_y(yScale, d) {
        return yScale(d.Latitude) + marginTop;
    }

    onMount(async () => {
        const res = await fetch('calfire.csv');
        const csv = await res.text();
        fireData = d3.csvParse(csv, d3.autoType);
        
        let xScale = d3.scaleLinear()
            .domain(d3.extent(fireData, d => d.Longitude))
            .range([marginLeft, width - marginRight]);
        let yScale = d3.scaleLinear()
                .domain(d3.extent(fireData, d => d.Latitude))
                .range([height - marginBottom, marginTop]);
        
        create_markers(fireData);
        position_markers(xScale, yScale);
        console.log(fireData)
    });

</script>



<svg
    bind:this={svg}
    style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);"
    width={width + marginLeft + marginRight}
    height={height + marginTop + marginBottom}
>

    <g transform={`translate(${marginLeft},${marginTop})`}>
        <g class="grid x-grid" id="xGrid" transform={`translate(0,${height})`}></g>
        <g class="grid y-grid" id="yGrid"></g>
        <g class="x-axis" id="xAxis"></g>
        <g class="y-axis" id="yAxis"></g>
        <g id="marker_container"></g>
    </g>

    

</svg>
