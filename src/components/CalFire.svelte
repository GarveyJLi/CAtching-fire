<script>
    import * as d3 from 'd3';
    import { onMount, afterUpdate } from 'svelte';

    export let tempData;

    const width = 650;
    const height = 550;
    const marginTop = 20;
    const marginRight = 80;
    const marginBottom = 40; // Increased to accommodate axis labels
    const marginLeft = 40; // Increased to accommodate axis labels
    const extendFactor_x = 0.4; // Factor by which to extend the axes beyond the data range
    const extendFactor_y = 0.1;
    let gx;
    let gy;
    let svg;
    let marker_container;
    let circle_markers;
    let x;
    let y;

    let tooltipText = '';
    let tooltipInfo;
    let acresBurned = '';
    let fatalities = '';
    let long = '';
    let lat = '';

    // Define a linear scale for the radius based on AcresBurned
    let radiusScale = d3.scaleLinear()
        .domain(d3.extent(tempData, d => d.AcresBurned))
        .range([3, 11]); // Adjust the range as needed for the desired circle sizes
    // Define a color scale based on AcresBurned
    let colorScale = d3.scaleSequential(d3.interpolateHslLong)
        .domain(d3.extent(tempData, d => d.AcresBurned))
        .range(["#edca00", "#de1102"]);

    // Set the zoom and Pan features: how much you can zoom, on which part, and what to do when there is a zoom
    const zoom = d3.zoom()
        .scaleExtent([.05, 20])  // This control how much you can unzoom (x0.5) and zoom (x20)
        .extent([[50, 100], [width, height]])
        .on("zoom", updateChart);

    // This add an invisible rect on top of the chart area. This rect can recover pointer events: necessary to understand when the user zoom
    function updateChart() {
        if (!d3.event || !d3.event.transform) return; // Check if d3.event or d3.event.transform is undefined

        // recover the new scale
        const newX = d3.event.transform.rescaleX(x);
        const newY = d3.event.transform.rescaleY(y);

        // update axes with these new boundaries
        gx.call(d3.axisBottom(newX));
        gy.call(d3.axisLeft(newY));

        // update circle position
        circle_markers
            .attr('cx', (d) => newX(d.Longitude))
            .attr('cy', (d) => newY(d.Latitude));
    }

    onMount(() => {
        // Append the SVG object to the body of the page
        svg = d3.select("#dataviz_axisZoom")
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${marginLeft}, ${marginTop})`);

        // Add a clipPath: everything out of this area won't be drawn.
        const clip = svg.append("defs").append("clipPath")
            .attr("id", "clip")
            .append("rect")
            .attr("width", width - marginLeft - marginRight)
            .attr("height", height - marginTop - marginBottom)
            .attr("x", marginLeft)
            .attr("y", marginTop);

        // Create the scatter variable: where both the circles and the brush take place
        const scatter = svg.append('g')
            .attr("clip-path", "url(#clip)");

        // Append the x axis
        gx = svg.append("g")
            .attr("transform", `translate(0, ${height - marginBottom})`);

        // Append the y axis
        gy = svg.append("g")
            .attr("transform", `translate(${marginLeft}, 0)`);

        // This add an invisible rect on top of the chart area. This rect can recover pointer events: necessary to understand when the user zoom
        svg.append("rect")
            .attr("width", width)
            .attr("height", height)
            .style("fill", "none")
            .style("pointer-events", "all")
            .call(zoom);
            
        // Append the marker container (tooltip rectangle and text)
        marker_container = svg.append('g')
            .attr('class', 'marker-container')
            .attr('transform', `translate(${width - marginRight - 100}, ${marginTop})`)
            .style('display', 'none'); // Hide initially

 
        marker_container.append('text')
            .attr('x', 50)
            .attr('y', 15)
            .attr('text-anchor', 'middle')
            .text('');
            
        
    });

    afterUpdate(() => {
        // Calculate extended domain for x-axis
        let xExtent = d3.extent(tempData, (d) => d.Longitude);
        let xRange = [marginLeft, width - marginRight];
        x = d3.scaleLinear()
            .domain([xExtent[0] - (xExtent[1] - xExtent[0]) * extendFactor_x,
                    xExtent[1] + (xExtent[1] - xExtent[0]) * extendFactor_x])
            .range(xRange);

        // Calculate extended domain for y-axis
        let yExtent = d3.extent(tempData, (d) => d.Latitude);
        let yRange = [height - marginBottom, marginTop];
        y = d3.scaleLinear()
            .domain([yExtent[0] - (yExtent[1] - yExtent[0]) * extendFactor_y,
                    yExtent[1] + (yExtent[1] - yExtent[0]) * extendFactor_y - 1])
            .range(yRange);

        // Update the x and y axes
        gx.call(d3.axisBottom(x));
        gy.call(d3.axisLeft(y));

        // Update circles
        circle_markers = svg.selectAll("circle")
            .data(tempData)
            .attr("cx", (d) => x(d.Longitude))
            .attr("cy", (d) => y(d.Latitude))
            .attr("r", (d) => radiusScale(d.AcresBurned)); // Use the radius scale

        // Enter new circles
        circle_markers.enter()
            .append("circle")
            .attr("cx", (d) => x(d.Longitude))
            .attr("cy", (d) => y(d.Latitude))
            .attr("r", (d) => radiusScale(d.AcresBurned)) // Use the radius scale
            .attr("fill", (d) => colorScale(d.AcresBurned))
            .attr("opacity", 0.5)
            .on("mouseover", (event, d) => {
                tooltipText = `Longitude: ${d.Longitude} Latitude: ${d.Latitude}`;
                tooltipInfo = d;
                acresBurned = d.AcresBurned;
                fatalities = d.Fatalities;
                long = d.Longitude;
                lat = d.Latitude;
                marker_container.select('text').text(tooltipText);
            })
            .on("mouseout", () => {
                tooltipText = '';
                tooltipInfo = '';
                acresBurned = '';
                fatalities = '';
                long = '';
                lat = '';
                marker_container.select('text').text(tooltipText);
            });
            
        // Remove old circles
        circle_markers.exit().remove();
    });
</script>

<div id = "wrapper">
    <div class="fire-plot" id="dataviz_axisZoom" style="overflow: auto; height: ${1000}px;"></div>


    <div id="div2">
        <p align = 'left'>
        Longitude: {long} <br />
        Latitude: {lat} <br />
        Acres Burned: {acresBurned} <br />
        Fatalities: {fatalities} <br />
        </p>
    </div>


</div>

<style>
    #wrapper {
        border: 1px solid blue;
    }
    #dataviz_axisZoom {
        display: inline-block;
        width:650;
        height:500;
        border: 1px solid red;
    }
    #div2 {
        vertical-align:top;
        display: inline-block;
        width:260px;
        height:200px;
        border: 1px solid green;
        font-size:0.5em;
    }

    p {
    margin: 35px;
    }

</style>
