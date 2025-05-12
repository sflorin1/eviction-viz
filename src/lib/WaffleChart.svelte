<script>
  import { createEventDispatcher } from 'svelte';
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  export let data = [];
  export let selectedIndex = -1;
  export let rows = 15; 
  export let columns = 40;
  export let cellSize = 20;
  export let cellPadding = 2;
  export let cellBorderRadius = 0;

  // Responsive sizing
  export let responsive = true; // New parameter to enable responsive sizing
  let containerWidth;
  let containerHeight;
  let containerElement;
  
  const dispatch = createEventDispatcher();

  let totalItems = 0;
  let processedData = [];
  let adjustedCellSize = cellSize;
  let width;
  let height;

  // Categorize data into groups
  let categorizedData = {
    high: [], // More than 100 eviction filings
    medium: [], // More than 50 evictions filings
    grey: [], // More than 30 evictions but less than 50 (to be shown in grey)
    other: [] // Less than 30 evictions or "Other" category
  };

  // Calculate percentage for top evictors
  let topEvictorsPercentage = 0;

  // Create color scales for each category - intense colors for highest values
  const blueScale = d3.scaleLinear()
    .domain([1, 0])
    .range(["#0047AB", "#99CCFF"]) 
    .interpolate(d3.interpolateHcl);

  const orangeScale = d3.scaleLinear()
    .domain([1, 0])
    .range(["#FF8C00", "#FFD700"]) 
    .interpolate(d3.interpolateHcl);

  // Get color based on item category and value
  function getColor(item, index) {
    if (item.isOther) {
      return "#FFFFFF";
    } else if (item.value > 100) {
      const highItems = categorizedData.high;
      const sortedHighItems = [...highItems].sort((a, b) => b.value - a.value);
      const itemPosition = sortedHighItems.findIndex(d => d.label === item.label);
      const normalizedPosition = 1 - (itemPosition / Math.max(1, highItems.length - 1));
      return blueScale(normalizedPosition);
    } else if (item.value > 50) {
      const mediumItems = categorizedData.medium;
      const sortedMediumItems = [...mediumItems].sort((a, b) => b.value - a.value);
      const itemPosition = sortedMediumItems.findIndex(d => d.label === item.label);
      const normalizedPosition = 1 - (itemPosition / Math.max(1, mediumItems.length - 1));
      return orangeScale(normalizedPosition);
    } else if (item.value > 30) {
      return "#FFFFFF";
    } else {
      return "#FFFFFF";
    }
  }

  // Handle legend item click
  function handleLegendClick(index, event) {
    event.stopPropagation();
    selectedIndex = selectedIndex === index ? -1 : index;
    dispatch('select', { index: selectedIndex });
  }

  // Process data when it changes
  $: {
    totalItems = d3.sum(data, d => d.value);
    
    // Categorize the data
    categorizedData = {
      high: data.filter(d => d.value > 100 && !d.isOther),
      medium: data.filter(d => d.value > 50 && d.value <= 100 && !d.isOther),
      grey: data.filter(d => d.value > 30 && d.value <= 50 && !d.isOther),
      other: data.filter(d => d.value <= 30 || d.isOther)
    };
    
    // Calculate the percentage of evictions from top evictors
    const topEvictorsTotal = d3.sum([...categorizedData.high, ...categorizedData.medium], d => d.value);
    topEvictorsPercentage = Math.round((topEvictorsTotal / totalItems) * 100);
    
    // Calculate how many cells each item needs
    const totalCells = rows * columns;
    processedData = data.map((d, i) => {
      const cellCount = Math.round((d.value / totalItems) * totalCells);
      return {
        ...d,
        cellCount,
        index: i
      };
    });
  }
  
  // Calculate cells array reactively
  $: cells = generateCells(processedData, rows, columns);
  
  // Function to generate cells - extracted for better readability
  function generateCells(processedData, rows, columns) {
    let cells = [];
    let dataItems = [...processedData];
    let totalCellsNeeded = dataItems.reduce((sum, item) => sum + item.cellCount, 0);
    let cellCount = 0;
    
    // Handle case where we need more or fewer cells than grid size
    const totalGridCells = rows * columns;
    if (totalCellsNeeded > totalGridCells) {
      // Scale down cell counts proportionally if we have too many
      const scaleFactor = totalGridCells / totalCellsNeeded;
      dataItems = dataItems.map(item => ({
        ...item,
        cellCount: Math.max(1, Math.floor(item.cellCount * scaleFactor))
      }));
      totalCellsNeeded = dataItems.reduce((sum, item) => sum + item.cellCount, 0);
    }
    
    // If we still don't have enough cells to fill the grid, adjust the largest item
    if (totalCellsNeeded < totalGridCells) {
      const largestItem = dataItems.reduce(
        (max, item) => (item.cellCount > max.cellCount ? item : max),
        { cellCount: 0 }
      );
      const indexOfLargest = dataItems.findIndex(item => item === largestItem);
      if (indexOfLargest >= 0) {
        dataItems[indexOfLargest].cellCount += (totalGridCells - totalCellsNeeded);
      }
    }
    
    // Generate the cells from left to right, top to bottom
    for (let i = 0; i < rows; i++) {
      for (let j = 0; j < columns; j++) {
        if (cellCount < totalGridCells) {
          // Find data item this cell belongs to
          let itemIndex = 0;
          while (itemIndex < dataItems.length && dataItems[itemIndex].cellCount <= 0) {
            itemIndex++;
          }
          
          if (itemIndex < dataItems.length) {
            cells.push({
              x: j * adjustedCellSize,
              y: i * adjustedCellSize,
              dataItem: dataItems[itemIndex],
              index: dataItems[itemIndex].index
            });
            
            dataItems[itemIndex].cellCount--;
            cellCount++;
          }
        }
      }
    }
    
    return cells;
  }
  
  // Calculate width and height based on container size if responsive is true
  onMount(() => {
    function updateSize() {
      if (responsive && containerElement) {
        containerWidth = containerElement.clientWidth;
        
        // Calculate cell size based on available width
        const maxCellSize = Math.floor((containerWidth - 30) / columns); // 30px for padding
        adjustedCellSize = Math.min(cellSize, maxCellSize);
        
        width = columns * adjustedCellSize;
        height = rows * adjustedCellSize;
      } else {
        adjustedCellSize = cellSize;
        width = columns * cellSize;
        height = rows * cellSize;
      }
    }
    
    updateSize();
    
    // Add resize listener for responsiveness
    const resizeObserver = new ResizeObserver(updateSize);
    if (containerElement) {
      resizeObserver.observe(containerElement);
    }
    
    return () => {
      if (containerElement) {
        resizeObserver.unobserve(containerElement);
      }
    };
  });
  
  // Update size when container element changes
  $: if (containerElement) {
    containerWidth = containerElement.clientWidth;
    if (responsive) {
      const maxCellSize = Math.floor((containerWidth - 30) / columns); // 30px for padding
      adjustedCellSize = Math.min(cellSize, maxCellSize);
    } else {
      adjustedCellSize = cellSize;
    }
    width = columns * adjustedCellSize;
    height = rows * adjustedCellSize;
  }
