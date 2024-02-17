<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import CalFire from './CalFire.svelte';
    
    let tempData = [];

    onMount(async () => {
      const res = await fetch('calfire.csv'); 
      const csv = await res.text();
      tempData = d3.csvParse(csv, d3.autoType)
      console.log(tempData);
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
  <div class="visualization">
    <CalFire {tempData} />
  </div>
</main>

<style>


  .header-background {
    margin-inline: auto;
    top: 0;
    align-items: center;

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
  
</style>
