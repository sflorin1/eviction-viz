<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  import { onMount } from 'svelte';
  
  let imageLoaded = false;
  let imgElement;
  let imgWidth, imgHeight;
  
  onMount(() => {
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
  });
</script>

<div class="container" style="width: {width}px; height: {height}px; {style}">
  <div class="content-wrapper">
    <div class="image-side">
      {#if imageLoaded}
        <img src="Step2.png" alt="Step 2" class="main-image" />
      {:else}
        <div class="loading-placeholder">
          <img 
            bind:this={imgElement}
            class="preload-image" 
            src="Step2.png" 
            alt="Step 2" 
          />
          Loading...
        </div>
      {/if}
    </div>
    
    <div class="chart-side">
      <iframe 
        title="Reasons For Eviction Filings" 
        aria-label="Stacked Columns" 
        id="datawrapper-chart-qJk9R" 
        src="https://datawrapper.dwcdn.net/qJk9R/2/" 
        scrolling="no" 
        frameborder="0" 
        class="chart-iframe">
      </iframe>
    </div>
  </div>
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
  
  .main-image {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
  }
  
  .chart-iframe {
    width: 100%;
    height: 100%;
    min-height: 592px;
    border: none;
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

