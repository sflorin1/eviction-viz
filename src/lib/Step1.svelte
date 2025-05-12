<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  import { onMount } from 'svelte';
  import * as d3 from "d3";
  import StackedBar from '$lib/StackedBar.svelte';
  import { base } from '$app/paths';
  import { fly, fade } from 'svelte/transition';
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  
  let processedData = [];
  let isDataLoaded = false;
  let activeEvictionType = null;
  let selectedIndex = -1;
  
  // Animation for data loading
  const progress = tweened(0, {
    duration: 800,
    easing: cubicOut
  });
  
  onMount(async () => {
    // Start loading animation
    progress.set(0.3);
    
    try {
      const raw = await d3.csv(`${base}/evictions_by_month.csv`, d3.autoType);
      progress.set(0.7);
      
      processedData = raw.map(d => {
        const { month, ...rest } = d;
        return {
          month,
          ...rest
        };
      });
      
      progress.set(1);
      setTimeout(() => {
        isDataLoaded = true;
      }, 300);
    } catch (error) {
      console.error("Error loading data:", error);
    }
  });
  
  function showEvictionInfo(type) {
    activeEvictionType = type;
  }
  
  function hideEvictionInfo() {
    activeEvictionType = null;
  }
  
  // Calculate total evictions by type
  $: evictionTotals = isDataLoaded ? 
    Object.keys(processedData[0] || {})
      .filter(key => key !== 'month')
      .map(type => ({
        type,
        total: d3.sum(processedData, d => d[type] || 0)
      }))
      .sort((a, b) => b.total - a.total) : [];
  </script>
  
  <body>
    <div class="content-container">
      <div class="text-left">
        <div class="text-left-inner">
          <div class="header-section">
            <div class="number">#1</div>
            <div in:fly={{ y: 20, duration: 400 }}><span class="title-highlight">NOTICE TO QUIT</span></div>
          </div>
          
          <div class="text-section">
            <div class="body-text" in:fade={{ duration: 400, delay: 100 }}>
              <p>
                <span class="highlight">You've received a Notice to Quit from your landlord.</span> This tells you the reason they're seeking
                to evict you and how long you have to respond.
              </p>
              <p>
                Not all eviction filings that happen in Boston are for failure to pay rent, in fact, many filings are what's called
                <span class="highlight interactive" on:click={() => showEvictionInfo("No Fault")}>"No Fault"</span> evictions.
              </p>
              <p>
                <span class="highlight interactive" on:click={() => showEvictionInfo("No Fault")}>"No Fault"</span> means the landlord is asking the tenant to move out without claiming that the lease
                was violated. This could be because the owner is selling the property, they want to renovate it, or something else.
                <span class="highlight interactive" on:click={() => showEvictionInfo("Cause")}>"Cause"</span> means the landlord claims there was a violation of the lease.
              </p>
              
              {#if isDataLoaded && evictionTotals.length > 0}
                <div class="eviction-stats" in:fade={{ duration: 400, delay: 300 }}>
                  <div class="stat-heading">MOST COMMON EVICTION TYPES:</div>
                  {#each evictionTotals.slice(0, 3) as { type, total }, i}
                    <div class="stat-item" style="--delay: {i * 100}ms" in:fly={{ y: 10, duration: 300, delay: i * 100 }}>
                      <div class="stat-rank">{i + 1}</div>
                      <div class="stat-type">{type}</div>
                      <div class="stat-total">{total} filings</div>
                    </div>
                  {/each}
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
            <div class="loading-text">Loading eviction data...</div>
          </div>
        {:else}
          <div class="chart-container" in:fade={{ duration: 800 }}>
            <div class="chart-header">EVICTION NOTICES BY TYPE</div>
            <div class="chart">
              <StackedBar data={processedData} height={450} width={550} margin={{ top: 30, right: 30, bottom: 60, left: 50 }} bind:selectedIndex={selectedIndex}/>
            </div>
            <div class="chart-footer">
              <div class="data-source">Source: Boston Housing Court Records</div>
              <div class="interaction-hint">Click on legend items to highlight specific types</div>
            </div>
          </div>
        {/if}
      </div>
    </div>
    
    {#if activeEvictionType}
      <div class="explanation-overlay" on:click={hideEvictionInfo} transition:fade={{ duration: 200 }}>
        <div class="explanation-card" on:click|stopPropagation in:fly={{ y: 20, duration: 300 }}>
          <div class="explanation-title">{activeEvictionType} Evictions</div>
          <div class="explanation-content">
            {#if activeEvictionType === "No Fault"}
              These evictions occur when the landlord asks the tenant to move out without claiming that the lease was violated. Common reasons include the owner selling the property, planning renovations, or wanting to use the unit for personal purposes.
            {:else if activeEvictionType === "Cause"}
              These evictions occur when the landlord claims there was a violation of the lease terms. This could include non-payment of rent, property damage, illegal activity, or other lease violations.
            {:else if activeEvictionType === "Non-Payment"}
              These evictions occur specifically when tenants fall behind on rent payments. This is one of the most common reasons for eviction filings in Boston.
            {:else}
              Information about {activeEvictionType} eviction notices.
            {/if}
          </div>
          <button class="close-button" on:click={hideEvictionInfo}>Close</button>
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
    grid-template-rows: subgrid;
    grid-column: 2;
    grid-row: 1 / span 2;
    width: 100%;
    height: 100%;
    min-width: 0;
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
    justify-content: space-between;
    margin-top: 1rem;
    font-family: 'Geist Mono', monospace;
    font-size: 12px;
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
  
  .eviction-stats {
    margin-top: 2rem;
    border-left: 4px solid #06D6A0;
    padding-left: 1rem;
  }
  
  .stat-heading {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 18px;
    margin-bottom: 0.75rem;
    letter-spacing: 1px;
  }
  
  .stat-item {
    display: flex;
    align-items: center;
    margin-bottom: 0.5rem;
    animation: fadeIn 0.5s ease forwards;
    animation-delay: var(--delay);
    opacity: 0;
  }
  
  .stat-rank {
    width: 24px;
    height: 24px;
    background-color: #14110F;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 14px;
    margin-right: 0.75rem;
  }
  
  .stat-type {
    font-weight: 600;
    margin-right: 0.75rem;
  }
  
  .stat-total {
    color: #666;
    font-size: 14px;
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
  
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
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
  }
  </style>