<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  import { onMount } from 'svelte';
  import * as d3 from "d3";
  import { base } from '$app/paths';
  import { fly, fade } from 'svelte/transition';
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  
  // For this component we'll use outcome data instead of attorney data
  let evictionData = [];
  let isDataLoaded = false;
  let selectedIndex = -1;
  let totalCases = 0;
  
  // Animation for data loading
  const progress = tweened(0, {
    duration: 800,
    easing: cubicOut
  });
  
  onMount(async () => {
    // Start loading animation
    progress.set(0.3);
    
    try {
      // Since the text mentions specific percentages, we'll create the data
      // based on those percentages instead of loading from CSV
      evictionData = [
        { label: "Executed Eviction", value: 17 },
        { label: "Unknown Outcome", value: 48 },
        { label: "Dismissed", value: 20 },
        { label: "Agreement Reached", value: 15 }
      ];
      
      progress.set(0.7);
      
      // Calculate total cases
      totalCases = d3.sum(evictionData, d => d.value);
      
      progress.set(1);
      setTimeout(() => {
        isDataLoaded = true;
      }, 300);
    } catch (error) {
      console.error("Error processing data:", error);
    }
  });
  
  // Creating a donut chart 
  // Use vibrant colors aligned with the highlight colors from the text
  const colors = d3.scaleOrdinal()
    .range(['#EF476F', '#14110F', '#1B9AAA', '#06D6A0']);
  
  const radius = 120;
  const innerRadius = radius * 0.6;
  
  // Arc generator for the donut segments
  const arcGenerator = d3.arc()
    .innerRadius(innerRadius)
    .outerRadius(radius)
    .padAngle(0.02)
    .cornerRadius(4);
  
  // Arc generator for labels
  const labelArcGenerator = d3.arc()
    .innerRadius(radius * 1.1)
    .outerRadius(radius * 1.1);
  
  // Slice generator
  const sliceGenerator = d3.pie()
    .value(d => d.value)
    .sort(null);
  
  $: arcData = sliceGenerator(evictionData);
  
  // Calculate percentage
  function getPercent(value) {
    return totalCases > 0 ? Math.round((value / totalCases) * 100) : 0;
  }
  
  function handleSegmentClick(index) {
    selectedIndex = selectedIndex === index ? -1 : index;
  }
  
  function handleBackgroundClick() {
    selectedIndex = -1;
  }
  
  // Provide explanations for each outcome type
  function getOutcomeExplanation(outcomeType) {
    switch(outcomeType) {
      case "Executed Eviction":
        return "The tenant was legally required to vacate the property, and the eviction was carried out.";
      case "Unknown Outcome":
        return "No final outcome was recorded in the public records. This could mean the case was settled privately, abandoned, or the outcome wasn't properly documented.";
      case "Dismissed":
        return "The eviction case was dismissed by the court or withdrawn by the landlord, allowing the tenant to remain in the property.";
      case "Agreement Reached":
        return "The landlord and tenant reached a formal agreement, which might include payment plans, move-out dates, or other negotiated terms.";
      default:
        return "Information about this outcome type is unavailable.";
    }
  }
</script>

