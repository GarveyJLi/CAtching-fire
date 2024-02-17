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

    

    let svg;
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

    
    const acreBurned_quantiles = [0, 35, 98, 422.5, 410203]
    const color_quantiles = ["#FFD94F", "#FAAE3B" , "#F58228" , "#F05714" , "#EB2B00" ]
    let colorScale2 = d3.scaleThreshold(d3.interpolateHslLong)
        .domain(acreBurned_quantiles)
        .range(color_quantiles);
    

    // Define a linear scale for the radius based on AcresBurned
    let radiusScale = d3.scaleLinear()
        .domain([0, 410203])
        .range([4, 15]); // Adjust the range as needed for the desired circle sizes
    

    onMount(() => {
        let zoom = d3.zoom()
            .scaleExtent([1, 10])
            .translateExtent([[-10, -10], [width + 50, height + 50]])
            .filter(filter)
            .on('zoom', handleZoom);
    
        // Append the SVG object to the body of the page
        svg = d3.select("#dataviz_axisZoom")
            .append("svg")
            .style("pointer-events", "all")
            .call(zoom)
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${marginLeft}, ${marginTop})`)
            
            ;


        // Append the x axis
        gx = svg.append("g")
            .attr("transform", `translate(0, ${height - marginBottom})`);

        // Append the y axis
        gy = svg.append("g")
            .attr("transform", `translate(${marginLeft}, 0)`);

        

        function handleZoom({transform}) {
            console.log(event.type)
            // Transforms chart
            svg.attr('transform', transform);
            // Dynamically changes x and y axes
            updateAxes();
            zoom_factor = transform.k   
            if (event.type === 'wheel') {
                updateCircles();
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
            .style("fill", (d) => colorScale2(d.AcresBurned))
            .attr("opacity", 0.5)
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
                console.log("mouse over")
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
                d3.select(this).style("fill", colorScale2(d.AcresBurned));
            });
        // Remove old circles
        circle_markers.exit().remove();
    }

    function updateAxes() {
        // Calculate extended domain for x-axis
        let xExtent = d3.extent(tempData, (d) => d.Longitude);
        let xRange = [marginLeft, width - marginRight];
        x = d3.scaleLinear()
            .domain([xExtent[0] - (xExtent[1] - xExtent[0]) * extendFactor_x,
                    xExtent[1] + (xExtent[1] - xExtent[0]) * extendFactor_x])
            .range(xRange);

        xAxis = d3.axisTop(x)
            .ticks(10 * zoom_factor)
            .tickSize(height)
            .tickPadding((8 - height) / zoom_factor);

        // Calculate extended domain for y-axis
        let yExtent = d3.extent(tempData, (d) => d.Latitude);
        let yRange = [height - marginBottom, marginTop];
        y = d3.scaleLinear()
            .domain([yExtent[0] - (yExtent[1] - yExtent[0]) * extendFactor_y,
                    yExtent[1] + (yExtent[1] - yExtent[0]) * extendFactor_y - 1])
            .range(yRange);

        yAxis = d3.axisRight(y)
            .ticks(10 * zoom_factor)
            .tickSize(width)
            .tickPadding((8 - width) / zoom_factor)
            
        // Update the x and y axes
        gx.call(xAxis);
        gy.call(yAxis);
    }
    

    afterUpdate(() => {
        updateAxes();
        updateCircles();
    });
</script>

<div id = "wrapper">
    <div class="fire-plot" id="dataviz_axisZoom" style="overflow: auto; height: ${1000}px;"></div>
    <div id="div2">
        <div id = "div-a">
            <div id = "items" align = 'left'>
                <span id = 'item'> Name:   </span> {name} <br />
                <p></p>
                <span id = 'item'> Longitude:   </span> {long} <br />
                <p></p>
                <span id = 'item'> Latitude:    </span> {lat} <br />
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
        <div id = "div-b">
            <div id = "items" align = 'left'>
                <span id = 'item-b'> About the Data: </span> <br />
                <p></p>
                <p>The data in this visualization is a collection of California wildfires spanning from 2013 to 2019. The dataset was sourced from CalFire – California Department of Forest and Fire Protection.</p>
                <p>You can scroll and zoom in to better see the different wildfires in California. You can also hover over a particular circle to learn more about its corresponding wildfire.</p>
            </div>
        </div>

    </div>
</div>

<style>

    #wrapper {
        border: px solid blue;
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
        border-radius: 8px;
        min-height: 400px;
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
