<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import CalFire from './CalFire.svelte';
    import calfire_clean from '../data/calfire_clean.json';
    
    let tempData = [];

    onMount(async () => {
      const res = await fetch('calfire.csv'); 
      const csv = await res.text();
      tempData = d3.csvParse(csv, d3.autoType)
      console.log(tempData);
    });


</script>

<main>
    <h1>CAtching Fire</h1>
    <h3>California wildfires</h3>
    <CalFire {calfire_clean} />
</main>

<style>
    :rppt {
        --color-bg: #ffffff;

    }
  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
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
    font-size: 2em;
    font-weight: 300;
    line-height: 2;
  }
  h3 {
    font-size:1em;
    font-weight: 250;
    line-height: 1.5;
  }

</style>