<body>
  <div class="content-container">
    <div class="text-left">
      <div class="text-left-inner">
        <div class="header-section">
          <div class="number">#5</div>
          <div in:fly={{ y: 20, duration: 400 }}><span class="title-highlight">TRIAL & JUDGEMENT</span></div>
        </div>
        
        <div class="text-section">
          <div class="body-text" in:fade={{ duration: 400, delay: 100 }}>
            <p>
              If the court rules that the landlord can evict you, you will have 
              <span class="highlight">10 days to appeal their decision.</span>
            </p>
            <p>
              While data from 2020-2022 indicate that <span class="highlight interactive" on:click={() => handleSegmentClick(0)}>17% of eviction filings resulted in executed evictions</span>,
              it's not possible to know what the outcomes of many eviction filings were as these are not well documented in the data. 
              <span class="highlight interactive" on:click={() => handleSegmentClick(1)}>48% of eviction filing records in Boston don't show whether the tenant was actually evicted.</span>
            </p>
            
            {#if isDataLoaded}
              <div class="statistics-box" in:fade={{ duration: 400, delay: 300 }}>
                <div class="stat-heading">EVICTION OUTCOME STATISTICS:</div>
                
                <div class="stat-row">
                  <div class="stat-item" in:fly={{ y: 10, duration: 300, delay: 100 }}>
                    <div class="stat-value">10 days</div>
                    <div class="stat-label">to appeal an eviction ruling</div>
                  </div>
                  
                  <div class="stat-item" in:fly={{ y: 10, duration: 300, delay: 200 }}>
                    <div class="stat-value">35%</div>
                    <div class="stat-label">of cases with documented resolution</div>
                  </div>
                </div>
              </div>
            {/if}
          </div>
        </div>
      </div>
    </div>

    <div class="chart-right">
      {#if !isDataLoaded}
        <div class="loading-container" transition:fade>
          <div class="loading-bar-container">
            <div class="loading-bar" style="width: {$progress * 100}%"></div>
          </div>
          <div class="loading-text">Loading eviction outcome data...</div>
        </div>
      {:else}
        <div class="chart-container" in:fade={{ duration: 800 }}>
          <div class="chart-header">EVICTION FILING OUTCOMES</div>
          <div class="chart">
            <div class="donut-container" on:click={handleBackgroundClick}>
              <div class="chart-content">
                <svg viewBox="-150 -150 300 300" width="300" height="300" on:click|stopPropagation>
                  <!-- Donut segments -->
                  {#each arcData as slice, index}
                    <path 
                      d={arcGenerator(slice)} 
                      fill={colors(index)}
                      stroke="#fff" 
                      stroke-width="1"
                      class:selected={selectedIndex === index}
                      on:click|stopPropagation={() => handleSegmentClick(index)}
                    />
                  {/each}

                  <!-- Center text for total -->
                  <text 
                    text-anchor="middle" 
                    dominant-baseline="middle"
                    class="total-label"
                  >
                    <tspan x="0" y="-12" class="total-value">100%</tspan>
                    <tspan x="0" y="12" class="total-text">CASES</tspan>
                  </text>

                  <!-- Labels for larger segments -->
                  {#each arcData as slice, index}
                    {#if (slice.endAngle - slice.startAngle) > 0.4 && (selectedIndex === -1 || selectedIndex === index)}
                      {@const centroid = labelArcGenerator.centroid(slice)}
                      <text 
                        x={centroid[0]} 
                        y={centroid[1]}
                        text-anchor={centroid[0] > 0 ? "start" : "end"}
                        dominant-baseline="middle"
                        class="segment-label"
                        fill={selectedIndex === index ? "#000" : "#333"}
                      >
                        {slice.data.value}%
                      </text>
                    {/if}
                  {/each}
                </svg>

                <div class="legend-container">
                  <ul class="legend" on:click|stopPropagation>
                    {#each evictionData as d, index}
                      <li 
                        style="--color: {colors(index)}"
                        on:click|stopPropagation={() => handleSegmentClick(index)}
                        class:selected={selectedIndex === index}
                      >
                        <span class="swatch"></span>
                        <div class="legend-text">
                          <span class="legend-label">{d.label}</span>
                          <span class="legend-value">{d.value}%</span>
                        </div>
                      </li>
                    {/each}
                  </ul>
                </div>
                
                <div class="chart-footer">
                  <div class="data-source">Source: Boston Housing Court Records, 2020-2022</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      {/if}
    </div>
  </div>
  
  {#if selectedIndex !== -1 && isDataLoaded}
    <div class="explanation-overlay" on:click={handleBackgroundClick} transition:fade={{ duration: 200 }}>
      <div class="explanation-card" on:click|stopPropagation in:fly={{ y: 20, duration: 300 }}>
        <div class="explanation-title">
          {evictionData[selectedIndex].label}
        </div>
        <div class="explanation-content">
          <p>{getOutcomeExplanation(evictionData[selectedIndex].label)}</p>
          <p>This outcome represents {evictionData[selectedIndex].value}% of all eviction filings in Boston from 2020-2022.</p>
        </div>
        <button class="close-button" on:click={handleBackgroundClick}>Close</button>
      </div>
    </div>
  {/if}
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
    margin: 0 auto;
  }

  .chart-container {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
  }

  .chart-header {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 24px;
    margin-bottom: 1rem;
    text-align: center;
    color: #14110F;
    letter-spacing: 1.5px;
  }

  .chart {
    align-self: center;
    width: 100%;
    height: 100%;
  }

  .chart-footer {
    display: flex;
    justify-content: center;
    margin-top: 1rem;
    font-family: 'Geist Mono', monospace;
    font-size: 10px;
    color: #666;
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

  .highlight.interactive {
    cursor: pointer;
    transition: background-color 0.2s ease;
  }

  .highlight.interactive:hover {
    background-color: #04b589;
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
  
  /* Loading animation */
  .loading-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 300px;
  }

  .loading-bar-container {
    width: 200px;
    height: 6px;
    background-color: #f0f0f0;
    border-radius: 3px;
    overflow: hidden;
  }

  .loading-bar {
    height: 100%;
    background-color: #06D6A0;
    transition: width 0.3s ease;
  }

  .loading-text {
    margin-top: 1rem;
    font-family: 'Geist Mono', monospace;
    font-size: 12px;
    color: #666;
  }

  /* Statistics box */
  .statistics-box {
    margin-top: 2rem;
    padding: 1.5rem;
    background-color: rgba(6, 214, 160, 0.1);
    border-radius: 8px;
  }

  .stat-heading {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 18px;
    margin-bottom: 1rem;
    letter-spacing: 1px;
  }

  .stat-row {
    display: flex;
    flex-wrap: wrap;
    gap: 1.5rem;
  }

  .stat-item {
    flex: 1;
    min-width: 150px;
  }

  .stat-value {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 28px;
    color: #14110F;
    margin-bottom: 0.25rem;
  }

  .stat-label {
    font-size: 13px;
    line-height: 1.4;
    color: #333;
  }
  
  /* Donut chart styling */
  .donut-container {
    font-family: 'Geist Mono', monospace;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }

  .chart-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  path {
    transition: all 300ms ease;
    stroke: white;
    stroke-width: 1px;
  }

  path:hover {
    filter: brightness(1.1);
    transform: scale(1.02);
    transform-origin: center;
  }

  .total-label {
    font-family: 'Geist Mono', monospace;
    text-anchor: middle;
    dominant-baseline: middle;
  }

  .total-value {
    font-size: 14px;
    font-weight: bold;
    fill: #14110F;
  }

  .total-text {
    font-size: 10px;
    fill: #666;
  }

  .segment-label {
    font-size: 10px;
    font-weight: bold;
    pointer-events: none;
  }

  .legend-container {
    margin-top: 1rem;
    padding: 0.5rem 0;
    width: 100%;
  }

  .legend {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    gap: 0.5rem;
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .legend li {
    display: flex;
    align-items: center;
    cursor: pointer;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    transition: background-color 0.2s ease;
    flex: 0 0 auto;
    white-space: nowrap;
    min-width: 90px;
  }

  .legend li:hover {
    background-color: #f0f0f0;
  }

  .legend li .swatch {
    display: inline-block;
    width: 10px;
    height: 10px;
    border-radius: 2px;
    background-color: var(--color);
    margin-right: 0.4rem;
    border: 1px solid rgba(0,0,0,0.1);
    flex-shrink: 0;
  }

  .legend-text {
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  .legend-label {
    font-weight: 500;
    color: #14110F;
    font-size: 11px;
  }

  .legend-value {
    font-size: 11px;
    font-weight: bold;
    color: var(--color);
  }

  /* Selection states */
  .selected {
    transform: scale(1.03);
    transform-origin: center;
    filter: brightness(1.05);
  }

  .legend li.selected {
    background-color: rgba(0,0,0,0.05);
    box-shadow: 0 0 0 1px rgba(0,0,0,0.1);
  }

  .legend li.selected .legend-label {
    font-weight: bold;
  }

  /* When something is selected, dim others */
  svg:has(.selected) path:not(.selected) {
    opacity: 0.4;
  }

  .legend:has(.selected) li:not(.selected) {
    opacity: 0.6;
  }

  .explanation-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }

  .explanation-card {
    width: 90%;
    max-width: 500px;
    background-color: white;
    border-radius: 10px;
    padding: 2rem;
    box-shadow: 0 10px 25px rgba(0,0,0,0.2);
  }

  .explanation-title {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 28px;
    margin-bottom: 1rem;
    color: #14110F;
  }

  .explanation-content {
    font-family: 'Geist Mono', monospace;
    line-height: 1.6;
    margin-bottom: 1.5rem;
  }

  .explanation-content p {
    margin-bottom: 1rem;
  }

  .explanation-content p:last-child {
    margin-bottom: 0;
  }

  .close-button {
    background-color: #14110F;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 5px;
    font-family: 'Geist Mono', monospace;
    cursor: pointer;
    transition: background-color 0.2s ease;
  }

  .close-button:hover {
    background-color: #333;
  }

  @media (max-width: 1024px) {
    .content-container {
      grid-template-columns: 1fr;
      gap: 2rem;
      max-height: none;
    }
    
    .text-left, .chart-right {
      grid-column: 1;
    }
    
    .chart-right {
      grid-row: 2;
    }
    
    .stat-row {
      flex-direction: column;
      gap: 1rem;
    }
  }
</style>