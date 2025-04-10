<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import { base } from '$app/paths';
    import mapboxgl from "mapbox-gl";
    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';
    import WaffleChart from '$lib/WaffleChart.svelte';
    
    mapboxgl.accessToken = 'pk.eyJ1Ijoic2Zsb3JpbjEyMyIsImEiOiJjbTkwdXpxcXEwMjd3Mmlwam02ZmZkNzdnIn0.MVg9b479HenNzIayd1vGSg';
    const urlBase = 'https://api.mapbox.com/isochrone/v1/mapbox/';
    
    let evictions = [];
    let censusData = [];
    let corpRateDiffByGeoid;
    let map;
    let pieData;
    let selectedEvictorIndex = -1;
    let selectedEvictor;
    let selectedEviction = null;
    let defaultCenter = [-71.0854339, 42.3454145];
    let corpOwnRateDiff = d3.scaleQuantize()
        .domain([-.8, .3])
        .range([-.5, 0, 0.5]);
    const corpOwnRateDiffColorMap = {
        [-.5]: "#d73027", 
          [0]: "#ffffbf",    
        [0.5]: "#1a9850"   
    };
    
    // Add a legend for the census tract coloring
    let tractLegend = [
        { color: "#d73027", label: "More Owner Occupied" },
        { color: "#ffffbf", label: "Similar Rates" },
        { color: "#1a9850", label: "More Corporately Owned" }
    ];
    
    async function loadMap(){
        map = new mapboxgl.Map({
            container: "map",
            style: "mapbox://styles/mapbox/streets-v12",
            zoom: 13,
            minZoom: 5,
            maxZoom: 18,
            center: filteredCenter
        });
        await new Promise(resolve => map.on("load", resolve));
        censusData = await d3.csv(`${base}/census_data.csv`, d => {
            return {
                geoid: d.GEOID,
                cor: +d.corp_own_rate,
                oor: +d.own_occ_rate
            }
        });
            
        corpRateDiffByGeoid = Object.fromEntries(
            censusData.map(d => [d.geoid, d.cor - d.oor])
        );

        const geojson = await d3.json(`${base}/Metro_Boston_Census_Tracts copy.geojson`);
        geojson.features.forEach(f => {
            const geoid = f.properties.geoid;
            const rawValue = corpRateDiffByGeoid[geoid];
            if (rawValue != null) {
                const quantized = corpOwnRateDiff(rawValue);
                f.properties.color = corpOwnRateDiffColorMap[quantized];
            } else {
                f.properties.color = "#ccc";
            }
        });
        
        map.addSource("boston_census_tracts", {
            type: "geojson", 
            data: geojson
        });
        
        map.addLayer({
            id: "census_tract",
            type: "fill",
            source: "boston_census_tracts",
            paint: {
                "fill-color": ["get", "color"],
                "fill-opacity": 0.6
            },
        });
    }
    
    function getCoords(eviction) {
        let point = new mapboxgl.LngLat(+eviction.long, +eviction.lat);
        let {x, y} = map.project(point);
        return {cx: x, cy: y};
    }
    
    let mapViewChanged = 0;
    $: map?.on("move", evt => mapViewChanged++);
    
    onMount(async () => {
        // Load fonts first
        await Promise.all([
            loadFont('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap'),
            loadFont('https://fonts.googleapis.com/css2?family=Inconsolata:wght@400;700&display=swap')
        ]);
        
        // Set the background color
        document.body.style.backgroundColor = "#F9EAE1";
        
        // Add event listener for clicking outside the waffle chart
        document.addEventListener('click', handleDocumentClick);
        
        evictions = await d3.csv(`${base}/total_merged_eviction_df2.csv`);
        await loadMap();
        
        // Clean up event listener when component is destroyed
        return () => {
            document.removeEventListener('click', handleDocumentClick);
        };
    });
    
    function loadFont(url) {
        return new Promise((resolve, reject) => {
            const link = document.createElement('link');
            link.href = url;
            link.rel = 'stylesheet';
            link.onload = () => resolve();
            link.onerror = () => reject();
            document.head.appendChild(link);
        });
    }
    
    let cutoff = 30;
    
    $: {
        let rolledData = d3.rollups(evictions, v => v.length, d => d.filtered_name_plaintiff);
        rolledData.sort((a, b) => b[1] - a[1]);
        let cutOffRolledData = rolledData.slice(0, cutoff);
        let otherData = rolledData.slice(cutoff);
        let otherSum = d3.sum(otherData, d => d[1]);
        
        if (otherSum > 0) {
            cutOffRolledData.push(["Other", otherSum]);
        }
        
        pieData = cutOffRolledData.map(([name_plaintiff, count]) => {
            return {
                value: count, 
                label: name_plaintiff,
                // Flag for special "Other" category
                isOther: name_plaintiff === "Other",
                // Store average coordinates for each evictor for map centering
                avgCoords: [
                    d3.mean(evictions.filter(e => e.filtered_name_plaintiff === name_plaintiff), d => +d.long) || defaultCenter[0],
                    d3.mean(evictions.filter(e => e.filtered_name_plaintiff === name_plaintiff), d => +d.lat) || defaultCenter[1]
                ]
            };
        });
    }
    
    $: groupedEvictionsByAddress = d3.rollups(
        filteredEvictions,
        v => v.length,
        d => d.add_p
    ).reduce((acc, [key, count]) => {
        acc[key] = count;
        return acc;
    }, {});
    
    $: rScale = d3.scaleSqrt()
        .domain(d3.extent(Object.values(groupedEvictionsByAddress)))
        .range([4, 12]);
    
    $: selectedEvictor = selectedEvictorIndex > -1 ? pieData[selectedEvictorIndex].label : null;
    
    $: filteredEvictions = selectedEvictorIndex === -1 
        ? evictions
        : selectedEvictor === "Other" 
            ? evictions.filter(eviction => !pieData.some(p => p.label === eviction.filtered_name_plaintiff && !p.isOther))
            : evictions.filter(eviction => eviction.filtered_name_plaintiff === selectedEvictor);
        
    $: filteredCenter = selectedEvictorIndex === -1 
        ? defaultCenter
        : pieData[selectedEvictorIndex].avgCoords;
        
    $: if (map && filteredCenter) {
        map.flyTo({ center: filteredCenter, zoom: 13, speed: 1.2 });
    }
    
    function handleWaffleSelection(event) {
        selectedEvictorIndex = event.detail.index;
    }
    
    // Handle clicks outside the waffle chart to reset selection
    function handleDocumentClick(event) {
        // Check if the click was outside the waffle chart, legend and not on the map
        const waffleEl = document.getElementById('waffle_chart');
        const mapEl = document.getElementById('map');
        const tooltipEl = document.querySelector('.tooltip');
        const legendEl = document.querySelector('.legend-wrapper');
        
        // Make sure we're not clicking on the waffle chart, map, tooltip or legend
        const isOutsideElements = (
            (!waffleEl || !waffleEl.contains(event.target)) &&
            (!mapEl || !mapEl.contains(event.target)) &&
            (!tooltipEl || !tooltipEl.contains(event.target)) &&
            (!legendEl || !legendEl.contains(event.target))
        );
        
        if (isOutsideElements && selectedEvictorIndex !== -1) {
            selectedEvictorIndex = -1;
        }
    }
