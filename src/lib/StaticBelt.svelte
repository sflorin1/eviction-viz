<!-- StaticBelt.svelte with conveyor belt and zoom functionality -->
<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import { fade } from 'svelte/transition';
    
    // Accept frames as a prop
    export let frames = [];
    export let count = 100; // Number of hugeline images for conveyor effect
    export let gap = -30; // Gap between conveyor belt images
    export let debug = false; // Debug mode
    
    // Paths for SVGs and images in a static folder
    const lineSvgPath = 'hugeline.svg';
    const boxSvgPath = 'box.svg';
    const noticeSvgPath = 'notice.svg';
    const evictionNoticeTitlePath = 'evictionnoticetitle.png';
    
    // Initial focus configuration
    const initialRowIndex = 7; // 8th row (0-based index)
    const initialBoxIndex = 6; // 7th box from right (0-based index)
    
    // Keep the original fixed positioning
    const boxRow = { 
      topPosition: 533, // Keep original pixel values
      rightOffset: 460, // Convert to number for calculations
      spacing: 0, 
      boxSize: '65px'
    };
    
    // Notice position matching the box
    const noticeRow = { 
      topPosition: 552, // Keep original pixel values
      rightOffset: 14100, // Convert to number for calculations
      spacing: 120, 
      size: '25px'
    };
    
    // Position for the focal point on the row
    const boxPosition = initialBoxIndex * 80; // Using spacing of 80px
    
    // Zoom functionality
    let scale = 3.5; // Start zoomed in
    let minScale = 1; // Fully zoomed out
    let maxScale = 3.5; // Max zoom in
    let zoomProgress = 0; // Progress of reveal (0-1)
    let showAllRows = false; // Flag to show all rows
    let showMapLink = false;

    // New values for eviction notice title animation
    const evictionTitleStartScroll = 500; // Start showing after 15px scroll
    const evictionTitleDuration = 900; // Show for 200px of scrolling
    const evictionTitleEndScroll = evictionTitleStartScroll + evictionTitleDuration;
    
    // New values for frame transitions - updated per requirements
    const frameVisibleDuration = 1500; // Each frame gets this many pixels of visible scroll space
    const frameFadeTransition = 250; // Each fade in/out gets this many pixels (faster transitions)
    const frameSeparation = 500; // Space between frames
    const frameSegmentSize = frameVisibleDuration + (frameFadeTransition * 2) + frameSeparation; // Total space per frame
    
    // Calculate frame transitions based on number of frames
    const frameCount = frames.length;
    const preZoomScrollHeight = frameCount * frameSegmentSize; // Total pre-zoom scroll height
    const zoomScrollHeight = 1500; // Zoom phase scroll height (per your requirement)
    const totalScrollHeight = preZoomScrollHeight + zoomScrollHeight; // Total scrollable height
    
    // Calculate frame transition points - completely recalculated
    const frameTransitions = frames.map((_, index) => {
        const startFadeIn = index * frameSegmentSize;
        const fullyVisible = startFadeIn + frameFadeTransition;
        const startFadeOut = fullyVisible + frameVisibleDuration;
        const fullyHidden = startFadeOut + frameFadeTransition;
        
        return {
            startFadeIn,
            fullyVisible,
            startFadeOut,
            fullyHidden
        };
    });
    
    // Full configuration (for later reveal)
    const boxRows = [
      { topPosition: 80, count: 13, rightOffset: 0, spacing: 80, boxSize: '65px' },
      { topPosition: 150, count: 13, rightOffset: 70, spacing: 80, boxSize: '65px' },
      { topPosition: 220, count: 13, rightOffset: 0, spacing: 80, boxSize: '65px' },
      { topPosition: 290, count: 13, rightOffset: 70, spacing: 80, boxSize: '65px' },
      { topPosition: 360, count: 13, rightOffset: 0, spacing: 80, boxSize: '65px' },
      { topPosition: 430, count: 13, rightOffset: 70, spacing: 80, boxSize: '65px' },
      { topPosition: 500, count: 13, rightOffset: 0, spacing: 80, boxSize: '65px' },
      { topPosition: 570, count: 13, rightOffset: 70, spacing: 80, boxSize: '65px' },
      { topPosition: 640, count: 13, rightOffset: 0, spacing: 80, boxSize: '65px' },
      { topPosition: 710, count: 13, rightOffset: 70, spacing: 80, boxSize: '65px' },
      { topPosition: 780, count: 13, rightOffset: 0, spacing: 80, boxSize: '65px' },
      { topPosition: 850, count: 13, rightOffset: 70, spacing: 80, boxSize: '65px' },
      { topPosition: 920, count: 13, rightOffset: 0, spacing: 80, boxSize: '65px' }
    ];

    const textNoticeRows = [
      { topPosition: 76, count: 13, rightOffset: 23, spacing: 120, size: '25px' },
      { topPosition: 146, count: 13, rightOffset: 92, spacing: 120, size: '25px' },
      { topPosition: 216, count: 13, rightOffset: 23, spacing: 120, size: '25px' },
      { topPosition: 286, count: 13, rightOffset: 92, spacing: 120, size: '25px' },
      { topPosition: 356, count: 13, rightOffset: 23, spacing: 120, size: '25px' },
      { topPosition: 426, count: 13, rightOffset: 92, spacing: 120, size: '25px' },
      { topPosition: 496, count: 13, rightOffset: 23, spacing: 120, size: '25px'},
      { topPosition: 566, count: 13, rightOffset: 92, spacing: 120, size: '25px' },
      { topPosition: 636, count: 13, rightOffset: 23, spacing: 120, size: '25px' },
      { topPosition: 706, count: 13, rightOffset: 92, spacing: 120, size: '25px' },
      { topPosition: 776, count: 13, rightOffset: 23, spacing: 120, size: '25px'},
      { topPosition: 846, count: 13, rightOffset: 92, spacing: 120, size: '25px' },
      { topPosition: 916, count: 13, rightOffset: 23, spacing: 120, size: '25px' }
    ];
    
    // Scroll tracking
    const scrollY = writable(0);
    let x = 0; // Horizontal translation for conveyor effect
    let currentScroll = 0; // Current scroll position
    let showEvictionTitle = false; // Flag to show the eviction notice title
    let evictionTitleOpacity = 0; // Opacity for the eviction notice title
    
    // Track if component is mounted
    let mounted = false;
    
    // Calculate current visible frame
    let currentFrameIndex = 0;
    
    // Add debug values
    let debugInfo = {
      scrollY: 0,
      x: 0,
      scale: scale,
      zoomProgress: zoomProgress,
      frameTransitions: frameTransitions,
      currentFrame: currentFrameIndex,
      windowWidth: 0,
      windowHeight: 0,
      showEvictionTitle: showEvictionTitle,
      evictionTitleOpacity: evictionTitleOpacity
    };
    
    // Zoom effect variables
    let zoomStartPosition = preZoomScrollHeight;
    let zoomStableEnd = zoomStartPosition + 1500; // Stay in place for 1500 pixels
    let zoomOutEnd = zoomStableEnd + zoomScrollHeight; // Then zoom out
    
    // Fix for box positioning to avoid animation glitches
    let boxScaleFixed = true; // Flag to disable box scaling animations
    
    // Helper function to calculate eviction title opacity based on scroll position
    function calculateEvictionTitleOpacity(scrollPos) {
        // Not visible before start
        if (scrollPos < evictionTitleStartScroll) {
            return 0;
        }
        // Fade in (quick fade in over 20px)
        else if (scrollPos >= evictionTitleStartScroll && scrollPos < evictionTitleStartScroll + 20) {
            return (scrollPos - evictionTitleStartScroll) / 20;
        }
        // Fully visible
        else if (scrollPos >= evictionTitleStartScroll + 20 && scrollPos < evictionTitleEndScroll - 20) {
            return 1;
        }
        // Fade out (quick fade out over 20px)
        else if (scrollPos >= evictionTitleEndScroll - 20 && scrollPos < evictionTitleEndScroll) {
            return 1 - ((scrollPos - (evictionTitleEndScroll - 20)) / 20);
        }
        // Not visible after end
        else {
            return 0;
        }
    }
    
    // New helper function to calculate frame opacity based on revised transition points
    function calculateFrameOpacity(frameIndex, scrollPos) {
        const transition = frameTransitions[frameIndex];
        
        // Fade in
        if (scrollPos >= transition.startFadeIn && scrollPos < transition.fullyVisible) {
            return (scrollPos - transition.startFadeIn) / frameFadeTransition;
        }
        // Fully visible
        else if (scrollPos >= transition.fullyVisible && scrollPos < transition.startFadeOut) {
            return 1;
        }
        // Fade out
        else if (scrollPos >= transition.startFadeOut && scrollPos < transition.fullyHidden) {
            return 1 - ((scrollPos - transition.startFadeOut) / frameFadeTransition);
        }
        // Not visible
        else {
            return 0;
        }
    }
    
    scrollY.subscribe(value => {
      currentScroll = value;
      
      // Calculate eviction title visibility
      showEvictionTitle = value >= evictionTitleStartScroll && value < evictionTitleEndScroll;
      evictionTitleOpacity = calculateEvictionTitleOpacity(value);
      showMapLink = value >= zoomOutEnd - 200;
      // First phase: Frame sequence and conveyor belt
      if (value < zoomStartPosition) {
        // Update conveyor belt position
        // CHANGED: Inverted the translation multiplier from negative to positive
        x = value * 0.5; // Side-scrolling effect now moves right as user scrolls down
        
        // Find current frame
        for (let i = 0; i < frameCount; i++) {
            if (value >= frameTransitions[i].startFadeIn && value < frameTransitions[i].fullyHidden) {
                currentFrameIndex = i;
                break;
            }
        }
        
        // Reset zoom settings during frame phase
        scale = maxScale;
        zoomProgress = 0;
        showAllRows = false;
      } 
      // Stable phase after frames but before zoom-out
      else if (value >= zoomStartPosition && value < zoomStableEnd) {
        // Keep conveyor belt position fixed
        // CHANGED: Inverted sign to match the new direction
        x = zoomStartPosition * 0.5;
        
        // Keep at full scale during stable phase
        scale = maxScale;
        
        // Instantly show the full grid (pop in)
        zoomProgress = 0.5; // Set to a value that will trigger showing the rows
        showAllRows = true;
      }
      // Zoom out phase
      else {
        // Keep conveyor belt position fixed
        // CHANGED: Inverted sign to match the new direction
        x = zoomStartPosition * 0.5;
        
        // Calculate zoom effect
        const zoomOutProgress = (value - zoomStableEnd) / (zoomOutEnd - zoomStableEnd);
        const clampedProgress = Math.min(1, Math.max(0, zoomOutProgress));
        
        scale = maxScale - clampedProgress * (maxScale - minScale);
        zoomProgress = clampedProgress;
        showAllRows = true;
      }
      
      // Update debug info
      debugInfo.scrollY = value;
      debugInfo.x = x;
      debugInfo.scale = scale;
      debugInfo.zoomProgress = zoomProgress;
      debugInfo.currentFrame = currentFrameIndex;
      debugInfo.showEvictionTitle = showEvictionTitle;
      debugInfo.evictionTitleOpacity = evictionTitleOpacity;
    });
    
    // Function to update window size in debug info
    function updateWindowSize() {
      if (typeof window !== 'undefined') {
        debugInfo.windowWidth = window.innerWidth;
        debugInfo.windowHeight = window.innerHeight;
      }
    }
    
    onMount(() => {
      // Reset scroll position
      window.scrollTo(0, 0);
      
      // Make sure we have proper body styling
      document.body.style.margin = '0';
      document.body.style.padding = '0';
      document.body.style.fontFamily = '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif';
      document.body.style.background = '#FCE9E0';
      document.body.style.minHeight = '100vh';
      document.body.style.width = '100%';
      document.body.style.overflowX = 'hidden';
      document.body.style.position = 'relative';
      
      // Set up scroll handler
      const handleScroll = () => {
        scrollY.set(window.scrollY);
      };
      
      window.addEventListener('scroll', handleScroll);
      window.addEventListener('resize', updateWindowSize);
      
      // Initial window size
      updateWindowSize();
      
      // Set component as mounted
      mounted = true;
      
      return () => {
        window.removeEventListener('scroll', handleScroll);
        window.removeEventListener('resize', updateWindowSize);
        mounted = false;
      };
    });
