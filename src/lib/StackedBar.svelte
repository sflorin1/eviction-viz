<script>
	import * as d3 from 'd3';
	import { onMount, tick, afterUpdate } from 'svelte';

	export let data = [];
	export let width = 1000;
	export let height = 800;
	export let margin = { top: 20, right: 30, bottom: 30, left: 40 };

	let svg;
	let xAxisG, yAxisG;
	let mounted = false;


	$: keys = data.length ? Object.keys(data[0]).filter(k => k !== 'month') : [];

	$: stackedData = d3.stack().keys(keys)(data);
	$: totals = data.map(d => ({
		month: d.month,
		total: d3.sum(keys, key => d[key])
	}));
	$: xScale = d3.scaleBand()
		.domain(data.map(d => d.month))
		.range([margin.left, width - margin.right])
		.padding(0.1);

	$: yMax = d3.max(totals, d => d.total) || 0;
	$: yScale = d3.scaleLinear()
		.domain([0, yMax || 1])
		.range([height - margin.bottom, margin.top]);

	$: color = d3.scaleOrdinal(d3.schemeTableau10).domain(keys);


	$: if (mounted && xAxisG && yAxisG) {
		updateAxes();
	}

	$: if (mounted && data.length > 0) {
		updateAxesAsync();
	}

	onMount(async () => {
		mounted = true;
		await tick(); 
		updateAxes();
	});

	async function updateAxesAsync() {
		await tick();
		updateAxes();
	}

	function parseMonth(monthStr) {
		
		let date;
		if (monthStr.includes('-')) {
		
			date = new Date(monthStr + '-01');
		} else if (monthStr.includes('/')) {
		
			const parts = monthStr.split('/');
			date = new Date(parts[1], parseInt(parts[0])-1, 1);
		} else {
	
			date = new Date(monthStr);
		}
		return date;
	}

	function formatXAxisLabel(monthStr) {
		if (!monthStr) return [''];
		
		try {
			const date = parseMonth(monthStr);
			
			if (isNaN(date)) return [monthStr];
			
			const month = date.toLocaleString('default', { month: 'short' });
			const year = date.getFullYear();
			
			if (date.getMonth() === 0) {
				return [month, year.toString()];
			} else {
				return [month];
			}
		} catch (e) {
			return [monthStr];
		}
	}

	function updateAxes() {
		if (xAxisG && yAxisG) {
			d3.select(xAxisG).call(
				d3.axisBottom(xScale)
					.tickFormat((d, i) => '')
			)
			.selectAll('.tick text')
			.each(function(d, i) {
				const labels = formatXAxisLabel(d);
				const text = d3.select(this);
				text.selectAll('tspan').remove();
				labels.forEach((label, j) => {
					text.append('tspan')
						.attr('x', 0)
						.attr('dy', j === 0 ? '1.2em' : '1em') // Initial offset for month, then year
						.text(label);
				});
			});
			d3.select(yAxisG).call(d3.axisLeft(yScale));
		}
	}

	export let selectedIndex = -1;

	$: legendData = keys.map((label, index) => ({
		label,
		value: d3.sum(data, d => d[label]),
		index
	}));
</script>

<svg bind:this={svg} {width} {height}>
	{#each stackedData as layer}
		{#each layer as [y0, y1], i}
			{#if xScale(data[i].month) !== undefined}
				<g>
					<rect
						x={xScale(data[i].month)}
						y={yScale(y1)}
						width={xScale.bandwidth()}
						height={yScale(y0) - yScale(y1)}
						fill={color(layer.key)}
						opacity={selectedIndex === -1 || selectedIndex === keys.indexOf(layer.key) ? 1 : 0.3}
					/>
		
					{#if yScale(y0) - yScale(y1) > 12}
						<text
							x={xScale(data[i].month) + xScale.bandwidth() / 2}
							y={yScale((y0 + y1) / 2)}
							text-anchor="middle"
							dominant-baseline="middle"
							fill="white"
							font-size="10"
						>
							{data[i][layer.key]}
						</text>
					{/if}
				</g>
			{/if}
		{/each}
	{/each}

	{#each totals as { month, total }, i}
		{#if xScale(month) !== undefined}
			<text
				x={xScale(month) + xScale.bandwidth() / 2}
				y={yScale(total) - 4}
				text-anchor="middle"
				fill="black"
				font-size="11"
			>
				{total}
			</text>
		{/if}
	{/each}

	<g bind:this={xAxisG} transform="translate(0,{height - margin.bottom })" />
	<g bind:this={yAxisG} transform="translate({margin.left},0)" />
</svg>

<ul class="legend">
	{#each legendData as d, i}
		<li
			style="--color: {color(d.label)}"
			on:click={() => selectedIndex = selectedIndex === i ? -1 : i}
			class:selected={selectedIndex === i}
		>
			<span class="swatch"></span>
			{d.label}<em>({d.value})</em>
		</li>
	{/each}
</ul>


<style>
	svg {
		font: 10px sans-serif;
	}
	.axis path,
	.axis line {
		fill: none;
		stroke: #000;
		shape-rendering: crispEdges;
	}
    svg {
		overflow: visible;
	}

	.legend {
		border: 1px solid black;
		padding: 0.5em;
		margin: 2em 0;
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(8em, 1fr));
		list-style: none;
	}

	.legend li {
		display: flex;
		align-items: center;
		cursor: pointer;
		color: var(--color);
	}

	.legend li .swatch {
		display: inline-block;
		width: 1em;
		height: 1em;
		margin-right: 0.4em;
		border-radius: 0.25em;
		background-color: var(--color);
	}

	.legend li em {
		font-style: normal;
		color: gray;
		margin-left: 0.25em;
	}

	/* Highlighting */
	.legend li.selected {
		font-weight: bold;
		color: var(--color);
	}

	.legend:has(.selected) li:not(.selected) {
		color: gray;
		opacity: 0.5;
	}
</style>