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
import Pie from '$lib/Pie.svelte';

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

/**
* TODO: 
 *  Color something off of corporate ownership rate vs owner occupancy rate (census tracts?)  [DONE, need legend]
 *  Better display for multiple evictions in home (fix opacity)
 *  Maybe do little houses for each eviction, offset location by random amount 
 *      so each eviction appears separately
 * Clean up data a bit to better identify serial evictors
 * Multiple points highlighted at same time.
 */


/**
 * 
 * RED = More owner occupied
 * YELLOW = more similar rates
 * Green = More corporately owned
 */
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
        censusData =  d3.csv(`${base}/census_data.csv`, d => {
            return {
                geoid: d.GEOID,
                cor: +d.corp_own_rate,
                oor: +d.own_occ_rate
            }
        }).then(
            data => {
            corpRateDiffByGeoid = Object.fromEntries(
                data.map(d => [d.geoid, d.cor - d.oor])
            );
            });

        const geojson = await d3.json(`${base}/Metro_Boston_Census_Tracts copy.geojson`);
        geojson.features.forEach(f => {
            const geoid = f.properties.geoid;
            const rawValue = corpRateDiffByGeoid[geoid];
            if (rawValue != null) {
            const quantized = corpOwnRateDiff(rawValue); // returns -0.3, 0, or 0.3
            console.log(geoid, rawValue, quantized, corpOwnRateDiffColorMap[quantized]);
            f.properties.color = corpOwnRateDiffColorMap[quantized];}
            else{
                f.properties.color = "#ccc";
            }
        });
        console.log(geojson);
        map.addSource("boston_census_tracts", {type: "geojson", data: geojson});
        //map.addSource("boston_census_tracts", {
	    //    type: "geojson",
	    //    data: `${base}/Metro_Boston_Census_Tracts copy.geojson`,
        //});
        
        map.addLayer({
	        id: "census_tract",
	        type: "fill", // one of the supported layer types, e.g. line, circle, etc.
	        source: "boston_census_tracts", // The id we specified in `addSource()`
	        paint: {
		        "fill-color": ["get", "color"],
                "fill-opacity": 0.6
	        },
        });
        
        /**map.addSource("test_data",{
            type: "geojson",
            data: tractData,
        });
        map.addLayer({
            id: "tract-fill",
            type: "fill",
            source: "test_data",
            paint: {
                "fill-color": "#f1eef6",
                "fill-opacity": 0.7
            }
        });**/
}
function getCoords (eviction) {
	let point = new mapboxgl.LngLat(+eviction.long, +eviction.lat);
	let {x, y} = map.project(point);
	return {cx: x, cy: y};
}
let mapViewChanged = 0;
$: map?.on("move", evt => mapViewChanged++);

onMount(async () => {
    loadMap();
    evictions = await d3.csv(`${base}/total_merged_eviction_df.csv`);
});

let cutoff = 20;

$: {
        pieData = {};
        let rolledData = d3.rollups(evictions, v => v.length, d => d.name_plaintiff);
        rolledData.sort((a, b) => b[1] - a[1]);
        let cutOffRolledData = rolledData.slice(0,cutoff);
        let otherData = rolledData.slice(cutoff);
        let otherSum = d3.sum(otherData, d => d[1]);
        
        if (otherSum > 0) {
            cutOffRolledData.push(["Other", otherSum]);
        }
        pieData = cutOffRolledData.map(([name_plaintiff, count]) => {
        return {value: count, label: name_plaintiff };
        
         
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

$: rScale = d3.scaleSqrt().domain(d3.extent(Object.values(groupedEvictionsByAddress))).range([4,12]);
//$: rScale = d3.scaleSqrt()
//	    .domain([0, d3.max(filteredStations, d => d.totalTraffic) || 0])
//	    .range(radiusRange);

//d3.scaleSqrt().domain(d3.extent(Object.values(groupedEvictionsByAddress))).range(2,10);
$: selectedEvictor = selectedEvictorIndex > -1 ? pieData[selectedEvictorIndex].label : null;
$: filteredEvictions = selectedEvictorIndex=== -1 ? evictions: evictions.filter(eviction => {
        return eviction.name_plaintiff ===  selectedEvictor
        })
$: filteredCenter = selectedEvictorIndex=== -1 ? defaultCenter: [d3.mean(filteredEvictions, d=>  d.long), d3.mean(filteredEvictions, d=>  d.lat)];
$: if (map && filteredCenter) {
    map.flyTo({ center: filteredCenter, zoom: 13, speed: 1.2});
}

</script>
<h1>Serial Evictors in Boston</h1>
<p>Total evictions in Boston from 2020-2023: {evictions.length}</p>


<div id="map">
    <svg>
    {#key mapViewChanged}
    {#each filteredEvictions as eviction}
    <circle cx={ getCoords(eviction).cx }
        cy={ getCoords(eviction).cy }
        class={eviction?.add_p === selectedEviction?.add_p ? "selected" : ""}
        on:mouseenter={() => selectedEviction = selectedEviction?.add_p !== eviction?.add_p ? eviction : null}
        on:mouseleave={()=> selectedEviction = null}
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

<!-- Add: Time, author, lines edited -->

<div id = "pie_chart">
    <Pie data = {pieData} bind:selectedIndex={selectedEvictorIndex}/>
</div>
<style>
svg{
    position: absolute;
    z-index: 1;
    width: 100%;
    height: 100%;
    pointer-events: none;
}
#map {
    flex: 1;
    width: 100%;
    height: 80vh;
    position: relative;
    &:has(circle.selected) circle:not(.selected) {
		opacity: 0.5;
	}
}
circle {
    stroke: white;
    pointer-events: auto;
}
circle.selected {
    fill: orange;
    stroke: black;
    stroke-width: 2;
}
.tooltip {
            position: fixed;
            top: 5em; 
            left: 1em;
            background-color: oklch(100% 0% 0 / 80%);
            box-shadow: 10px;
            border-radius: 5px;
            padding: 10px;
        }
dl.info{
        display: grid;
        grid-template-rows: 8fr;
        gap: 1em;
        transition-duration: 500ms;
        transition-property: opacity, visibility;
        &[hidden]:not(:hover, :focus-within) {
        opacity: 0;
        visibility: hidden;
}

}
dt{
    grid-column: 1;
}
dd{
    grid-column: 2;
}
</style>
