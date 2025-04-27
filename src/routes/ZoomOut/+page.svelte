// App.svelte
<script>
  import { onMount } from 'svelte';
  
  // Paths for SVGs in a static folder (SvelteKit standard)
  const lineSvgPath = '/hugeline.svg';
  const boxSvgPath = '/box.svg';
  const noticeSvgPath = '/notice.svg';
  
  // Enhanced configuration for each row of boxes
  const boxRows = [
    { topPosition: 80, count: 13, rightOffset: '0px', spacing: 80, boxSize: '65px' },
    { topPosition: 150, count: 13, rightOffset: '70px', spacing: 80, boxSize: '65px' },
    { topPosition: 220, count: 13, rightOffset: '0px', spacing: 80, boxSize: '65px' },
    { topPosition: 290, count: 13, rightOffset: '70px', spacing: 80, boxSize: '65px' },
    { topPosition: 360, count: 13, rightOffset: '0px', spacing: 80, boxSize: '65px' },
    { topPosition: 430, count: 13, rightOffset: '70px', spacing: 80, boxSize: '65px' },
    { topPosition: 500, count: 13, rightOffset: '0px', spacing: 80, boxSize: '65px' },
    { topPosition: 570, count: 13, rightOffset: '70px', spacing: 80, boxSize: '65px' },
    { topPosition: 640, count: 13, rightOffset: '0px', spacing: 80, boxSize: '65px' },
    { topPosition: 710, count: 13, rightOffset: '70px', spacing: 80, boxSize: '65px' },
    { topPosition: 780, count: 13, rightOffset: '0px', spacing: 80, boxSize: '65px' },
    { topPosition: 850, count: 13, rightOffset: '70px', spacing: 80, boxSize: '65px' },
    { topPosition: 920, count: 13, rightOffset: '0px', spacing: 80, boxSize: '65px' }
  ];

  const textNoticeRows = [
    { topPosition: 76, count: 13, rightOffset: '23px', spacing: 120, size: '25px' },
    { topPosition: 146, count: 13, rightOffset: '92px', spacing: 120, size: '25px' },
    { topPosition: 216, count: 13, rightOffset: '23px', spacing: 120, size: '25px' },
    { topPosition: 286, count: 13, rightOffset: '92px', spacing: 120, size: '25px' },
    { topPosition: 356, count: 13, rightOffset: '23px', spacing: 120, size: '25px' },
    { topPosition: 426, count: 13, rightOffset: '92px', spacing: 120, size: '25px' },
    { topPosition: 496, count: 13, rightOffset: '23px', spacing: 120, size: '25px'},
    { topPosition: 566, count: 13, rightOffset: '92px', spacing: 120, size: '25px' },
    { topPosition: 636, count: 13, rightOffset: '23px', spacing: 120, size: '25px' },
    { topPosition: 706, count: 13, rightOffset: '92px', spacing: 120, size: '25px' },
    { topPosition: 776, count: 13, rightOffset: '23px', spacing: 120, size: '25px'},
    { topPosition: 846, count: 13, rightOffset: '92px', spacing: 120, size: '25px' },
    { topPosition: 916, count: 13, rightOffset: '23px', spacing: 120, size: '25px' }
  ];
  
  // Zoom scale state
  let scale = 3.5; // Start zoomed in (2.5x)
  let minScale = 1; // Fully zoomed out scale (normal size)
  let maxScale = 3.5; // Maximum zoom-in
  
  // Scroll tracking
  let scrollY = 0;
  let totalScrollHeight = 1000; // Total amount of scrolling needed to fully zoom out
  
  // Update scale based on scroll position
  function handleScroll() {
    scrollY = window.scrollY;
    
    // Calculate scale based on scroll position
    // Map scroll from 0 -> totalScrollHeight to scale from maxScale -> minScale
    scale = maxScale - (scrollY / totalScrollHeight) * (maxScale - minScale);
    
    // Clamp scale between min and max values
    scale = Math.max(minScale, Math.min(maxScale, scale));
  }
  
  onMount(() => {
    // Set initial zoom
    scale = maxScale;
    
    // Add scroll event listener
    window.addEventListener('scroll', handleScroll);
    
    // Set document height to allow enough scrolling
    document.body.style.height = `${totalScrollHeight + window.innerHeight}px`;
    
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  });
</script>

<svelte:window on:scroll={handleScroll} />

