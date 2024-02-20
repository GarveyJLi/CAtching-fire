<script>
    import * as d3 from 'd3';
    import { onMount, afterUpdate } from 'svelte';

    export let tempData;

    const width = 650;
    const height = 550;
    const k = height / width;
    const marginTop = 0;
    const marginRight = 0;
    const marginBottom = 0; // Increased to accommodate axis labels
    const marginLeft = 0; // Increased to accommodate axis labels

    // Create extended domain for x-axis
    const xRange = [marginLeft, width - marginRight];
    const x = d3.scaleLinear()
            // Manually set domain past range of x values in dataset
            .domain([-127.5, -113.5])
            .range(xRange);

    // Create extended domain for y-axis
    const yRange = [height - marginBottom, marginTop];
    const y = d3.scaleLinear()
        // Manually set domain past range of y values in dataset
        .domain([31.5, 42.5])
        .range(yRange);

    const xAxis = (g, x) => g
            .attr("transform", `translate(0,${height - marginBottom})`)
            .call(d3.axisTop(x)
            .ticks(9)
            .tickSize(height)
            // Tick label location
            .tickPadding((27 - height)))
            .call(g => g.select(".domain"))
            // Axis label
            .append("text")
            .attr("x", width /2)
            .attr("y", marginBottom - 10)
            .attr("fill", "#000")
            .style("font", "14px times")
            .attr("text-anchor", "middle")
            .text("Longitude");
            ;

    const yAxis = (g, y) => g
        .attr("transform", `translate(${marginLeft}, 0)`)
        .call(d3.axisRight(y)
        .ticks(10)
        .tickSize(width)
        // Tick label location
        .tickPadding((40 - width)))
        .call(g => g.select(".domain"))
        // Axis label
        .append("text")
        .attr("x", -width/2)
        .attr("y", 20)
        .style("font", "14px times")
        .attr("transform", "rotate(-90)")
        .attr("fill", "#000")
        .attr("text-anchor", "middle")
        .text("Latitude");;
    

    // Define a linear scale for the radius based on AcresBurned
    const radiusScale = d3.scaleLinear()
        .domain([0, 410203])
        .range([2.5, 15]); // Adjust the range as needed for the desired circle sizes
    // Define a color scale based on AcresBurned
    const colorScale = d3.scaleLinear(d3.interpolateHslLong)
        .domain([0, 410203])
        .range(["#ffd6d1", "#de1102"]);
    

    let svg;
    let gGrid;
    let grid;
    let gx;
    let gy;
    let circle_markers;
    var zoom_factor = 1;

    
    var acresBurned = '';
    var fatalities = '';
    var year = '';
    var description = '';
    var name = '';
    var long = '';
    var lat = '';
    var county = '';

    

    
    onMount(async () => {

        const res = await fetch('calfire.csv'); 
        const csv = await res.text();
        tempData = d3.csvParse(csv, d3.autoType)
        
        let zoom = d3.zoom()
            .scaleExtent([1, 10])
            .translateExtent([[0, 0], [width, height]])
            .filter(filter)
            .on('zoom', handleZoom);
            
    
        // Append the SVG object to the body of the page
        svg = d3.select("#dataviz_axisZoom")
            .append("svg")
            .attr('width', width)
            .attr('height', height)
            .attr("viewBox", [0, 0, width, height]);

        gGrid = svg.append("g");

        // Append the x axis
        gx = svg.append("g");

        // Append the y axis
        gy = svg.append("g");

        circle_markers = svg.selectAll("circle").data(tempData);
        svg.call(zoom).call(zoom.transform, d3.zoomIdentity);

        grid = (g, x, y) => g
            .attr("stroke", "currentColor")
            .attr("stroke-opacity", 0.1)
            .call(g => g
            .selectAll(".x")
            .data(x.ticks(10))
            .join(
                enter => enter.append("line").attr("class", "x").attr("y2", height),
                update => update,
                exit => exit.remove()
            )
                .attr("x1", d => 0.5 + x(d))
                .attr("x2", d => 0.5 + x(d)))
            .call(g => g
            .selectAll(".y")
            .data(y.ticks(10 * k))
            .join(
                enter => enter.append("line").attr("class", "y").attr("x2", width),
                update => update,
                exit => exit.remove()
            )
                .attr("y1", d => 0.5 + y(d))
                .attr("y2", d => 0.5 + y(d)));
        


        function handleZoom({transform}) {
            zoom_factor = transform.k   
            const zx = transform.rescaleX(x).interpolate(d3.interpolateRound);
            const zy = transform.rescaleY(y).interpolate(d3.interpolateRound);
            

            // Translate circle_markers according to transform (zoom or pan)
            circle_markers.attr("transform", transform).attr("r", (d) => radiusScale(d.AcresBurned) / zoom_factor);
            // Update axes 
            gx.call(xAxis, zx);
            gy.call(yAxis, zy);
            gGrid.call(grid, zx, zy)
            }

        function filter(event) {
            event.preventDefault();
            return (!event.ctrlKey || event.type === 'wheel') && !event.button;
        }
    });
    
    afterUpdate(() => {
        // Update circles
        circle_markers = svg.selectAll("circle")
            .data(tempData)

        // Enter new circles
        circle_markers.enter()
            .append("circle")
            .attr("cx", (d) => x(d.Longitude))
            .attr("cy", (d) => y(d.Latitude))
            .attr("r", (d) => radiusScale(d.AcresBurned) / zoom_factor)
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
            .on("mouseout", function(event, d) {
                acresBurned = '';
                fatalities = '';
                long = '';
                lat = '';
                year= '';
                county = '';
                name = '';
                description= '';
                d3.select(this).style("fill", colorScale(d.AcresBurned));
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
        border: px solid blue;
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
