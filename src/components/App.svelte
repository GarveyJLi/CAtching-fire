<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import CalFire from './CalFire.svelte';
    
    let tempData = [];

    onMount(async () => {
      const res = await fetch('calfire.csv'); 
      const csv = await res.text();
      tempData = d3.csvParse(csv, d3.autoType)
    });


</script>

<main>
  <div class = "heading">
    
    <div class="header-background">
      <header>
        <span><h1>CA-tching Fire</h1></span>
        <span><h3>Penny King & Garvey Li</h3></span>
      </header>
    </div>
    <p></p>
    
  </div>


  <div>
    <CalFire {tempData} />
  </div>
  

</main>

<div id="div2" align = 'left'>
  <div id = "div-b" align = 'left'>
    <div id = "items" align = 'left'>
        <span id = 'item-b'> Write Up: </span> <br />
        <p></p>
        <p>The data in this visualization is a collection of California wildfires spanning from 2013 to 2019. The dataset was sourced from CalFire – California Department of Forest and Fire Protection.
          We chose circle encodings for each fire with our x and y axis being longitude and latitude so our audience could see the relative positions of the wildfires and see a relationship of where wildfires tend to occur in California. Exploring the wildfires also is easier when you can visually see where the wildfires are happening. We used the radius/size of our circles as well as the color of our circles to signify the acres burned in that respective wildfire. Having 2 attributes to express the same channel (acres burned) more effectively to my audience. It also helps the audience not use as much cognitive work trying to see the severity of the wildfire. I also used a color scheme that was reminiscent of the color of fire and used a gradient from light colors (light yellow) to dark colors (dark red) to create an intuitive color gradient for my audience to see less severity to higher severity fires.
          For interaction techniques we chose to implement a scroll, pan, and zoom interactive component to our map-like graph. This lets the user see finer grain distances that they cannot normally see in graphs from a long distance perspective. We also incorporated a hover feature where hovering over a certain dot will allow the user to better learn information about that particular fire. This allows them to learn more from the graph from data that usually cannot be graphed (i.e. descriptions of the fires, etc). 
          An alternative design we thought about doing was a scatter plot where the x axis was average acres burned and the y axis was average fatalities and every dot was a “cause for fire” category. We thought that maybe knowing latitude and longitude was not exactly necessary and instead we could instead see patterns in how different categories have different severity. The tooltip could have been a graph showing all individual fires in a mini graph for that cause. However, there is a lot of variability in our data within a specific cause. This means that finding the average and plotting it will not be that meaningful. There are also not a lot of different cause categories, which would leave very little data points in the main visualization. We also believed that having latitude and longitude better helps the reader understand where in California these fires were starting, which made our final visualization more compelling to make. Having a hover tooltip also allows for more detail to be shown that otherwise could not be shown in this other visualization idea– for example the short paragraph fire descriptions cannot be shown in our other ideas. 
          To develop this application, we spent around 2 to 3 hours everyday at minimum, usually more (5 hours). The process began with brainstorming potential ideas for the project and exploring different datasets. After finalizing our idea to be about California wildfires, we conducted EDA to see which dataset would fit our needs best. After, we brainstormed multiple different graphics, debating the pros and cons of implementation and effectiveness of the visualizations. Then, most of our efforts went into coding the visualization. We started off with the circles, axes, hover tooltip, header design, and finally implementing the interactivity. The aspect of our visualization that took the most time was developing the interactive feature of scrolling, panning, and zooming. That took around 3 days to get right.
          It was difficult to get the axes to not move along with the dots and to control spacing and dot movements with the scrolls and pans. For splitting up the work amongst team members, our roles looked like the following: Penny explored datasets to use. Together Penny and Garvey brainstormed designs and project ideas for potential datasets. Penny and Garvey performed EDA on the final dataset to be turned into a csv for the web application. Penny designed the visualization and header and chose color schemes. Garvey and Penny both worked on coding and displaying the dots on the webpage. Garvey implemented the interactivity scroll, pan, and zoom features. Penny implemented the hover and statistics tool tips interactivity and Garvey helped a bit too.
          </p>
    </div>
  </div>
</div>

<style>

  div {
    font-family: 'Nunito', sans-serif;
  }
  .header-background {
    margin-inline: auto;
    top: 0;
    align-items: center;
    font-family: 'Nunito', sans-serif;
    width: 80%;
    height: 100px;
    max-width: 1200px;
    color: #fcfcfc;
    background: linear-gradient(to left, #FEC44F, #cf1702);
    border-radius: 12px; /* Adjust the border-radius for rounded corners */
  }

  main {
    text-align: center;
    font-family: 'Nunito', sans-serif;
    font-weight: 300;
    line-height: 2;
    font-size: 20px;
    color: var(--color-text);
  }

  header h1 {
    font-size: 50px;
    font-weight: 600;
    color: #fcfcfc;
    padding: 10px;
    text-align: left;
    width: 100%;
    margin-left:15px;
  }
  header h3 {
    font-size: 20px;
    font-weight: 250;
    color: white;
    transform: translateY(-255%);
    text-align: right;
    margin-right: 30px;
  }
  

  #div-b {
      display: inline-block;
      background: #f0f0f0;
      border-radius: 8px;
      width: 100%; /* Adjust the width to fit content */
      padding: 20px;
      text-align: left; /* Align the content to the left */
      margin-left: 12%; /* Add some left margin for spacing */
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
    #div2 {
        vertical-align:left;
        display: inline-block;
        width: 78%;
        font-size:0.605em;
        text-align: left; /* Align content to the left */
        font-size:.75em
    }
</style>
