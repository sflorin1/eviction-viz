<script>
	import * as d3 from 'd3';
	import { onMount, tick, afterUpdate } from 'svelte';

	export let data = [];
	export let width = 600;
	export let height = 500;
	export let margin = { top: 50, right: 30, bottom: 50, left: 50 };

	let svg;
	let xAxisG, yAxisG;
	let mounted = false;
	let hoveredMonth = null;

	$: keys = data.length ? Object.keys(data[0]).filter(k => k !== 'month') : [];

	$: stackedData = d3.stack().keys(keys)(data);
	$: totals = data.map(d => ({
		month: d.month,
		total: d3.sum(keys, key => d[key])
	}));
	
	$: xScale = d3.scaleBand()
		.domain(data.map(d => d.month))
		.range([margin.left, width - margin.right])
		.padding(0.2);  // Increased padding for more modern look

	$: yMax = d3.max(totals, d => d.total) || 0;
	$: yScale = d3.scaleLinear()
		.domain([0, yMax * 1.1]) // Add some headroom for total labels
		.range([height - margin.bottom, margin.top]);

	// Using a more vibrant color scheme that matches the parent component's style
	$: color = d3.scaleOrdinal()
		.domain(keys)
		.range(['#06D6A0', '#1B9AAA', '#EF476F', '#FFC43D', '#E56B6F', '#118AB2']);

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
			
			if (isNaN(date)) return [monthStr.charAt(0), ''];
			
			// Just get the first letter of the month
			const month = date.toLocaleString('default', { month: 'short' }).charAt(0);
			const year = date.getFullYear();
			
			// Always return the year as second line
			return [month, year.toString()];
		} catch (e) {
			return [monthStr.charAt(0), ''];
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
						.attr('dy', j === 0 ? '1.2em' : '1em')
						.text(label);
				});
			});
			
			d3.select(yAxisG).call(
				d3.axisLeft(yScale)
				.ticks(5)
				.tickFormat(d => d)
			);
		}
	}

	export let selectedIndex = -1;
	let hoverIndex = -1;

	function setHoverIndex(index) {
		hoverIndex = index;
	}

	function clearHoverIndex() {
		hoverIndex = -1;
	}

	$: legendData = keys.map((label, index) => ({
		label,
		value: d3.sum(data, d => d[label]),
		index
	}));

	function handleMonthHover(month) {
		hoveredMonth = month;
	}

	function handleMonthLeave() {
		hoveredMonth = null;
	}

	// Calculate percentage for each segment
	function getSegmentPercentage(value, total) {
		return total > 0 ? Math.round((value / total) * 100) : 0;
	}
</script>