</script>

<h1>Serial Evictors in Boston</h1>
<p>Total evictions in Boston from 2020-2023: {evictions.length}</p>

<div class="tract-legend">
    <h3>Census Tract Colors</h3>
    <div class="tract-legend-items">
        {#each tractLegend as item}
            <div class="tract-legend-item">
                <div class="tract-color" style="background-color: {item.color};"></div>
                <div class="tract-label">{item.label}</div>
            </div>
        {/each}
    </div>
</div>

<div id="map" on:click|stopPropagation>
    <svg>
    {#key mapViewChanged}
    {#each filteredEvictions as eviction}
        <circle 
            cx={getCoords(eviction).cx}
            cy={getCoords(eviction).cy}
            class={eviction?.add_p === selectedEviction?.add_p ? "selected" : ""}
            on:mouseenter={() => selectedEviction = selectedEviction?.add_p !== eviction?.add_p ? eviction : null}
            on:mouseleave={() => selectedEviction = null}
            on:click|stopPropagation
            r={rScale(groupedEvictionsByAddress[eviction.add_p])}
            fill="steelblue" />
    {/each}
    {/key}
    </svg>
</div>

{#if selectedEviction}
<dl class="info tooltip">
    <dt>Date</dt>
    <dd>{selectedEviction.file_date}</dd>
    <dt>Address</dt>
    <dd>{selectedEviction.add_p}</dd>
    <dt>Evictor</dt>
    <dd>{selectedEviction.name_plaintiff}</dd>
    <dt>Number of Evictions</dt>
    <dd>{groupedEvictionsByAddress[selectedEviction.add_p]}</dd>
</dl>
{/if}

<div class="chart-container">
    <h2>Top Evictors by Number of Evictions</h2>
    <div id="waffle_chart">
        <WaffleChart 
            data={pieData} 
            bind:selectedIndex={selectedEvictorIndex}
            on:select={handleWaffleSelection}
            rows={10}
            columns={20}
            cellSize={24}
            cellPadding={3}
            cellBorderRadius={0}
        />
    </div>
</div>

<style>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Inconsolata:wght@400;700&display=swap');

:global(body) {
    font-family: 'Inconsolata', monospace;
    color: #333;
    line-height: 1.6;
    margin: 0;
    padding: 20px;
    background-color: #F9EAE1;
}

h1, h2, h3, h4, h5, h6 {
    font-family: 'Bebas Neue', sans-serif;
    letter-spacing: 1px;
}

p, div, span, dl, dt, dd {
    font-family: 'Inconsolata', monospace;
}

svg {
    position: absolute;
    z-index: 1;
    width: 100%;
    height: 100%;
    pointer-events: none;
}

#map {
    flex: 1;
    width: 100%;
    height: 70vh;
    position: relative;
    margin-bottom: 20px;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

#map:has(circle.selected) circle:not(.selected) {
    opacity: 0.2;
}

circle {
    stroke: white;
    pointer-events: auto;
    transition: all 0.2s ease;
}

circle.selected {
    fill: orange;
    stroke: black;
    stroke-width: 2;
    filter: drop-shadow(0 0 2px black);
    z-index: 10; 
    opacity: 1;
}

.tooltip {
    position: fixed;
    top: 5em; 
    left: 1em;
    background-color: white;
    box-shadow: 0 2px 10px rgba(0,0,0,0.2);
    border-radius: 5px;
    padding: 12px;
    z-index: 1000;
}

dl.info {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 0.5em 1em;
    transition-duration: 500ms;
    transition-property: opacity, visibility;
}

dl.info[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
}

dt {
    font-weight: bold;
    color: #555;
}

dd {
    margin: 0;
}

.chart-container {
    margin-top: 30px;
    padding: 0 20px 40px;
    background-color: rgba(255, 255, 255, 0.2);
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.chart-container h2 {
    text-align: center;
    margin-bottom: 25px;
    padding-top: 20px;
    color: #333;
    font-size: 2rem;
}

#waffle_chart {
    width: 100%;
    max-width: 1400px;
    margin: 0 auto;
    display: flex;
    justify-content: center;
}

.tract-legend {
    margin-bottom: 15px;
    padding: 10px;
    background-color: rgba(255, 255, 255, 0.6);
    border-radius: 6px;
}

.tract-legend h3 {
    margin: 0 0 10px 0;
    font-size: 1.2rem;
    color: #333;
}

.tract-legend-items {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
}

.tract-legend-item {
    display: flex;
    align-items: center;
}

.tract-color {
    width: 16px;
    height: 16px;
    border-radius: 3px;
    margin-right: 6px;
}

.tract-label {
    font-family: 'Inconsolata', monospace;
}

h1 {
    color: #2c3e50;
    margin-top: 20px;
    margin-bottom: 10px;
    font-size: 3rem;
}

/* Make the page more responsive */
@media (max-width: 768px) {
    #map {
        height: 50vh;
    }
    
    .tract-legend-items {
        flex-direction: column;
        gap: 8px;
    }
    
    #waffle_chart :global(.waffle-container) {
        flex-direction: column;
        align-items: center;
    }
    
    #waffle_chart :global(.waffle-chart) {
        margin-bottom: 20px;
        overflow-x: auto;
    }
    
    #waffle_chart :global(.legend-wrapper) {
        width: 100%;
    }
}
</style>