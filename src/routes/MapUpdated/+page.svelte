// Key changes to fix the WaffleChart display issue

// 1. Update the WaffleChart component parameters to be more dynamic
<WaffleChart 
    data={pieData} 
    bind:selectedIndex={selectedEvictorIndex}
    on:select={handleWaffleSelection}
    rows={8}
    columns={10}
    cellSize={14} 
    cellPadding={2} 
    cellBorderRadius={0}
/>

// 2. Update CSS for chart container and waffle chart

/* Update the chart container CSS */
.chart-container {
    flex: 3;
    width: 30%;
    padding: 15px; /* Reduced padding to give more space to the chart */
    background-color: rgba(255, 255, 255, 0.2);
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    display: flex;
    flex-direction: column;
    overflow: hidden; /* Changed from auto to hidden */
}

.chart-section {
    height: 100%;
    display: flex;
    flex-direction: column;
    overflow: hidden; /* Added to prevent scrolling within section */
}

/* Improve the sizing for the waffle chart */
#waffle_chart {
    flex: 1;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: flex-start; /* Changed from center to ensure chart starts at top */
    overflow: visible; /* Changed from auto to visible */
    max-height: 100%; /* Ensure it doesn't grow beyond container */
}

/* Add these new styles for better waffle chart handling */
:global(.waffle-container) {
    width: 100% !important;
    height: auto !important;
    display: flex;
    flex-direction: column;
}

:global(.waffle-chart) {
    width: 100%;
    height: auto;
    display: flex;
    justify-content: center;
}

:global(.legend-wrapper) {
    width: 100%;
    display: flex;
    flex-direction: column;
    margin-top: 10px;
    max-height: 200px; /* Limit legend height */
    overflow-y: auto; /* Allow scrolling for legend only */
}

:global(.legend-item) {
    display: flex;
    align-items: center;
    font-size: 12px;
    padding: 2px 0;
}

/* Better responsive handling for the dashboard */
@media (max-width: 1200px) {
    .dashboard-container {
        flex-direction: column;
        height: auto;
    }
    
    #map-container, .chart-container {
        width: 100%;
    }
    
    #map {
        height: 50vh;
    }
    
    .chart-container {
        height: 80vh; /* Give more height on smaller screens */
    }
    
    #waffle_chart {
        height: 100%;
        overflow: visible;
    }
    
    :global(.waffle-container) {
        flex-direction: column !important;
        align-items: center;
        justify-content: flex-start;
        height: 100% !important;
    }
    
    :global(.waffle-chart) {
        height: auto;
        margin-bottom: 10px;
    }
    
    :global(.legend-wrapper) {
        width: 100% !important;
        max-height: 30vh;
    }
}

/* For very small screens */
@media (max-width: 768px) {
    .chart-container {
        height: auto;
        min-height: 80vh;
    }
    
    :global(.waffle-chart) {
        transform: scale(0.9);
        transform-origin: top center;
    }
}