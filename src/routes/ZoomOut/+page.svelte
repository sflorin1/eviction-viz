// App.svelte
<script>
  import { onMount } from 'svelte';
  
  // Paths for SVGs in a static folder (SvelteKit standard)
  const lineSvgPath = '/hugeline.svg';
  const boxSvgPath = '/box.svg';
  const noticeSvgPath = '/notice.svg';
  
  // Initial focus configuration
  const initialRowIndex = 7; // 8th row (0-based index)
  const initialBoxIndex = 6; // 7th box from right (0-based index)
  
  // Box positions (we'll use a simplified array for initial view)
  const boxRow = { 
    topPosition: 533, // This is the 8th row's position (index 7)
    rightOffset: '170px',
    spacing: 0, 
    boxSize: '65px'
  };
  
  // Notice position matching the box
  const noticeRow = { 
    topPosition: 552, // Matching 8th row
    rightOffset: '192px',
    spacing: 120, 
    size: '25px'
  };
  
  // Full configuration (for later reveal)
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
  
  // View state
  let scale = 3.5; // Start zoomed in
  let minScale = 1; // Fully zoomed out
  let maxScale = 3.5; // Max zoom in
  let zoomProgress = 0; // Progress of reveal (0-1)
  let showAllRows = false; // Flag to show all rows
  
  // Position for the focal point on the row
  let boxPosition = initialBoxIndex * 80; // Using spacing of 80px

  // Scroll handling
  let scrollY = 0;
  let totalScrollHeight = 1500;
  
  function handleScroll() {
    scrollY = window.scrollY;
    
    // Calculate scale based on scroll position
    scale = maxScale - (scrollY / totalScrollHeight) * (maxScale - minScale);
    scale = Math.max(minScale, Math.min(maxScale, scale));
    
    // Calculate reveal progress
    zoomProgress = 1 - ((scale - minScale) / (maxScale - minScale));
    
    // Show all rows when zoomed out enough
    showAllRows = zoomProgress > 0.1;
  }
  
  onMount(() => {
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
  <!-- Container with zoom transformation -->
  <div 
    class="content-wrapper"
    style="transform: scale({scale}); transform-origin: center center;"
  >
    <!-- Initial focused state - always visible but fades out after everything appears -->
    <div 
      class="single-box-container" 
      style="
        top: {boxRow.topPosition}px;
        right: {boxPosition + parseInt(boxRow.rightOffset)}px;
        opacity: {zoomProgress < 0.9 ? 1 : Math.max(0, 3 * (1 - zoomProgress))};
        transition: opacity 0.5s ease;
      "
    >
      <div class="box-item" style="width: {boxRow.boxSize};">
        <img src={boxSvgPath} alt="Box SVG" />
      </div>
    </div>

    <!-- Single notice - always visible but fades out after everything appears -->
    <div 
      class="single-notice-container" 
      style="
        top: {noticeRow.topPosition}px;
        right: {boxPosition + parseInt(noticeRow.rightOffset)}px;
        opacity: {zoomProgress < 0.9 ? 1 : Math.max(0, 3 * (1 - zoomProgress))};
        transition: opacity 0.5s ease;
      "
    >
      <div class="text-notice-item" style="width: {noticeRow.size};">
        <img src={noticeSvgPath} alt="Notice SVG" />
      </div>
    </div>

    <!-- Rest of the boxes and notices - shown when zoomed out -->
    {#if showAllRows}
      <!-- Multiple rows of Box SVGs -->
      {#each boxRows as row, rowIndex}
        <div 
          class="box-container" 
          style="
            height: {row.topPosition}px;
            right: 0;
            transform: translateX(-{row.rightOffset});
            opacity: {Math.min(1, (zoomProgress - 0.3) * 1.5)};
            transition: opacity 0.5s ease;
          "
        >
          <div class="box-row-wrapper">
            {#each Array(row.count) as _, i}
              <div 
                class="box-item"
                style="
                  margin-right: {i < row.count - 1 ? row.spacing + 'px' : '0'};
                  width: {row.boxSize};
                  /* Highlight the same box position in all rows */
                  transform: scale({
                    (row.count - 1 - i) === initialBoxIndex ? 
                    (1 + (0.2 * (1 - Math.min(1, zoomProgress * 2)))) : 1
                  });
                  transition: transform 0.5s ease;
                "
              >
                <img src={boxSvgPath} alt="Box SVG" />
              </div>
            {/each}
          </div>
        </div>
      {/each}

      <!-- Text notice SVGs -->
      {#each textNoticeRows as row, rowIndex}
        <div 
          class="text-notice-container" 
          style="
            height: {row.topPosition}px;
            right: 0;
            transform: translateX(-{row.rightOffset});
            opacity: {Math.min(1, (zoomProgress - 0.3) * 1.5)};
            transition: opacity 0.5s ease;
          "
        >
          <div class="text-notice-wrapper">
            {#each Array(row.count) as _, i}
              <div 
                class="text-notice-item"
                style="
                  margin-right: {i < row.count - 1 ? row.spacing + 'px' : '0'};
                  width: {row.size};
                  /* Highlight the same notice position in all rows */
                  transform: scale({
                    (row.count - 1 - i) === initialBoxIndex ? 
                    (1 + (0.2 * (1 - Math.min(1, zoomProgress * 2)))) : 1
                  });
                  transition: transform 0.5s ease;
                "
              >
                <img src={noticeSvgPath} alt="Notice SVG" />
              </div>
            {/each}
          </div>
        </div>
      {/each}
    {/if}
    
    <!-- Vertical line SVGs - always visible, opacity varies -->
    <div class="svg-container">
      {#each Array(13) as _, i}
        <div 
          class="svg-item"
          style="
            opacity: {i === initialRowIndex ? 1 : Math.min(1, zoomProgress * 3)};
            transition: opacity 0.5s ease;
          "
        >
          <img src={lineSvgPath} alt="Line SVG" />
        </div>
      {/each}
    </div>
  </div>
  
  <!-- Instructions -->
  {#if scale > 1.5}
    <div class="instructions" style="opacity: {2 - zoomProgress};">
      <p>Scroll down to zoom out and reveal more</p>
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
    overflow-x: hidden;
    position: relative;
  }

  main {
    width: 100%;
    height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    position: fixed;
    top: 0;
    left: 0;
    overflow: visible;
  }
  
  .content-wrapper {
    position: relative;
    width: 100%;
    height: 100%;
    transition: transform 0.1s ease-out;
  }

  /* Style for the initially focused single box */
  .single-box-container {
    position: absolute;
    z-index: 10;
  }

  /* Style for the initially focused single notice */
  .single-notice-container {
    position: absolute;
    z-index: 20;
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
    pointer-events: none;
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
    justify-content: flex-end;
    z-index: 10;
    box-sizing: border-box;
    overflow: visible;
  }

  .box-row-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: flex-end;
    width: 100%;
  }

  .box-item {
    display: flex;
    align-items: flex-end;
    height: 100%;
    flex-shrink: 0;
    position: relative;
  }

  .box-item img {
    height: auto;
    max-height: 100%;
    width: 100%;
    max-width: 100%;
    display: block;
    object-fit: contain;
  }

  /* Text Notice SVG styles */
  .text-notice-container {
    position: absolute;
    top: 0;
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: flex-end;
    z-index: 20;
    box-sizing: border-box;
    overflow: visible;
  }

  .text-notice-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: flex-end;
    width: 100%;
  }

  .text-notice-item {
    display: flex;
    align-items: flex-end;
    height: 100%;
    flex-shrink: 0;
  }

  .text-notice-item img {
    height: auto;
    max-height: 100%;
    width: 100%;
    max-width: 100%;
    display: block;
    object-fit: contain;
  }

  /* Line SVG styles */
  .svg-container {
    display: flex;
    flex-direction: column;
    width: 100%;
    padding-top: 53px;
    padding-bottom: 10px;
    height: calc(114vh - 228px);
    position: relative;
    z-index: 5;
  }

  .svg-item {
    width: 100%;
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: visible;
  }

  .svg-item img {
    width: 100%;
    height: auto;
    display: block;
  }

  /* Media query for responsive behavior */
  @media (max-width: 768px) {
    .box-container, .text-notice-container, .single-box-container, .single-notice-container {
      justify-content: flex-end;
    }
    
    .box-item, .text-notice-item {
      max-width: 80px;
    }
  }
</style>