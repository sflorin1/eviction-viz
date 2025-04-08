<script>
    import * as d3 from 'd3';
    
    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    let arc = arcGenerator({
        startAngle : 0,
        endAngle : 2*Math.PI   
    });
    let colors = d3.scaleOrdinal(d3.schemeTableau10);
    /*let data = [
        { value: 1, label: "apples" },
        { value: 2, label: "oranges" },
        { value: 3, label: "mangos" },
        { value: 4, label: "pears" },
        { value: 5, label: "limes" },
        { value: 5, label: "cherries" }
    ];*/
    export let data = [];
    export let selectedIndex = -1;

    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData;
    let arcs;
    $:{
        arcData = sliceGenerator(data);
        arcs = arcData.map(d => arcGenerator(d));
    }
</script>
<div class="container">
<svg viewBox="-50 -50 100 100">
    {#each arcs as arc, index}
        <path d={arc} fill={colors(index)}
        class:selected={selectedIndex === index}
        on:click = {e => selectedIndex = selectedIndex === index ? -1 : index}/>
    {/each}
</svg>
<ul class = "legend">
    {#each data as d, index}
        <li style = "--color: {colors(index)}" 
        class:selected = {selectedIndex === index}>
            <span class = "swatch"></span>
            {d.label}<em>({d.value})</em>
        </li>
    {/each}
</ul>
</div>
<style>
    svg {
        max-width: 20em;
        margin-block: 2em;
        overflow: visible;
    }
    svg:has(path:hover) path:not(:hover) {
        opacity: 0.5;
    }
    path {
	    transition: 300ms;
    }
    .swatch {
        display: inline-block;
        width: 1em;
        height: 1em;
        margin-right: 0.4em;
        border-radius: 0.25em;
        background-color: var(--color);
        
    }
    ul {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(8em, 1fr));
    }
    li {
        display: flex;
        align-items: center;
    }
    em {
        padding: 0.25em;
    }
    .legend {
        border: 1px solid black;
        padding: .5em;
        margin: 3em;
        flex: 1;   
    }
    .container{
        display: flex;
    }
    .selected {
        --color: oklch(60% 45% 0) !important;
        
        &:is(path) {
            fill: var(--color) !important;
            opacity: 100% !important;
        }
        
        &:is(li) {
            color: var(--color);
        }
    }


    ul:has(.selected) li:not(.selected) {
    	color: gray;
    }
    svg:has(.selected) path:not(.selected) {
    	opacity: 50%;
    }
    path:hover {
    	opacity: 100% !important;
    }

</style>