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
        <div id = "question">
          <h4> Do different areas of California experience wildifire severity differently? (2013-2019) </h4>
        <span><h3>Penny King & Garvey Li</h3></span>
      </div>
      </header>
    </div>
    <p></p>
  </div>
  
  <div>
    <h2>Pan & Zoom to explore the map. Hover over to learn more about the wildfire.</h2>
  </div>

  <div>
    <CalFire {tempData} />
  </div>
  

</main>


<style>
  h2 {
    font-size: 12px;
    font-weight: 100;
    color: grey;
    padding-bottom: -10px;
    padding-top: -10px;
    padding-left:5px;
    text-align: left;
    width: 100%;
    margin-left:10%;
  }
  #question{
    font-size:100px
  }
  div {
    font-family: 'Nunito', sans-serif;
  }
  .header-background {
    margin-inline: auto;
    top: 0;
    align-items: center;
    font-family: 'Nunito', sans-serif;
    width: 80%;
    height: 120px;
    max-width: 1200px;
    color: #fcfcfc;
    background: linear-gradient(to left, #FEC44F, #cf1702);
    border-radius: 12px; /* Adjust the border-radius for rounded corners */
  }

  main {
    text-align: center;
    font-family: 'Nunito', sans-serif;
    font-weight: 100;
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
    font-size: 15px;
    font-weight: 250;
    color: white;
    transform: translateY(-355%);
    text-align: right;
    margin-right: 30px;
    margin-left: 30px;
  }
  
  header h4{
    font-size: 12px;
    font-weight: 250;
    color: white;
    transform: translateY(-300%);
    text-align: left;
    margin-left: 30px;
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
