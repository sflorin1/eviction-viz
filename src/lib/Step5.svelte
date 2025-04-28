<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  // Import the image to get its dimensions
  import { onMount } from 'svelte';
  
  let imageLoaded = false;
  let imgElement;
  let imgWidth, imgHeight;
  let containerWidth, containerHeight;
  
  onMount(() => {
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
  });
</script>

<div class="image-wrapper" style="width: {width}px; height: {height}px; {style}">
  {#if imageLoaded}
    <div class="scaled-container" 
         style="width: {containerWidth}px; height: {containerHeight}px; 
                transform: translate(-50%, -50%) scale({containerWidth/imgWidth});
                left: 50%; top: 50%;">
      <img src="Screenshot 2025-04-28 at 12.22.22 PM.png" alt="Step 5" style="transform: translateX({-450}px);"
      />
    </div>
  {:else}
    <div class="loading-container">
      <img 
        bind:this={imgElement}
        class="preload-image" 
        src="Screenshot 2025-04-28 at 12.22.22 PM.png" 
        alt="Step 5" 
      />
    </div>
  {/if}
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');
  
  .image-wrapper {
    position: relative;
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .scaled-container {
    position: absolute;
    pointer-events: all;
    transform-origin: center;
  }
  
  .scaled-container img {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
  
  .loading-container {
    display: none;
  }
  
  .preload-image {
    visibility: hidden;
    position: absolute;
  }
</style>