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
  
  let pieData = [];
  let isDataLoaded = false;
  let selectedIndex = -1;
  let totalAttorneys = 0;
  
  // Animation for data loading
  const progress = tweened(0, {
    duration: 800,
    easing: cubicOut
  });
  
  onMount(async () => {
    // Start loading animation
    progress.set(0.3);
    
    try {
      const rawData = await d3.csv(`${base}/alternate_landlord_attorneys.csv`, d => ({
        label: d.name,
        value: +d.count
      }));
      
      progress.set(0.7);
      
      // Sort data by value in descending order
      pieData = rawData.sort((a, b) => b.value - a.value);
      
      // Calculate total eviction cases
      totalAttorneys = d3.sum(pieData, d => d.value);
      
      progress.set(1);
      setTimeout(() => {
        isDataLoaded = true;
      }, 300);
    } catch (error) {
      console.error("Error loading data:", error);
    }
  });
  
  // Creating a donut chart 
  // Use vibrant colors aligned with the highlight colors from the text
  const colors = d3.scaleOrdinal()
    .range(['#06D6A0', '#1B9AAA', '#EF476F', '#FFC43D', '#E56B6F', '#118AB2']);
  
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
  
  $: arcData = sliceGenerator(pieData);
  
  // Format number with commas
  function formatNumber(num) {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
  }
  
  // Calculate percentage
  function getPercent(value) {
    return totalAttorneys > 0 ? Math.round((value / totalAttorneys) * 100) : 0;
  }
  
  function handleSegmentClick(index) {
    selectedIndex = selectedIndex === index ? -1 : index;
  }
  
  function handleBackgroundClick() {
    selectedIndex = -1;
  }
</script>

<body>
  <div class="content-container">
    <div class="text-left">
      <div class="text-left-inner">
        <div class="header-section">
          <div class="number">#3</div>
          <div in:fly={{ y: 20, duration: 400 }}><span class="title-highlight">LEGAL DEFENSE</span></div>
        </div>
        
        <div class="text-section">
          <div class="body-text" in:fade={{ duration: 400, delay: 100 }}>
            <p>
              Lets imagine you're one of the <span class="highlight">90% of tenants in Boston who do not have legal representation</span>
              for their eviction case. This means that when your court date comes, you will defend yourself against eviction.
            </p>
            <p>
              At the same time, <span class="highlight">86% of landlords in Boston Housing Court cases use legal representation.</span> 
              In fact, your landlord is likely to use an attorney who works on <i>many</i> eviction cases for landlords in Boston—  
              <i>an eviction expert.</i>
            </p>
            <p>
              Our analysis found that a single Boston-Area attorney represented landlords in <span class="highlight">688 eviction filings in the two year span</span>
              between 2020–2022.
            </p>
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
          <div class="loading-text">Loading attorney data...</div>
        </div>
      {:else}
        <div class="chart" in:fade={{ duration: 800 }}>
          <div class="donut-container" on:click={handleBackgroundClick}>
            <div class="chart-title">TOP LANDLORD ATTORNEYS</div>
            
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
                  <tspan x="0" y="-12" class="total-value">{formatNumber(totalAttorneys)}</tspan>
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
                      {getPercent(slice.data.value)}%
                    </text>
                  {/if}
                {/each}
              </svg>

              <div class="legend-container">
                <ul class="legend" on:click|stopPropagation>
                  {#each pieData.slice(0, 14) as d, index}
                    <li 
                      style="--color: {colors(index)}"
                      on:click|stopPropagation={() => handleSegmentClick(index)}
                      class:selected={selectedIndex === index}
                    >
                      <span class="swatch"></span>
                      <div class="legend-text">
                        <span class="legend-label">{d.label}</span>
                        <span class="legend-value">{formatNumber(d.value)}</span>
                        <span class="legend-percent">({getPercent(d.value)}%)</span>
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
      {/if}
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
  
  /* Donut chart styling */
  .donut-container {
    font-family: 'Geist Mono', monospace;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }

  .chart-title {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 18px;
    text-align: center;
    margin-bottom: 1rem;
    color: #14110F;
    letter-spacing: 1px;
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

  .legend-percent {
    font-size: 9px;
    color: #666;
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
  
  .chart-footer {
    margin-top: 1rem;
    font-family: 'Geist Mono', monospace;
    font-size: 10px;
    color: #666;
    text-align: center;
  }
</style>