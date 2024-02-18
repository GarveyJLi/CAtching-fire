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


    function handleMouseOver(event,d){
        acresBurned = d.AcresBurned;
        fatalities = d.Fatalities;
        long = d.Longitude;
        lat = d.Latitude;
        year = d.ArchiveYear;
        county = d.Counties;
        name = d.Name;
        description = d.SearchDescription;
    }
    
    const acreBurned_quantiles = [0, 35, 98, 422.5, 410203]
    const color_quantiles = ["#FFD94F", "#FAAE3B" , "#F58228" , "#F05714" , "#EB2B00" ]
    let acresBurned = '';
    let fatalities = '';
    let year = '';
    let description = '';
    let name = '';
    let long = '';
    let lat = '';
    let county = '';

    let zoom = d3.zoom()
        .on('zoom', handleZoom);

    // Define a linear scale for the radius based on AcresBurned
    let radiusScale = d3.scaleLinear()
        .domain([0, 410203])
        .range([2.5, 15]); // Adjust the range as needed for the desired circle sizes
    // Define a color scale based on AcresBurned
    let colorScale = d3.scaleLinear(d3.interpolateHslLong)
        .domain([0, 410203])
        .range(["#ffd6d1", "#de1102"]);

    
    onMount(() => {
        // Append the SVG object to the body of the page
        svg = d3.select("#dataviz_axisZoom")
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${marginLeft}, ${marginTop})`);

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
            .style("fill", (d) => colorScale(d.AcresBurned))
            .attr("r", (d) => radiusScale(d.AcresBurned));
            
        // Enter new circles
        circle_markers.enter()
            .append("circle")
            .attr("cx", (d) => x(d.Longitude))
            .attr("cy", (d) => y(d.Latitude))
            .attr("r", (d) => radiusScale(d.AcresBurned))
            .style("fill", (d) => colorScale(d.AcresBurned))
            .attr("opacity", 0.75)
            .on("mouseover", function(event, d) {
                // Update tooltip content
                acresBurned = d.AcresBurned;
                fatalities = d.Fatalities;
                long = d.Longitude;
                lat = d.Latitude;
                year = d.ArchiveYear;
                county = d.Counties;
                name = d.Name;
                description = d.SearchDescription;
                // Highlight the element
                d3.select(this).style("fill", "blue");
            })

            .on("mouseout", () => {
                acresBurned = '';
                fatalities = '';
                long = '';
                lat = '';
                year= '';
                county = '';
                name = '';
                description= '';

            });
        // Remove old circles
        circle_markers.exit().remove();
    });
</script>

<div id = "wrapper">
    <div class="fire-plot" id="dataviz_axisZoom" ></div>
    <div id="div2">
        <div id = "div-b">
            <div id = "items" align = 'left'>
                <span id = 'item-b'> Legend: </span> <br />
                <p></p>
                <div id = "legend">
                    <span>Acres Burned</span>
                    <svg style="display: block;"> 
                        <defs> 
                            <linearGradient id="GFGGradient"> 
                                <stop offset="0%" stop-color="#ffd6d1" /> 
                                <stop offset="100%" stop-color="#de1102" /> 
                            </linearGradient> 
                        </defs> 
                        <g> 
                            <rect width="100%" height="20" fill="url(#GFGGradient)" /> 
                        </g> 
                    </svg> 
                    <span id = "legend" >
                        <span id = "min">0</span>
                        <span id = "max" style = "float:right">410203</span>
                    </span>

                    <svg style="display: block; height: 40px;">
                        <g>
                            /* 2.5, 15*/
                            <circle cx="2.5%" cy="15" r = "2.5" fill = "#757575" />
                            <circle cx="30%" cy="15" r = "5" fill = "#757575" />
                            <circle cx="60%" cy="15" r = "10" fill = "#757575" />
                            <circle cx="92.5%" cy="15" r = "15" fill = "#757575" />
                        </g>
                        
                    </svg>
                </div>
            </div>
        </div>
        <div id = "div-a">
            <div id = "items" align = 'left'>
                <span id = 'item'> Name:   </span> {name} <br />
                <p></p>
                <span id = 'item'> Coordinates:   </span> {lat}, {long} <br />
                <p></p>
                <span id = 'item'> County:     </span> {county} <br />
                <p></p>
                <span id = 'item'> Acres Burned:  </span> {acresBurned} <br />
                <p></p>
                <span id = 'item'> Fatalities:  </span> {fatalities} <br />
                <p></p>
                <span id = 'item'> Year:     </span> {year} <br />
                <p></p>
                <span id = 'item'> Description:     </span> {description} 

            </div>
        </div>
        <br />
        <p></p>

    </div>
</div>

<style>

    svg{
        padding:-10000px;
        margin-bottom:-130px;
        width: 100%;
        height: 20px;
        display: block;
        margin: auto;
    }
    
    #legend{
        margin-bottom:10px;
    }
    #wrapper {
        border: 0px solid blue;
    }
    #dataviz_axisZoom {
        display: inline-block;
        width: 55%;
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
        margin: 15px;
    }
    #item {
        background: #d95e00;
        border-radius: 5px;
        padding:5px;
        color:white;
        font-weight: bold;
    }

    #div-a {
        vertical-align:top;
        display: inline-block;
        background: #ffe1a3;
        border-radius: 8px;
        min-height: 380px;
        height: auto;
        width: 100%;
    }
    #div-b {
        vertical-align:top;
        display: inline-block;
        background: #f0f0f0;
        border-radius: 8px;
        width: 100%;
        height: auto;
        margin-bottom: 10px;
    }
    #item-b {
        background: #ffc445;
        border-radius: 5px;
        padding:5px;
        color:white;
        font-weight: bold;

    }
    



</style>