</script>

<main>
  <!-- Eviction Notice Title - only shows during specific scroll range -->
  {#if mounted}
    <div 
      class="eviction-title-container"
      style="opacity: {evictionTitleOpacity};"
    >
      <img src={evictionNoticeTitlePath} alt="Eviction Notice Title" />
    </div>
  {/if}

  <!-- Container with zoom transformation -->
  <div 
    class="content-wrapper"
    style="transform: scale({scale}); transform-origin: center center;"
  >
    <!-- Single box - Always visible during framed sequence -->
    <div 
      class="single-box-container" 
      style="
        top: {boxRow.topPosition}px;
        right: {boxPosition + boxRow.rightOffset}px;
        opacity: {currentScroll >= zoomStartPosition ? (zoomProgress < 0.9 ? 1 : Math.max(0, 3 * (1 - zoomProgress))) : 1};
      "
    >
      <div class="box-item" style="width: {boxRow.boxSize};">
        <img src={boxSvgPath} alt="Box SVG" />
      </div>
    </div>

    <!-- Single notice - Always visible during framed sequence -->
    <div 
      class="single-notice-container" 
      style="
        top: {noticeRow.topPosition}px;
        right: {boxPosition + noticeRow.rightOffset}px;
        opacity: {currentScroll >= zoomStartPosition ? (zoomProgress < 0.9 ? 1 : Math.max(0, 3 * (1 - zoomProgress))) : 1};
      "
    >
      <div class="text-notice-item" style="width: {noticeRow.size};">
        <img src={noticeSvgPath} alt="Notice SVG" />
      </div>
    </div>
    
    <!-- Phase 2: Show all rows when zoomed out in second phase -->
    {#if showAllRows && currentScroll >= zoomStartPosition}
      <!-- Multiple rows of Box SVGs -->
      {#each boxRows as row, rowIndex}
        <div 
          class="box-container" 
          style="
            height: {row.topPosition}px;
            right: 0;
            transform: translateX(-{row.rightOffset}px);
            opacity: 1; /* Instant pop-in, no fade */
            transition: transform 0.3s ease;
          "
        >
          <div class="box-row-wrapper">
            {#each Array(row.count) as _, i}
              <div 
                class="box-item"
                style="
                  margin-right: {i < row.count - 1 ? row.spacing + 'px' : '0'};
                  width: {row.boxSize};
                  transform: scale(1); /* Remove scaling animation for stable boxes */
                  transition: none; /* Remove transition for boxes */
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
            transform: translateX(-{row.rightOffset}px);
            opacity: 1; /* Instant pop-in, no fade */
            transition: none; /* Remove transition to prevent animation glitches */
          "
        >
          <div class="text-notice-wrapper">
            {#each Array(row.count) as _, i}
              <div 
                class="text-notice-item"
                style="
                  margin-right: {i < row.count - 1 ? row.spacing + 'px' : '0'};
                  width: {row.size};
                  transform: scale(1); /* Remove scaling animation for stable notices */
                  transition: none; /* Remove transition for notices */
                "
              >
                <img src={noticeSvgPath} alt="Notice SVG" />
              </div>
            {/each}
          </div>
        </div>
      {/each}
      
      <!-- Phase 2: Vertical line SVGs - always visible in phase 2 -->
      <div class="svg-container">
        {#each Array(13) as _, i}
          <div 
            class="svg-item"
            style="
              opacity: 1; /* Instant pop-in, no fade */
              transition: transform 0.3s ease;
            "
          >
            <img src={lineSvgPath} alt="Line SVG" />
          </div>
        {/each}
      </div>
    {/if}
  </div>
  
  <!-- Conveyor Belt implementation for hugelines -->
  <div 
    class="conveyor-container"
    style="opacity: {currentScroll >= zoomStartPosition ? Math.max(0, 1 - (currentScroll - zoomStartPosition) / 300) : 1};"
  >
    <div 
      class="conveyor-belt"
      style="transform: translateX({x}px); --gap: {gap}px"
    >
      <!-- CHANGED: Added flex-direction: row-reverse to the CSS to prepend items from left -->
      {#each Array(count) as _, i}
        <div class="conveyor-item">
          <img src={lineSvgPath} alt="Conveyor Line SVG" />
        </div>
      {/each}
    </div>
  </div>
  
  <!-- Center Frame Container - always maintains center position -->
  <div class="center-container">
    <!-- Display frames with updated fade in/out based on scroll position -->
    {#if frames.length > 0 && mounted}
      {#each frames as frame, index}
        <div 
          class="frame-container" 
          style="
            {frame.style || ''}
            opacity: {calculateFrameOpacity(index, currentScroll)};
          "
        >
          <svelte:component 
            this={frame.source} 
            width={frame.width} 
            height={frame.height}
            {...frame}
          />
        </div>
      {/each}
    {/if}
  </div>
  
  <!-- Instructions that change with scroll phases -->
  <div 
    class="instructions" 
    style="opacity: {
      currentScroll < zoomStartPosition 
        ? Math.max(0.3, 1 - currentScroll/1000)
        : (currentScroll < zoomStableEnd ? 1 : Math.max(0, 1 - (currentScroll - zoomStableEnd) / 500))
    };"
  >
    <p>
      {#if currentScroll < zoomStartPosition}
        Scroll down to see all steps
      {:else if currentScroll < zoomStableEnd}
        Continue scrolling to zoom out
      {:else}
        Continue scrolling to zoom out completely
      {/if}
    </p>
  </div>
  {#if showMapLink}
    <div 
      class="map-link-container"
      transition:fade={{ duration: 500 }}
    >
      <a href="./" target="_blank" rel="noopener noreferrer">
        Learn more about Serial Evictors in Boston
      </a>
    </div>
  {/if}
  <!-- Debug information -->
  {#if debug}
    <div class="debug-info">
      <p>Scroll: {debugInfo.scrollY}, x: {debugInfo.x}</p>
      <p>Scale: {debugInfo.scale.toFixed(2)}, Zoom: {debugInfo.zoomProgress.toFixed(2)}</p>
      <p>Current Frame: {debugInfo.currentFrame}</p>
      <p>Eviction Title: {debugInfo.showEvictionTitle ? 'Visible' : 'Hidden'} ({debugInfo.evictionTitleOpacity.toFixed(2)})</p>
      <p>Window: {debugInfo.windowWidth}px × {debugInfo.windowHeight}px</p>
      <p>Frames: {frames.length}, Pre-Zoom Height: {preZoomScrollHeight}</p>
      <p>Phase: {
        currentScroll < zoomStartPosition 
          ? "Frames Sequence" 
          : (currentScroll < zoomStableEnd ? "Grid Stable" : "Zoom Out")
      }</p>
    </div>
  {/if}
</main>

<!-- Make the page scrollable -->
<div class="scroll-container" style="height: {totalScrollHeight}vh;"></div>

<style>
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
    z-index: 10;
    transition: transform 0.1s linear; /* Change to linear for smoother zoom */
    will-change: transform; /* Optimize for transform changes */
  }

  /* Eviction Notice Title styles */
  .eviction-title-container {
    position: fixed;
    top: 40%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 100;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: opacity 0.2s ease;
    pointer-events: none;
  }

  .eviction-title-container img {
    max-width: 90%;
    height: auto;
    display: block;
  }

  /* Style for the initially focused single box */
  .single-box-container {
    position: absolute;
    z-index: 10;
    transition: opacity 0.5s ease;
  }

  /* Style for the initially focused single notice */
  .single-notice-container {
    position: absolute;
    z-index: 20;
    transition: opacity 0.5s ease;
  }
  
  /* Added center container to ensure frames stay centered */
  .center-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 50;
    pointer-events: none;
  }
  
  /* Frame container styles */
  .frame-container {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    pointer-events: all; /* Ensure links and interactions work */
    transition: opacity 0.2s ease; /* Faster transitions */
  }
  
  /* Debug info styles */
  .debug-info {
    position: fixed;
    top: 10px;
    left: 10px;
    background: rgba(0, 0, 0, 0.7);
    color: white;
    padding: 10px;
    border-radius: 5px;
    font-size: 14px;
    z-index: 1001;
  }
  
  /* Make sure links inside frames work */
  .frame-container :global(a) {
    cursor: pointer;
    pointer-events: auto;
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

  /* Box item styles */
  .box-item {
    display: flex;
    align-items: flex-end;
    height: 100%;
    flex-shrink: 0;
    position: relative;
    will-change: transform; /* Optimize for transform changes */
  }

  .box-item img {
    height: auto;
    max-height: 100%;
    width: 100%;
    max-width: 100%;
    display: block;
    object-fit: contain;
  }

  /* Text Notice item styles */
  .text-notice-item {
    display: flex;
    align-items: flex-end;
    height: 100%;
    flex-shrink: 0;
    will-change: transform; /* Optimize for transform changes */
  }

  .text-notice-item img {
    height: auto;
    max-height: 100%;
    width: 100%;
    max-width: 100%;
    display: block;
    object-fit: contain;
  }

  /* Box SVG styles for phase 2 */
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

  /* Text Notice SVG styles for phase 2 */
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

  /* Line SVG styles for phase 2 */
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
  
  /* Conveyor belt styles for hugelines */
  .conveyor-container {
    position: fixed;
    width: 100%;
    height: 100vh;
    z-index: 5;
    overflow: hidden;
    pointer-events: none;
    transition: opacity 0.5s ease;
  }
  
  .conveyor-belt {
    position: absolute;
    top: 730px; /* Original positioning */
    left: 0;
    width: 100%;
    display: flex;
    flex-direction: row-reverse; /* CHANGED: Reversed the flex direction to make items appear from left */
    transition: transform 0.05s ease-out;
    will-change: transform;
  }
  
  .conveyor-item {
    flex-shrink: 0;
    margin-left: var(--gap);
  }
  
  .conveyor-item:first-child {
    margin-left: 0;
  }
  
  .conveyor-item img {
    width: 5000px; /* Make the hugeline image wide enough for the belt */
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
    
    .eviction-title-container img {
      max-width: 80%;
    }
  }
  .map-link-container {
    position: fixed;
    bottom: 40px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 1000;
    padding: 15px 25px;
    border-radius: 8px;
    background-color: #4285F4;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  }
  
  .map-link-container a {
    color: white;
    text-decoration: none;
    font-size: 18px;
    font-weight: bold;
    letter-spacing: 0.5px;
    transition: all 0.3s ease;
  }
</style>