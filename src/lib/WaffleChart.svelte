<script>
  import { createEventDispatcher } from 'svelte';
  import * as d3 from 'd3';

  export let data = [];
  export let selectedIndex = -1;
  export let rows = 10;
  export let columns = 30; // Default to wider chart 
  export let cellSize = 20;
  export let cellPadding = 2;
  export let cellBorderRadius = 0; // Changed to 0 for completely square corners

  const dispatch = createEventDispatcher();

  let totalItems = 0;
  let processedData = [];
  let colorScale;
  let width = columns * cellSize;
  let height = rows * cellSize;

  // Categorize data into three groups
  let categorizedData = {
    high: [], // More than 70 evictions
    medium: [], // More than 30 evictions
    low: [] // Less than 30 evictions
  };

  $: {
    totalItems = d3.sum(data, d => d.value);
    
    // Categorize the data (exclude "Other" from categorization for display)
    categorizedData = {
      high: data.filter(d => d.value > 70 && !d.isOther),
      medium: data.filter(d => d.value > 30 && d.value <= 70 && !d.isOther),
      low: data.filter(d => d.value <= 30 && !d.isOther)
    };
    
    // Make the chart wider by adjusting columns - target a width similar to the legend's width
    columns = 30; // Increased from default 20 to make the chart wider
    width = columns * cellSize; // Update the width based on new columns value
    
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
    
    // Generate a color scale for the data items
    colorScale = function(i) {
      // Check if this is the "Other" category
      if (data[i] && data[i].isOther) {
        return "#FFFFFF"; // White color for "Other"
      }
      // Otherwise use the standard color scale
      const colorScheme = d3.schemeTableau10;
      return colorScheme[i % colorScheme.length];
    };
  }

  function handleLegendClick(index, event) {
    // Stop event from bubbling up to document
    event.stopPropagation();
    
    selectedIndex = selectedIndex === index ? -1 : index;
    dispatch('select', { index: selectedIndex });
  }

  // Generate grid cells
  $: cells = [];
  $: {
    cells = [];
    let cellIndex = 0;
    let dataIndex = 0;
    
    for (let i = 0; i < rows; i++) {
      for (let j = 0; j < columns; j++) {
        if (cellIndex < totalItems) {
          // Find which data item this cell belongs to
          while (dataIndex < processedData.length && 
                processedData[dataIndex].cellCount <= 0) {
            dataIndex++;
          }
          
          if (dataIndex < processedData.length) {
            cells.push({
              x: j * cellSize,
              y: i * cellSize,
              dataItem: processedData[dataIndex],
              index: dataIndex
            });
            
            processedData[dataIndex].cellCount--;
            cellIndex++;
          }
        }
      }
    }
  }
</script>

<div class="waffle-container">
  <div class="waffle-chart" on:click|stopPropagation>
    <svg width={width} height={height}>
      {#each cells as cell}
        <rect
          width={cellSize - cellPadding}
          height={cellSize - cellPadding}
          x={cell.x + cellPadding / 2}
          y={cell.y + cellPadding / 2}
          rx={cellBorderRadius}
          ry={cellBorderRadius}
          fill={colorScale(cell.index)}
          stroke={cell.dataItem?.isOther ? "#dddddd" : "none"}
          stroke-width={cell.dataItem?.isOther ? 1 : 0}
          opacity={selectedIndex === -1 || selectedIndex === cell.index ? 1 : 0.3}
        />
      {/each}
    </svg>
  </div>
  
  <div class="legend-wrapper">
    <div class="legend-columns">
      <!-- High evictions column -->
      <div class="legend-column">
        <h3>More than 70 evictions</h3>
        <div class="legend-items">
          {#each categorizedData.high as item, i}
            <div 
              class="legend-item" 
              class:selected={selectedIndex === data.findIndex(d => d.label === item.label)}
              on:click={(e) => handleLegendClick(data.findIndex(d => d.label === item.label), e)}
            >
              <div class="legend-item-container">
                <div class="legend-color" style="background-color: {colorScale(data.findIndex(d => d.label === item.label))};"></div>
                <div class="legend-label">{item.label}</div>
              </div>
              <div class="legend-value">Number of evictions: {item.value}</div>
            </div>
          {/each}
        </div>
      </div>
      
      <!-- Medium evictions column -->
      <div class="legend-column">
        <h3>More than 30 evictions</h3>
        <div class="legend-items">
          {#each categorizedData.medium as item, i}
            <div 
              class="legend-item" 
              class:selected={selectedIndex === data.findIndex(d => d.label === item.label)}
              on:click={(e) => handleLegendClick(data.findIndex(d => d.label === item.label), e)}
            >
              <div class="legend-item-container">
                <div class="legend-color" style="background-color: {colorScale(data.findIndex(d => d.label === item.label))};"></div>
                <div class="legend-label">{item.label}</div>
              </div>
              <div class="legend-value">Number of evictions: {item.value}</div>
            </div>
          {/each}
        </div>
      </div>
      
      <!-- Low evictions column -->
      <div class="legend-column">
        <h3>Less than 30 evictions</h3>
        <div class="legend-items">
          {#each categorizedData.low as item, i}
            <div 
              class="legend-item" 
              class:selected={selectedIndex === data.findIndex(d => d.label === item.label)}
              on:click={(e) => handleLegendClick(data.findIndex(d => d.label === item.label), e)}
            >
              <div class="legend-item-container">
                <div class="legend-color" style="background-color: {colorScale(data.findIndex(d => d.label === item.label))};"></div>
                <div class="legend-label">{item.label}</div>
              </div>
              <div class="legend-value">Number of evictions: {item.value}</div>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>
</div>

<style>
  .waffle-container {
    display: flex;
    flex-direction: column; /* Changed from row to column for stacking */
    width: 100%;
    margin: 0 auto;
    gap: 20px; /* Reduced from 30px for vertical stacking */
    align-items: center; /* Center horizontally */
  }
  
  .waffle-chart {
    width: 100%; /* Use full width */
    display: flex;
    justify-content: center;
    background-color: rgba(255, 255, 255, 0.3);
    border-radius: 8px;
    padding: 15px;
    box-sizing: border-box;
  }
  
  .legend-wrapper {
    width: 100%; /* Use full width */
    overflow-y: auto;
    max-height: 450px;
    background-color: rgba(255, 255, 255, 0.3);
    border-radius: 8px;
    padding: 15px;
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
  }
  
  .legend-items {
    display: flex;
    flex-direction: column;
    gap: 8px;
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
  
  /* Adjusted responsive design for different screen sizes */
  @media (max-width: 1200px) {
    .legend-columns {
      flex-wrap: wrap;
    }
    
    .legend-column {
      flex-basis: calc(50% - 10px);
      min-width: 180px;
    }
  }
  
  @media (max-width: 768px) {
    .legend-columns {
      flex-direction: column;
    }
    
    .legend-column {
      flex-basis: 100%;
    }
  }
</style>