<main>
  <div 
    class="content-wrapper"
    style="transform: scale({scale}); transform-origin: center center;"
  >
    <!-- Multiple rows of Box SVGs with customizable positioning -->
    {#each boxRows as row, rowIndex}
      <div 
        class="box-container" 
        style="
          height: {row.topPosition}px;
          right: 0;
          transform: translateX(-{row.rightOffset});
        "
      >
        <div class="box-row-wrapper">
          <!-- Using a loop to render the specified number of box SVGs horizontally -->
          {#each Array(row.count) as _, i}
            <div 
              class="box-item"
              style="
                margin-right: {i < row.count - 1 ? row.spacing + 'px' : '0'};
                width: {row.boxSize !== 'auto' ? row.boxSize : 'auto'};
              "
            >
              <img src={boxSvgPath} alt="Box SVG" />
            </div>
          {/each}
        </div>
      </div>
    {/each}

    <!-- Text notice SVGs on top of boxes -->
    {#each textNoticeRows as row, rowIndex}
      <div 
        class="text-notice-container" 
        style="
          height: {row.topPosition}px;
          right: 0;
          transform: translateX(-{row.rightOffset});
        "
      >
        <div class="text-notice-wrapper">
          <!-- Using a loop to render the specified number of text notice SVGs horizontally -->
          {#each Array(row.count) as _, i}
            <div 
              class="text-notice-item"
              style="
                margin-right: {i < row.count - 1 ? row.spacing + 'px' : '0'};
                width: {row.size !== 'auto' ? row.size : 'auto'};
              "
            >
              <img src={noticeSvgPath} alt="Notice SVG" />
            </div>
          {/each}
        </div>
      </div>
    {/each}
    
    <!-- Vertical line SVGs -->
    <div class="svg-container">
      <!-- Using a loop to render 13 line SVGs vertically -->
      {#each Array(13) as _, i}
        <div class="svg-item">
          <img src={lineSvgPath} alt="Line SVG" />
        </div>
      {/each}
    </div>
  </div>
  
  <!-- Overlay instructions (visible only when zoomed in) -->
  {#if scale > 1.5}
    <div class="instructions">
      <p>Scroll down to zoom out</p>
    </div>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
    background: #FCE9E0;
    min-height: 100vh;
    width: 100%;
    overflow-x: hidden; /* Hide horizontal overflow */
    position: relative; /* Set position context for absolute positioning */
  }

  main {
    width: 100%;
    height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center; /* Center content vertically */
    position: fixed; /* Keep content fixed while scrolling */
    top: 0;
    left: 0;
    overflow: hidden;
  }
  
  .content-wrapper {
    position: relative;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center; /* Center children vertically */
    justify-content: center; /* Center children horizontally */
    transition: transform 0.1s ease-out; /* Smooth transition when zooming */
  }

  /* Instructions overlay styles */
  .instructions {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.7);
    color: white;
    padding: 10px 20px;
    border-radius: 20px;
    font-size: 16px;
    pointer-events: none; /* Doesn't interfere with clicks */
    z-index: 1000;
    opacity: 0.8;
    transition: opacity 0.3s ease;
  }

  /* Box SVG styles */
  .box-container {
    position: absolute;
    top: 0;
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: flex-end; /* Align boxes from the right side */
    z-index: 10; /* Make boxes appear above the lines */
    box-sizing: border-box;
    overflow: hidden; /* Hide boxes that are moved outside the container */
  }

  .box-row-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: flex-end; /* Align boxes from the right side */
    width: 100%;
  }

  .box-item {
    display: flex;
    align-items: flex-end; /* Align items to the bottom */
    height: 100%;
    flex-shrink: 0; /* Prevent items from shrinking */
    position: relative; /* For maintaining position during scaling */
  }

  .box-item img {
    height: auto;
    max-height: 100%;
    width: 100%; /* Make image fill its container */
    max-width: 100%;
    display: block;
    object-fit: contain; /* Maintain aspect ratio */
  }

  /* Text Notice SVG styles */
  .text-notice-container {
    position: absolute;
    top: 0;
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: flex-end; /* Align items from the right side */
    z-index: 20; /* Make text notices appear above the boxes */
    box-sizing: border-box;
    overflow: hidden; /* Hide items that are moved outside the container */
  }

  .text-notice-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: flex-end; /* Align items from the right side */
    width: 100%;
  }

  .text-notice-item {
    display: flex;
    align-items: flex-end; /* Align items to the bottom */
    height: 100%;
    flex-shrink: 0; /* Prevent items from shrinking */
  }

  .text-notice-item img {
    height: auto;
    max-height: 100%;
    width: 100%; /* Make image fill its container */
    max-width: 100%;
    display: block;
    object-fit: contain; /* Maintain aspect ratio */
  }

  /* Line SVG styles */
  .svg-container {
    display: flex;
    flex-direction: column;
    width: 100%;
    padding-top: 30px; /* First SVG starts from the top */
    padding-bottom: 10px;
    height: calc(114vh - 228px);
  }

  .svg-item {
    width: 100%; /* Make each item take full width */
    flex: 1; /* Distribute available space evenly between items */
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: visible; /* Allow SVG to overflow if needed */
  }

  .svg-item img {
    width: 100%; /* Make image fill the full width */
    height: auto; /* Maintain aspect ratio */
    display: block;
  }

  /* Media query for responsive behavior - ensure both boxes and notices scale together */
  @media (max-width: 768px) {
    .box-container, .text-notice-container {
      justify-content: flex-end; /* Keep right alignment on smaller screens */
    }
    
    .box-item, .text-notice-item {
      /* Apply the same max-width to both for proportional scaling */
      max-width: 80px; /* Example value - adjust as needed */
    }
  }
</style>