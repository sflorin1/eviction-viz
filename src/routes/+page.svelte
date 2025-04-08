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
let map;
let pieData;
let selectedEvictorIndex = -1;
let selectedEvictor;
let selectedEviction = null;
let defaultCenter = [-71.0854339, 42.3454145];
/**let tractData = {
    "25017350103": 125,
    "25017351500": 250,
    "25017373100": 180,
    "25017373600": 230,
    "25021400600": 110
};**/

/**
 * TODO:
 *  Make it so selections on pie chart filter map data [DONE]
 *  Maybe recenter map so evictor properties are centered [DONE]
 *  Make tool tip appears when hovering over eviction (location, number of evictions, owner, tract demo?) [DONE]
 *  Maybe group evictions based on location, size off number of evictions at a certain property [DONE]
 *  Color something off of corporate ownership rate vs owner occupancy rate (census tracts?) 
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
            console.log(otherSum);
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
    {console.log(d3.extent(Object.values(groupedEvictionsByAddress)))}
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
