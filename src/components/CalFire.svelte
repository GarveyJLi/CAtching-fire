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
    let year = '';
    let long = '';
    let lat = '';
    let zoom = d3.zoom()
        .on('zoom', handleZoom);

    // Define a linear scale for the radius based on AcresBurned
    let radiusScale = d3.scaleLinear()
        .domain(d3.extent(tempData, d => d.AcresBurned))
        .range([3, 11]); // Adjust the range as needed for the desired circle sizes
    // Define a color scale based on AcresBurned
    let colorScale = d3.scaleSequential(d3.interpolateHslLong)
        .domain(d3.extent(tempData, d => d.AcresBurned))
        .range(["#edca00", "#de1102"]);

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
        //const scatter = svg.append('g')
        //    .attr("clip-path", "url(#clip)");

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

    });

    function handleZoom(e) {
         svg.attr('transform', e.transform);   
        }

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
            .attr("fill", (d) => colorScale(d.AcresBurned))
            .attr("r", 5); // Use the radius scale

        // Enter new circles
        circle_markers.enter()
            .append("circle")
            .attr("cx", (d) => x(d.Longitude))
            .attr("cy", (d) => y(d.Latitude))
            .attr("r", 5) // Use the radius scale
            .attr("fill", (d) => colorScale(d.AcresBurned))
            .attr("opacity", 0.5)
            .on("mouseover", (event, d) => {
                tooltipText = `Longitude: ${d.Longitude} Latitude: ${d.Latitude}`;
                tooltipInfo = d;
                acresBurned = d.AcresBurned;
                fatalities = d.Fatalities;
                long = d.Longitude;
                lat = d.Latitude;
                year = d.ArchiveYear;
            })
            .on("mouseout", () => {
                tooltipText = '';
                tooltipInfo = '';
                acresBurned = '';
                fatalities = '';
                long = '';
                lat = '';
                year= '';

            });
            console.log('here')
        // Remove old circles
        //circle_markers.exit().remove();
    });
</script>

<div id = "wrapper">
    <div class="fire-plot" id="dataviz_axisZoom" style="overflow: auto; height: ${1000}px;"></div>
    <div id="div2">
        <div id = "div-a">
            <div id = "items" align = 'left'>
                <span id = 'item'> Longitude:   </span> {long} <br />
                <p></p>
                <span id = 'item'> Latitude:    </span> {lat} <br />
                <p></p>
                <span id = 'item'> Acres Burned:  </span> {acresBurned} <br />
                <p></p>
                <span id = 'item'> Fatalities:  </span> {fatalities} <br />
                <p></p>
                <span id = 'item'> Year:     </span> {year} <br />
            </div>
        </div>
        <br />
        <p></p>
        <div id = "div-b">
            <div id = "items" align = 'left'>
                <span id = 'item-b'> About the Data: </span> <br />
                <p></p>
                <p>The data in this visualization is a collection of California wildfires spanning from 2013 to 2019. The dataset was sourced from CalFire – California Department of Forest and Fire Protection.</p>
            </div>
        </div>

    </div>
</div>

<style>

    #wrapper {
        border: 0px solid blue;
    }
    #dataviz_axisZoom {
        display: inline-block;
        width:55%;
        height:400;
        border: 1px solid red;
    }
    #div2 {
        vertical-align:top;
        display: inline-block;
        width: 25%;
        font-size:0.605em;
        text-align: left; /* Align content to the left */
    }
    #items {
        margin: 20px;
    }
    #item {
        background: #d95e00;
        border-radius: 5px;
        padding-top: 5px;
        padding-right: 5px;
        padding-bottom: 5px;
        padding-left: 5px;
        color:white;
        font-weight: bold;
    }

    #div-a {
        vertical-align:top;
        display: inline-block;
        background: #ffe1a3;
        border-radius: 12px;
        width: 100%;

    }
    #div-b {
        vertical-align:top;
        display: inline-block;
        background: #f0f0f0;
        border-radius: 12px;
        width: 100%;
    }
    #item-b {
        background: #ffc445;
        border-radius: 5px;
        padding-top: 5px;
        padding-right: 5px;
        padding-bottom: 5px;
        padding-left: 5px;
        color:white;
        font-weight: bold;
    }
    



</style>