</script>

<div class="waffle-container" bind:this={containerElement}>
  <!-- Total eviction notices -->
  <div class="total-evictions">
    <h3>Total Eviction Notices: {totalItems}</h3>
  </div>
  
  <div class="waffle-chart-wrapper">
    <div class="waffle-chart" on:click|stopPropagation>
      <svg width={width} height={height}>
        {#each cells as cell}
          <rect
            width={adjustedCellSize - cellPadding}
            height={adjustedCellSize - cellPadding}
            x={cell.x + cellPadding / 2}
            y={cell.y + cellPadding / 2}
            rx={cellBorderRadius}
            ry={cellBorderRadius}
            fill={getColor(cell.dataItem, cell.index)}
            stroke={cell.dataItem?.isOther ? "#dddddd" : "none"}
            stroke-width={cell.dataItem?.isOther ? 1 : 0}
            opacity={selectedIndex === -1 || selectedIndex === cell.index ? 1 : 0.3}
            class={selectedIndex === cell.index ? 'selected' : ''}
          />
        {/each}
      </svg>
    </div>
  </div>
  
  <div class="legend-wrapper">
    <div class="legend-columns">
      <!-- High evictions column -->
      <div class="legend-column">
        <h3>More than 100 evictions</h3>
        <div class="legend-items">
          {#each [...categorizedData.high].sort((a, b) => b.value - a.value) as item, i}
            <div 
              class="legend-item" 
              class:selected={selectedIndex === data.findIndex(d => d.label === item.label)}
              on:click={(e) => handleLegendClick(data.findIndex(d => d.label === item.label), e)}
            >
              <div class="legend-item-container">
                <div class="legend-color" style="background-color: {getColor(item)};"></div>
                <div class="legend-label" title={item.label}>{item.label}</div>
              </div>
              <div class="legend-value">{item.value} evictions</div>
            </div>
          {/each}
        </div>
      </div>
      
      <!-- Medium evictions column -->
      <div class="legend-column">
        <h3>More than 50 evictions</h3>
        <div class="legend-items">
          {#each [...categorizedData.medium].sort((a, b) => b.value - a.value) as item, i}
            <div 
              class="legend-item" 
              class:selected={selectedIndex === data.findIndex(d => d.label === item.label)}
              on:click={(e) => handleLegendClick(data.findIndex(d => d.label === item.label), e)}
            >
              <div class="legend-item-container">
                <div class="legend-color" style="background-color: {getColor(item)};"></div>
                <div class="legend-label" title={item.label}>{item.label}</div>
              </div>
              <div class="legend-value">{item.value} evictions</div>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>
  
  <!-- Percentage information -->
  <div class="evictors-percentage">
    <p>The {categorizedData.high.length + categorizedData.medium.length} evictors shown are responsible for <strong>{topEvictorsPercentage}%</strong> of eviction notices</p>
  </div>
</div>

<style>
  .waffle-container {
    display: flex;
    flex-direction: column;
    width: 100%;
    margin: 0 auto;
    gap: 20px;
    align-items: center;
    height: 100%;
  }
  
  .waffle-chart-wrapper {
    width: 100%;
    display: flex;
    justify-content: center;
    overflow: auto;
    padding: 5px;
    box-sizing: border-box;
  }
  
  .waffle-chart {
    width: fit-content;
    display: flex;
    justify-content: center;
    background-color: rgba(255, 255, 255, 0.3);
    border-radius: 8px;
    padding: 15px;
    box-sizing: border-box;
  }
  
  svg {
    display: block;
    max-width: 100%;
  }
  
  rect {
    transition: opacity 0.2s ease;
  }
  
  rect.selected {
    stroke: #000;
    stroke-width: 2px;
    opacity: 1;
  }
  
  .legend-wrapper {
    width: 100%;
    overflow-y: auto;
    max-height: 300px; /* Reduced from 450px */
    background-color: rgba(255, 255, 255, 0.3);
    border-radius: 8px;
    padding: 15px;
    box-sizing: border-box;
  }
  
  .legend-columns {
    display: flex;
    width: 100%;
    gap: 15px;
  }
  
  .legend-column {
    flex: 1;
    min-width: 0;
    margin-bottom: 15px;
  }
  
  .legend-column h3 {
    font-family: 'Bebas Neue', sans-serif;
    margin: 0 0 12px 0;
    padding-bottom: 8px;
    border-bottom: 1px solid #ddd;
    letter-spacing: 1px;
    font-size: 1rem;
    white-space: nowrap;
    text-align: center;
  }
  
  .legend-items {
    display: flex;
    flex-direction: column;
    gap: 8px;
    overflow-y: auto;
    max-height: 200px; /* Added max-height to enable scrolling */
  }
  
  .legend-item {
    display: flex;
    flex-direction: column;
    padding: 8px 10px;
    border-radius: 4px;
    background-color: rgba(255, 255, 255, 0.7);
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    margin-bottom: 5px;
  }
  
  .legend-item-container {
    display: flex;
    align-items: center;
    width: 100%;
  }
  
  .legend-item:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  }
  
  .legend-item.selected {
    background-color: rgba(255, 255, 255, 0.9);
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    border: 1px solid #999;
    transform: translateY(-2px);
  }
  
  .legend-color {
    width: 14px;
    height: 14px;
    border-radius: 3px;
    margin-right: 8px;
    flex-shrink: 0;
  }
  
  .legend-label {
    font-family: 'Inconsolata', monospace;
    font-size: 0.85em;
    line-height: 1.2;
    margin-right: 5px;
    flex: 1;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  
  .legend-value {
    font-family: 'Inconsolata', monospace;
    font-weight: bold;
    font-size: 0.8em;
    color: #555;
    margin-top: 4px;
    margin-left: 22px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  
  .total-evictions {
    background-color: rgba(245, 245, 245, 0.8);
    padding: 12px 20px;
    border-radius: 6px;
    text-align: center;
    margin-bottom: 15px;
    width: 100%;
    box-sizing: border-box;
  }
  
  .total-evictions h3 {
    font-family: 'Bebas Neue', sans-serif;
    margin: 0;
    font-size: 1.3rem;
    letter-spacing: 1px;
    color: #333;
  }
  
  .evictors-percentage {
    background-color: rgba(245, 245, 245, 0.8);
    padding: 12px 20px;
    border-radius: 6px;
    text-align: center;
    margin-top: 15px;
    width: 100%;
    box-sizing: border-box;
  }
  
  .evictors-percentage p {
    font-family: 'Inconsolata', monospace;
    margin: 0;
    font-size: 1.1rem;
    color: #333;
  }
  
  .evictors-percentage strong {
    color: #0047AB;
    font-size: 1.2rem;
  }
  
  /* Improved responsive design for different screen sizes */
  @media (max-width: 1200px) {
    .legend-columns {
      flex-direction: column;
    }
    
    .legend-column {
      width: 100%;
      margin-bottom: 20px;
    }
    
    .legend-wrapper {
      max-height: 400px; /* Allow more height on smaller screens */
    }
    
    .legend-items {
      max-height: 150px;
    }
  }
  
  @media (max-width: 768px) {
    .waffle-chart {
      padding: 10px;
    }
    
    .legend-column h3 {
      font-size: 0.9rem;
    }
    
    .legend-item {
      padding: 6px 8px;
    }
    
    .legend-label {
      font-size: 0.8em;
    }
    
    .legend-value {
      font-size: 0.75em;
    }
    
    .total-evictions h3,
    .evictors-percentage p {
      font-size: 0.9rem;
    }
    
    .evictors-percentage strong {
      font-size: 1rem;
    }
  }
</style>