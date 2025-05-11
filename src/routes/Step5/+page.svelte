<script>
    export let width = 1000; // default value if none passed
    export let height = 1000;
    export let style = ""; // optional style parameter
    export let chartScale = 0.8; // Control chart size (0.7 = 70% of original size)
    
    import { onMount } from 'svelte';
    import * as d3 from "d3";
    import Pie from '$lib/Pie.svelte';
    import { base } from '$app/paths';
    let pieData = [];
    let attorneyData = []
    let imageLoaded = false;
    let imgElement;
    let imgWidth, imgHeight;
    
    onMount(async() => {
        pieData = await d3.csv(`${base}/tenant_attorneys.csv`, d => ({
            label: d.name,
            value: +d.count
        }));
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
          const chartIframe = document.getElementById("datawrapper-chart-1E9iq");
          if (chartIframe && chartIframe.contentWindow === event.source) {
            for (let chartId in event.data["datawrapper-height"]) {
              const newHeight = event.data["datawrapper-height"][chartId] + "px";
              chartIframe.style.height = newHeight;
            }
          }
        }
      });
    });
  </script>
  <div class="container" style="width: {width}px; height: {height}px; {style}">
  <!--
  
    <div class="content-wrapper">
      <div class="image-side">
        {#if imageLoaded}
          <img src="Step4.png" alt="Step 4" class="main-image" />
        {:else}
          <div class="loading-placeholder">
            <img 
              bind:this={imgElement}
              class="preload-image" 
              src="Step4.png" 
              alt="Step 4" 
            />
            Loading...
          </div>
        {/if}
      </div>
      
      <div class="chart-side">
        <div class="chart-container" style="transform: scale({chartScale}); transform-origin: center center;">
          <iframe 
            title="Landlord Legal Representation" 
            aria-label="Donut Chart" 
            id="datawrapper-chart-1E9iq" 
            src="https://datawrapper.dwcdn.net/1E9iq/5/" 
            scrolling="no" 
            frameborder="0" 
            style="width: 100%; border: none;" 
            height="709">
          </iframe>
        </div>
      </div>
    </div>
  </div>
  -->
  <Pie data = {pieData}/>
    </div>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');
    
    .container {
      position: relative;
      overflow: hidden;
    }
    
    .content-wrapper {
      display: flex;
      width: 100%;
      height: 100%;
    }
    
    .image-side, .chart-side {
      flex: 1;
      display: flex;
      justify-content: center;
      align-items: center;
      overflow: hidden;
    }
    
    .chart-container {
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    
    .main-image {
      max-width: 100%;
      max-height: 100%;
      object-fit: contain;
    }
    
    .loading-placeholder {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      height: 100%;
      font-family: 'Bebas Neue', sans-serif;
      font-size: 24px;
    }
    
    .preload-image {
      visibility: hidden;
      position: absolute;
      width: 1px;
      height: 1px;
    }
  </style>
  
  