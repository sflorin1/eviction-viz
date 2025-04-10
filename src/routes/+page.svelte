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
    [-.5]: "#EA553E", 
      [0]: "#f2917e",    
    [0.5]: "#f5c5b8"   
    };

/**
* TODO: 
 *  Color something off of corporate ownership rate vs owner occupancy rate (census tracts?)  [DONE, need legend]
 *  Better display for multiple evictions in home (fix opacity) 
 *  Maybe do little houses for each eviction, offset location by random amount 
 *      so each eviction appears separately [Doesn't really work] 
 *      [One of the other groups got negative feedback for using houses to = evictions, so lets pass on this -JD]
 * Clean up data a bit to better identify serial evictors [DONE]
 * Multiple points highlighted at same time. [Leaving for now]
 * Filter points outside of Boston (see Brookline/Newton area) [nice to have] -JD
 * Lower map centerpoint coordinates [nice to have] -JD
 */


/**
 * 
 * dark-red (-.5) = More owner occupied
 * mid-red (0) = more similar rates
 * light-red (0.5) = More corporately owned
 */
async function loadMap(){
        map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/mapbox/light-v11",
        zoom: 11,
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
            f.properties.color = corpOwnRateDiffColorMap[quantized];}
            else{
                f.properties.color = "#ccc";
            }
        });
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
    evictions = await d3.csv(`${base}/total_merged_eviction_df2.csv`);
});

let cutoff = 30;

$: {
        pieData = {};
        let rolledData = d3.rollups(evictions, v => v.length, d => d.filtered_name_plaintiff);
        console.log(rolledData);
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
        return eviction.filtered_name_plaintiff ===  selectedEvictor
        })
$: filteredCenter = selectedEvictorIndex=== -1 ? defaultCenter: [d3.mean(filteredEvictions, d=>  d.long), d3.mean(filteredEvictions, d=>  d.lat)];
$: if (map && filteredCenter) {
    map.flyTo({ center: filteredCenter, zoom: 11, speed: 1.2});
}

</script>
<h1>Evictions in Boston by Address</h1>
<h3>Executed evictions in Boston from 2020-2023: {evictions.length}</h3>


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
        fill= #3943B7 
        />

        <!--<image
        href="house.svg"
        x={getCoords(eviction).cx - rScale(groupedEvictionsByAddress[eviction.add_p]) / 2}
        y={getCoords(eviction).cy - rScale(groupedEvictionsByAddress[eviction.add_p]) / 2}
        width={rScale(groupedEvictionsByAddress[eviction.add_p])}
        height={rScale(groupedEvictionsByAddress[eviction.add_p])}
        class={eviction?.add_p === selectedEviction?.add_p ? "selected" : ""}
        on:mouseenter={() => selectedEviction = selectedEviction?.add_p !== eviction?.add_p ? eviction : null}
        on:mouseleave={()=> selectedEviction = null}
        fill="steelblue" 
        style="pointer-events: auto;"/>-->
    {/each}
    {/key}
    </svg>

    <div id="legend">
        <h4>Evictions Legend</h4>


        <div class="size-scale">
            <h5>Circle Size = # of Evictions</h5>
            <div class="svg-wrapper-1-outer">
                <div class="svg-wrapper-1-inner">
                    <svg width="100%" height="60">
                        <circle cx="20" cy="30" r="4" fill="#3943B7" stroke="white"/>
                        <circle cx="60" cy="30" r="8" fill="#3943B7" stroke="white"/>
                        <circle cx="100" cy="30" r="12" fill="#3943B7" stroke="white"/>
                        <text x="20" y="55" text-anchor="middle" font-size="10">Few</text>
                        <text x="60" y="55" text-anchor="middle" font-size="10">Some</text>
                        <text x="100" y="55" text-anchor="middle" font-size="10">Many</text>
                    </svg>
                </div>
            </div>
            
        </div>
        
        <div class="color-items">
            <h5>Ownership Difference</h5>
            <div class="color-item">
                <div class="color-box" style="background: #EA553E;"></div>
                <div>More owner-occupied</div>
            </div>
            <div class="color-item">
                <div class="color-box" style="background: #f2917e;"></div>
                <div>Similar rates</div>
            </div>
            <div class="color-item">
                <div class="color-box" style="background: #f5c5b8;"></div>
                <div>More corporate-owned</div>
            </div>
        </div>
    </div>
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
		opacity: 0.2;
	}
    &:has(image.selected) image:not(.selected) {
		opacity: 0.3;
	}
}
circle, image {
    stroke: white;
    pointer-events: auto;
    opacity: 0.7;
}
circle.selected, image.selected {
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

#legend {
    display:grid;
    grid-template-rows: auto auto auto;
    position: absolute;
    bottom: 20px;
    right: 20px;
    background: white;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 0 5px rgba(0,0,0,0.3);
    z-index: 2;
    max-width: 200px;
    font-size: 12px;
    /* flex-direction: column; */
    gap: 10px;
}

/* .legend-section {
    display: flex;
    flex-direction: column;
    gap: 5px;
} */

.legend-item {
    display: flex;
    align-items: center;
}

.legend-color {
    width: 20px;
    height: 20px;
    margin-right: 8px;
    border: 1px solid #ccc;
}

.size-scale {
    /*display: contents; *//* Allows h5 to be part of grid */
}

.size-scale h5 {
    grid-row: 2;
}

.size-scale svg {
    display: block;
    margin: 0 auto;
    grid-row: 2;
    margin-top: 25px; /* Space for h5 */
}

.color-items {
    display: flex;
    flex-direction: column;
    gap: 5px;
    grid-row: 3;
}

.color-item {
    display: flex;
    align-items: center;
}

.color-box {
    width: 15px;
    height: 15px;
    margin-right: 8px;
    border: 1px solid #ccc;
}

h4, h5 {
    margin: 0 0 5px 0;
    font-weight: bold;
}

h4 {
    font-size: 14px;
    border-bottom: 1px solid #eee;
    padding-bottom: 5px;
    margin-bottom: 10px;
}

.svg-wrapper-1-outer {
    height: 60px; 
}

.svg-wrapper-1-inner {
    height: 60px; 
    position: relative;
    top: -30px; 
}
</style>
