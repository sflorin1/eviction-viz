<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  import { onMount } from 'svelte';
  import * as d3 from "d3";
  import StackedBar from '$lib/StackedBar.svelte';
  import { base } from '$app/paths';
  let processedData = [];
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
    const raw = await d3.csv(`${base}/evictions_by_month.csv`, d3.autoType);
		processedData = raw.map(d => {
			const { month, ...rest } = d;
			return {
				month,
				...rest
			};
		});
  }
  );
</script>

  <body>
    <div class="content-container">
      <div class="text-left">
        <div class="text-left-inner">
        <div class="header-section">
          <div class="number">#1</div>
          <div> <span class="title-highlight">NOTICE TO QUIT</span></div>
        </div>
        
        <div class="text-section">
          <div class="body-text">
            <p>
              <span class="highlight">You've received a Notice to Quit from your landlord.</span> This tells you the reason they're seeking 
              to evict you and how long you have to respond.
            </p>
            <p>
              Not all eviction filings that happen in Boston are for failure to pay rent, in fact, many filings are what's called 
              <span class="highlight">"No Fault"</span> evictions.
            </p>
            <p>
              <span class="highlight">"No Fault"</span> means the landlord is asking the tenant to move out without claiming that the lease 
              was violated. This could be because the owner is selling the property, they want to renovate it, or something else.
              <span class="highlight">"Cause"</span> means the landlord claims there was a violation of the lease.
            </p>
          </div>
        </div>
        </div>
      </div>

      <div class="chart-right">
        <div class="chart">
          <StackedBar data = {processedData}/>
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
    width: 100%;
    height: 100%;
    min-width: 0;
  }

  .chart {
    align-self: center;
    width: 100%;
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
