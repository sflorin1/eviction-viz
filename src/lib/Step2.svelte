<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  import { onMount } from 'svelte';
  import StackedBar from '$lib/StackedBar.svelte';
  import * as d3 from "d3";
  import { base } from '$app/paths';
  import { fly, fade } from 'svelte/transition';
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  
  let processedData = [];
  let isDataLoaded = false;
  let activeExplanation = null;
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
  
  function showExplanation(type) {
    activeExplanation = type;
  }
  
  function hideExplanation() {
    activeExplanation = null;
  }
  
  // Calculate total evictions by type for statistics
  $: evictionTotals = isDataLoaded ? 
    Object.keys(processedData[0] || {})
      .filter(key => key !== 'month')
      .map(type => ({
        type,
        total: d3.sum(processedData, d => d[type] || 0)
      }))
      .sort((a, b) => b.total - a.total) : [];
  
  // Calculate the percentage of non-payment evictions
  $: nonPaymentPercent = isDataLoaded && evictionTotals.length > 0 ? 
    Math.round((evictionTotals.find(t => t.type === "Non-Payment")?.total || 0) / 
    d3.sum(evictionTotals, d => d.total) * 100) : 0;
  
  // Calculate year-to-year growth for contextual statistics
  $: totalEvictions = isDataLoaded ? d3.sum(evictionTotals, d => d.total) : 0;
  </script>
  
  <body>
    <div class="content-container">
      <div class="text-left">
        <div class="text-left-inner">
          <div class="header-section">
            <div class="number">#2</div>
            <div in:fly={{ y: 20, duration: 400 }}><span class="title-highlight">FILING IN HOUSING COURT</span></div>
          </div>
          
          <div class="text-section">
            <div class="body-text" in:fade={{ duration: 400, delay: 100 }}>
              <p>
                It's no wonder that <span class="highlight interactive" on:click={() => showExplanation("nonPayment")}>"Non-Payment"</span> is the most common cause
                cited in filings. A 2024 analysis found that rapidly rising rents mean that the average Boston
                renter spends <span class="highlight">47% of their income on housing costs,</span> yet wages in
                Boston have not increased at the same rate.
              </p>
              <p>
                It's important to keep in mind that even when an eviction filing claims a tenant is being asked
                to leave for non-payment or a lease violation, <span class="highlight interactive" on:click={() => showExplanation("court")}> a landlord doesn't need to prove their claim until
                the case is heard in court.</span>
              </p>
              
              {#if isDataLoaded && evictionTotals.length > 0}
                <div class="statistics-box" in:fade={{ duration: 400, delay: 300 }}>
                  <div class="stat-heading">EVICTION FILING STATISTICS:</div>
                  
                  <div class="stat-row">
                    <div class="stat-item" in:fly={{ y: 10, duration: 300, delay: 100 }}>
                      <div class="stat-value">{nonPaymentPercent}%</div>
                      <div class="stat-label">of eviction filings cite "Non-Payment" as the cause</div>
                    </div>
                    
                    <div class="stat-item" in:fly={{ y: 10, duration: 300, delay: 200 }}>
                      <div class="stat-value">47%</div>
                      <div class="stat-label">of income spent on housing by average Boston renters</div>
                    </div>
                    
                    <div class="stat-item" in:fly={{ y: 10, duration: 300, delay: 300 }}>
                      <div class="stat-value">{totalEvictions.toLocaleString()}</div>
                      <div class="stat-label">total eviction filings in the dataset</div>
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
            <div class="loading-text">Loading eviction data...</div>
          </div>
        {:else}
          <div class="chart-container" in:fade={{ duration: 800 }}>
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
    
    {#if activeExplanation}
      <div class="explanation-overlay" on:click={hideExplanation} transition:fade={{ duration: 200 }}>
        <div class="explanation-card" on:click|stopPropagation in:fly={{ y: 20, duration: 300 }}>
          <div class="explanation-title">
            {#if activeExplanation === "nonPayment"}
              Non-Payment Evictions
            {:else if activeExplanation === "court"}
              The Court Process
            {:else}
              {activeExplanation}
            {/if}
          </div>
          <div class="explanation-content">
            {#if activeExplanation === "nonPayment"}
              <p>Non-payment evictions occur when a landlord claims that a tenant has failed to pay rent according to the lease agreement. This is the most common type of eviction filing in Boston.</p>
              <p>However, non-payment claims can sometimes mask other motivations. With rapidly rising property values in Boston, some landlords may use minor payment issues as a reason to remove tenants from units they wish to renovate or rent at higher rates.</p>
            {:else if activeExplanation === "court"}
              <p>When a landlord files an eviction case, they make claims about why they want to evict the tenant. However, these claims aren't automatically verified when filed.</p>
              <p>The burden of proof comes later during the actual court hearing. This is why it's crucial for tenants to appear in court and contest any inaccurate claims, as failure to appear may result in a default judgment in favor of the landlord.</p>
              <p>Having legal representation greatly increases a tenant's chances of successfully contesting an eviction case.</p>
            {/if}
          </div>
          <button class="close-button" on:click={hideExplanation}>Close</button>
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
    align-items: center;
    justify-content: center;
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