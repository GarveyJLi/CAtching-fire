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

  <div class="header-background"></div>
  <div class="blocked">
    <header>
      <h1>CA-tching Fire</h1>
    </header>
  </div>

    <h3>California wildfires</h3>

    <div>
      <CalFire {tempData} />
    </div>


</main>

<style>
    :rppt {
        --color-bg: #ffffff;
    }

  *,
  *::before,
  *::after {
    font-family: Verdana;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  .header-background {
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 80%;
    max-width: 1200px;
    height: 120px;
    background: linear-gradient(to left, #FEC44F, #cf1702);
    //background-color: #FEC44F;
    z-index: -1;
    border-radius: 12px; /* Adjust the border-radius for rounded corners */
  }

  main {
    text-align: center;
    font-family: 'Nunito', sans-serif;
    font-weight: 300;
    line-height: 2;
    font-size: 24px;
    color: var(--color-text);
  }
  h1 {
    font-size: 1em;
    font-weight: 300;
    line-height: 2;
  }
  h3 {
    font-size:0.5em;
    font-weight: 250;
    line-height: 1.5;
  }
  div {
    margin-top: 20px;
  }
  header h1 {
    font-size: 70px;
    font-weight: 600;
    //background-image: linear-gradient(to left, #FEC44F, #cf1702);
    color: #fcfcfc;
    background-clip: text;
    -webkit-background-clip: text;
}
  

</style>