<div class="chart-container" on:click={() => selectedIndex = -1}>
	<svg bind:this={svg} {width} {height} on:click|stopPropagation>
		<!-- Background grid lines -->
		<g class="grid-lines">
			{#each yScale.ticks(5) as tick}
				<line 
					x1={margin.left} 
					y1={yScale(tick)} 
					x2={width - margin.right} 
					y2={yScale(tick)} 
					stroke="#e0e0e0" 
					stroke-dasharray="3,3"
				/>
			{/each}
		</g>

		<!-- Bars -->
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
							opacity={selectedIndex === -1 && hoverIndex === -1 ? 0.95 : 
									(selectedIndex === keys.indexOf(layer.key) || hoverIndex === keys.indexOf(layer.key)) ? 1 : 0.3}
							rx="2" 
							ry="2"
							on:mouseenter={() => handleMonthHover(data[i].month)}
							on:mouseleave={handleMonthLeave}
							class="bar-segment"
						/>
			
						<!-- Value labels but only for visible segments -->
						{#if (yScale(y0) - yScale(y1) > 14) && (selectedIndex === -1 || selectedIndex === keys.indexOf(layer.key))}
							<text
								x={xScale(data[i].month) + xScale.bandwidth() / 2}
								y={yScale((y0 + y1) / 2)}
								text-anchor="middle"
								dominant-baseline="middle"
								fill="white"
								font-size="10"
								font-weight="bold"
								font-family="'Geist Mono', monospace"
							>
								{data[i][layer.key]}
							</text>
						{/if}
					</g>
				{/if}
			{/each}
		{/each}

		<!-- Total labels above bars -->
		{#each totals as { month, total }, i}
			{#if xScale(month) !== undefined}
				<text
					x={xScale(month) + xScale.bandwidth() / 2}
					y={yScale(total) - 5}
					text-anchor="middle"
					fill="#14110F"
					font-size="10"
					font-weight="bold"
					font-family="'Geist Mono', monospace"
					class="total-label"
				>
					{total}
				</text>
			{/if}
		{/each}

		<!-- Chart title -->
		<text
			x={width / 2}
			y={margin.top / 2}
			text-anchor="middle"
			font-size="16"
			font-weight="bold"
			font-family="'Bebas Neue', sans-serif"
			fill="#14110F"
		>
			EVICTION NOTICES BY TYPE
		</text>

		<!-- Axes -->
		<g bind:this={xAxisG} transform="translate(0,{height - margin.bottom})" class="axis x-axis" />
		<g bind:this={yAxisG} transform="translate({margin.left},0)" class="axis y-axis" />
		
		<!-- Axis labels -->
		<text
			x={width / 2}
			y={height - 5}
			text-anchor="middle"
			font-size="14"
			font-family="'Geist Mono', monospace"
		>
			Months
		</text>
		
		<text
			transform="rotate(-90)"
			x={-(height / 2)}
			y={margin.left / 3}
			text-anchor="middle"
			font-size="14"
			font-family="'Geist Mono', monospace"
		>
			Number of Evictions Filings
		</text>
	</svg>

	<!-- Enhanced interactive legend -->
	<div class="legend-container">
		<div class="legend-title">EVICTION FILINGS BY TYPES</div>
		<ul class="legend" on:click|stopPropagation>
			{#each legendData as d, i}
				<li
					style="--color: {color(d.label)}"
					on:click|stopPropagation={() => selectedIndex = selectedIndex === i ? -1 : i}
					on:mouseenter={() => setHoverIndex(i)}
					on:mouseleave={clearHoverIndex}
					class:selected={selectedIndex === i}
					class:hovered={hoverIndex === i}
				>
					<span class="swatch"></span>
					<div class="legend-text">
						<span class="legend-label">{d.label}</span>
						<span class="legend-value">{d.value}</span>
						<span class="legend-percent">({getSegmentPercentage(d.value, d3.sum(legendData, d => d.value))}%)</span>
					</div>
				</li>
			{/each}
		</ul>
	</div>
</div>

<style>
	.chart-container {
		display: flex;
		flex-direction: column;
		width: 100%;
		font-family: 'Geist Mono', monospace;
	}

	svg {
		font: 12px 'Geist Mono', monospace;
		overflow: visible;
	}

	.axis path,
	.axis line {
		stroke: #666;
		stroke-width: 0.5px;
	}

	.y-axis text,
	.x-axis text {
		fill: #14110F;
		font-size: 12px;
	}

	.bar-segment {
		transition: opacity 0.2s ease, filter 0.2s ease;
	}
	
	.bar-segment:hover {
		filter: brightness(1.1);
	}

	.total-label {
		transition: font-size 0.2s ease;
	}

	.legend-container {
		margin-top: 1rem;
		border: none;
		border-radius: 0;
		padding: 0.5rem 0;
		background-color: transparent;
	}

	.legend-title {
		font-family: 'Bebas Neue', sans-serif;
		font-size: 16px;
		margin-bottom: 0.25rem;
		color: #14110F;
		letter-spacing: 1px;
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

	/* Highlighting */
	.legend li.selected {
		background-color: rgba(0,0,0,0.05);
		box-shadow: 0 0 0 1px rgba(0,0,0,0.1);
	}

	.legend li.selected .legend-label,
	.legend li.hovered .legend-label {
		font-weight: bold;
	}

	.legend:has(.selected) li:not(.selected) {
		opacity: 0.7;
	}
</style>