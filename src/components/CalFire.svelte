<script>
    import * as d3 from 'd3';
    import { onMount, afterUpdate } from 'svelte';

    export let tempData;

    const width = 650;
    const height = 550;
    let k = height / width;
    const marginTop = 20;
    const marginRight = 80;
    const marginBottom = 40; // Increased to accommodate axis labels
    const marginLeft = 40; // Increased to accommodate axis labels

    

    let svg;
    let gGrid;
    let grid;
    let gx;
    let gy;
    let circle_markers;
    let x;
    let y;
    let xAxis;
    let yAxis;
    let zoom_factor = 1;

    
    let acresBurned = '';
    let fatalities = '';
    let year = '';
    let description = '';
    let name = '';
    let long = '';
    let lat = '';
    let county = '';

    // Define a linear scale for the radius based on AcresBurned
    let radiusScale = d3.scaleLinear()
        .domain([0, 410203])
        .range([2.5, 15]); // Adjust the range as needed for the desired circle sizes
    // Define a color scale based on AcresBurned
    let colorScale = d3.scaleLinear(d3.interpolateHslLong)
        .domain([0, 410203])
        .range(["#ffd6d1", "#de1102"]);

    
    onMount(() => {
        let zoom = d3.zoom()
            .scaleExtent([1, 10])
            .translateExtent([[-10, -10], [width + 50, height + 50]])
            .filter(filter)
            .on('zoom', handleZoom);
    
        // Append the SVG object to the body of the page
        svg = d3.select("#dataviz_axisZoom")
            .append("svg")
            .attr("viewBox", [0, 0, width, height])

            //.style("pointer-events", "all")
            //.call(zoom)
            //.append("g")
            //.attr("transform", `translate(${marginLeft}, ${marginTop})`);


        gGrid = svg.append("g");

        // Append the x axis
        gx = svg.append("g");
            //.attr("transform", `translate(0, ${height - marginBottom})`);

        // Append the y axis
        gy = svg.append("g");
            //.attr("transform", `translate(${marginLeft}, 0)`);

        svg.call(zoom).call(zoom.transform, d3.zoomIdentity);

        grid = (g, x, y) => g
            .attr("stroke", "currentColor")
            .attr("stroke-opacity", 0.1)
            .call(g => g
            .selectAll(".x")
            .data(x.ticks(12))
            .join(
                enter => enter.append("line").attr("class", "x").attr("y2", height),
                update => update,
                exit => exit.remove()
            )
                .attr("x1", d => 0.5 + x(d))
                .attr("x2", d => 0.5 + x(d)))
            .call(g => g
            .selectAll(".y")
            .data(y.ticks(12 * k))
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
            
            circle_markers.attr("transform", transform).attr("r", (d) => radiusScale(d.AcresBurned) / zoom_factor);
            gx.call(xAxis, zx);
            gy.call(yAxis, zy);
            gGrid.call(grid, zx, zy)

            console.log(event.type)
            // Transforms chart
            //svg.attr('transform', transform);
            // Dynamically changes x and y axes
            updateAxes();
            if (event.type === 'wheel') {
                //updateCircles();
                }
            }

        function filter(event) {
            event.preventDefault();
            return (!event.ctrlKey || event.type === 'wheel' || !event.type === 'mouseover') && !event.button;
        }
    });

    function updateCircles() {
        // Update circles
        circle_markers = svg.selectAll("circle")
            .data(tempData)
            .attr("r", (d) => radiusScale(d.AcresBurned) / zoom_factor); 


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
    }

    function updateAxes() {
        // Create extended domain for x-axis
        let xRange = [marginLeft, width - marginRight];
        x = d3.scaleLinear()
            // Manually set domain past range of x values in dataset
            .domain([-128, -112])
            .range(xRange);

        xAxis = (g, x) => g
            .attr("transform", `translate(0,${height - marginBottom})`)
            .call(d3.axisTop(x)
            .ticks(10)
            .tickSize(height)
            .tickPadding((8 - height)))
            .call(g => g.select(".domain").attr("display", "none"));
             

        // Create extended domain for y-axis
        let yRange = [height - marginBottom, marginTop];
        y = d3.scaleLinear()
            // Manually set domain past range of y values in dataset
            .domain([32, 42])
            .range(yRange);


            
        yAxis = (g, y) => g
        .attr("transform", `translate(${marginLeft}, 0)`)
        .call(d3.axisRight(y)
            .ticks(10)
            .tickSize(width)
            .tickPadding((8 - width)))
        .call(g => g.select(".domain").attr("display", "none"));
        
            
        // Update the x and y axes
        // To get rid of axis, get rid of call()
        //gx.call(xAxis)
        //gy.call(yAxis)
    }
    

    afterUpdate(() => {
        updateAxes();
        updateCircles();
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
