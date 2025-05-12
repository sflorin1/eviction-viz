<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  // Import the image to get its dimensions
  import { onMount } from 'svelte';
  import Pie from '$lib/Pie.svelte';
  import * as d3 from "d3";
  import { base } from '$app/paths';
  let pieData = [];
  
  let imageLoaded = false;
  let imgElement;
  let imgWidth, imgHeight;
  let containerWidth, containerHeight;
  
  onMount(async () => {
    if (imgElement) {
      imgElement.onload = () => {
        // Get natural dimensions of the image
        imgWidth = imgElement.naturalWidth;
        imgHeight = imgElement.naturalHeight;
        
        // Calculate container dimensions based on proportional scaling
        const scaleRatio = Math.min(width / imgWidth, height / imgHeight);
        containerWidth = imgWidth * scaleRatio;
        containerHeight = imgHeight * scaleRatio;
        
        imageLoaded = true;
      };
    }
    pieData = await d3.csv(`${base}/adjusted_tenant_attorneys.csv`, d => ({
            label: d.name,
            value: +d.count
        }));
  }
  );
</script>

<body>
  <div class="content-container">
    <div class="text-left">
      <div class="text-left-inner">
      <div class="header-section">
        <div class="number">#6</div>
        <div> <span class="title-highlight">EXECUTION & MOVING DAY</span></div>
      </div>
      
      <div class="text-section">
        <div class="body-text">
          <p>
            If you lose the appeal, you must 
            <span class="highlight">move out within 48 hours.</span> A marshal will post a notice, and 
            lockouts typically happen about nine days later.
          </p>
          <p>
            Move-out, locksmith, and storage fees — 
            <span class="highlight">averaging $1,150</span> — are billed to the tenant.
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
    width: 80%;
    height: 100%;
    min-width: 0;
  }

  .chart {
    align-self: center;
    width: 170%;
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

  .footnote {
    color: #14110F;
    font-family: 'Geist Mono', monospace;
    font-size: 11px;
    word-wrap: break-word;
    line-height: 1.7;
  }

  a {
    color: #3E3E3D;
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