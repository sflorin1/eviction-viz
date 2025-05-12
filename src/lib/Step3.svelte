<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  import { onMount } from 'svelte';
  import Pie from '$lib/Pie.svelte';
  import * as d3 from "d3";
  import { base } from '$app/paths';
  let pieData = [];
    
  let imageLoaded = false;
  let imgElement;
  let imgWidth, imgHeight;
  
  onMount(async () => {
    if (imgElement) {
      imgElement.onload = () => {
        // Get natural dimensions of the image
        imgWidth = imgElement.naturalWidth;
        imgHeight = imgElement.naturalHeight;
        imageLoaded = true;
      };
      
    }
    
    // Handle Datawrapper iframe height adjustments
    window.addEventListener("message", (event) => {
      if (event.data["datawrapper-height"]) {
        const chartIframe = document.getElementById("datawrapper-chart-qJk9R");
        if (chartIframe && chartIframe.contentWindow === event.source) {
          for (let chartId in event.data["datawrapper-height"]) {
            const newHeight = event.data["datawrapper-height"][chartId] + "px";
            chartIframe.style.height = newHeight;
          }
        }
      }
    });
    pieData = await d3.csv(`${base}/alternate_landlord_attorneys.csv`, d => ({
            label: d.name,
            value: +d.count
      }));
  });
</script>

<body>
  <div class="content-container">
    <div class="text-left">
      <div class="text-left-inner">
      <div class="header-section">
        <div class="number">#3</div>
        <div> <span class="title-highlight">LEGAL DEFENSE</span></div>
      </div>
      
      <div class="text-section">
        <div class="body-text">
          <p>
            Lets imagine you’re one of the <span class="highlight">90% of tenants in Boston who do not have legal representation</span>
            for their eviction case. This means that when your court date comes, you will defend yourself against eviction.
          </p>
          <p>
            At the same time, <span class="highlight">86% of landlords in Boston Housing Court cases use legal representation.</span> 
            In fact, your landlord is likely to use an attorney who works on <i>many</i> eviction cases for landlords in Boston—  
            <i>an eviction expert.</i>
          </p>
          <p>
            Our analysis found that single Boston-Area attorney represented landlords in <span class="highlight">688 eviction filings in the two year span</span>
            between 2020–2022.
          </p>
        </div>
      </div>
      </div>
    </div>

    <div class="chart-right">
      <div class="chart">
        <Pie data = {pieData}/>
      </div>
    </div>
  </div>
</body>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');

  .content-container {
    width: 90%;
    margin-inline: auto;
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: auto;
    column-gap: 1rem;
    max-height: calc(100vh - 320px);
    overflow: hidden;
    padding-top: 15px;
    gap: 3rem;
  }

  .text-left {
    display: grid;
    grid-template-rows: subgrid;
    grid-row: span 2;
    grid-column: 1;
    width: 100%;
    height: 100%;
    min-width: 0;
  }

  .chart-right {
    display: grid;
    grid-template-rows: subgrid;
    grid-column: 2;
    grid-row: 1 / span 2;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    min-width: 0;
  }

  .chart {
    align-self: center;
    width: 150%;
    height: 100%;
  }

  .header-section {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 0.5rem;
  }

  .number {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 52px;
    font-weight: bold;
    margin-right: 15px;
    line-height: 1;
    color: #14110F;
    width: 1fr;
  }

  .body-text {
    color: #14110F;
    font-size: clamp(12px, 3.5vw, 15px);
    font-family: 'Geist Mono', monospace;
    font-weight: 400;
    word-wrap: break-word;
    line-height: 1.6;
    margin-top: 3.5rem;
  }

  .highlight {
    background-color: #06D6A0;
    padding: 0 4px;
    border-radius: 3px;
  }

  .title-highlight {
    background-color: #000000;
    color: white;
    padding: 7px 20px;
    border-radius: 10px;
    font-family: 'Bebas Neue', sans-serif;
    font-size: 40px;
    letter-spacing: 1.3px;
  }
</style>
