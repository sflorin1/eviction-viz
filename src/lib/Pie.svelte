<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    
    export let data = [];
    export let title = "DISTRIBUTION BY TYPE";
    export let width = 400;
    export let height = 400;
    export let selectedIndex = -1;
    export let hoverIndex = -1;
    
    // Create a donut chart by setting innerRadius > 0
    const radius = Math.min(width, height) / 2.5;
    const innerRadius = radius * 0.6; // This creates the donut hole
    let arcGenerator = d3.arc()
      .innerRadius(innerRadius)
      .outerRadius(radius)
      .padAngle(0.02)
      .cornerRadius(4); // Rounded corners for more modern look
    
    // Arc for labels (positioned further out)
    let labelArcGenerator = d3.arc()
      .innerRadius(radius * 1.1)
      .outerRadius(radius * 1.1);
    
    // Use a pleasing color scheme that matches our stacked bar
    let colors = d3.scaleOrdinal()
      .range(['#06D6A0', '#1B9AAA', '#EF476F', '#FFC43D', '#E56B6F', '#118AB2']);
    
    let sliceGenerator = d3.pie()
      .value(d => d.value)
      .sort(null); // Don't sort so order is maintained
    
    let arcData;
    let arcs;
    let labelArcs;
    let totalValue = 0;
    
    $: {
      arcData = sliceGenerator(data);
      arcs = arcData.map(d => arcGenerator(d));
      labelArcs = arcData.map(d => labelArcGenerator(d));
      totalValue = d3.sum(data, d => d.value);
    }
    
    function setHoverIndex(index) {
      hoverIndex = index;
    }
    
    function clearHoverIndex() {
      hoverIndex = -1;
    }
    
    // Calculate percentage for each segment
    function getPercent(value) {
      return totalValue > 0 ? Math.round((value / totalValue) * 100) : 0;
    }
    
    // Format a large number with commas
    function formatNumber(num) {
      return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }
    
    // Format label with line breaks for better fit
    function formatLabel(label) {
      if (label.length > 10) {
        const words = label.split(' ');
        if (words.length > 1) {
          return words.join('\n');
        }
      }
      return label;
    }
    
    </script>
    
    <div class="donut-container" on:click={() => selectedIndex = -1}>
      <div class="chart-title">{title}</div>
      
      <div class="chart-content">
        <svg viewBox="{-width/2} {-height/2} {width} {height}" width={width} height={height} on:click|stopPropagation>
          <!-- Donut segments -->
          {#each arcData as slice, index}
            <path 
              d={arcGenerator(slice)} 
              fill={colors(index)}
              stroke="#fff" 
              stroke-width="1"
              class:selected={selectedIndex === index}
              class:hovered={hoverIndex === index}
              on:click|stopPropagation={() => selectedIndex = selectedIndex === index ? -1 : index}
              on:mouseenter={() => setHoverIndex(index)}
              on:mouseleave={clearHoverIndex}
            />
          {/each}
    
          <!-- Center text for total -->
          <text 
            text-anchor="middle" 
            dominant-baseline="middle"
            class="total-label"
          >
            <tspan x="0" y="-12" class="total-value">{formatNumber(totalValue)}</tspan>
            <tspan x="0" y="12" class="total-text">TOTAL</tspan>
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
    
        <!-- Value inside segments for larger ones -->
        <div class="legend-container">
          <ul class="legend" on:click|stopPropagation>
            {#each data as d, index}
              <li 
                style="--color: {colors(index)}"
                on:click|stopPropagation={() => selectedIndex = selectedIndex === index ? -1 : index}
                on:mouseenter={() => setHoverIndex(index)}
                on:mouseleave={clearHoverIndex}
                class:selected={selectedIndex === index}
                class:hovered={hoverIndex === index}
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
      </div>
    </div>
    
    <style>
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
        flex-wrap: nowrap;
        gap: 1rem;
        list-style: none;
        padding: 0;
        margin: 0;
        overflow-x: auto;
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
    
      .hovered:not(.selected) {
        filter: brightness(1.05);
      }
    
      .legend li.selected {
        background-color: rgba(0,0,0,0.05);
        box-shadow: 0 0 0 1px rgba(0,0,0,0.1);
      }
    
      .legend li.selected .legend-label,
      .legend li.hovered .legend-label {
        font-weight: bold;
      }
    
      /* When something is selected, dim others */
      svg:has(.selected) path:not(.selected) {
        opacity: 0.4;
      }
    
      .legend:has(.selected) li:not(.selected) {
        opacity: 0.6;
      }
    
      /* Force full opacity on hover even when others are selected */
      path.hovered {
        opacity: 1 !important;
      }
    </style>