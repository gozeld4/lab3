<script>
    import * as d3 from 'd3';

    let width = 500;
    let height = 240;

    export let data = [];
    export let title = "";

    // Extra left margin to fit language name labels on the y-axis
    let margin = { top: 32, right: 170, bottom: 42, left: 78 };
    let innerWidth  = width  - margin.left - margin.right;
    let innerHeight = height - margin.top  - margin.bottom;

    // y is the categorical axis (labels), x is the quantitative axis (values)
    $: yScale = d3.scaleBand()
        .domain(data.map(d => d.label))
        .range([0, innerHeight])
        .padding(0.2);

    $: xScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.value) || 1])
        .range([0, innerWidth]);

    $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
        .domain(data.map(d => d.label));

    $: maxBar = d3.greatest(data, d => d.value);

    let xAxis, yAxis;

    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(
            d3.axisBottom(xScale)
                .ticks(Math.min(d3.max(data, d => d.value) || 1, 10))
        );
        d3.select(yAxis).call(d3.axisLeft(yScale));
    }
</script>

<div class="container">
    <svg viewBox="0 0 {width} {height}">
        <!-- Chart title -->
        <text
            x={margin.left + innerWidth / 2}
            y={margin.top / 2 + 2}
            text-anchor="middle"
            class="chart-title">
            {title}
        </text>

        <!-- x-axis at bottom of chart area -->
        <g transform="translate({margin.left}, {margin.top + innerHeight})"
           bind:this={xAxis} />

        <!-- y-axis at left of chart area -->
        <g transform="translate({margin.left}, {margin.top})"
           bind:this={yAxis} />

        <g transform="translate({margin.left}, {margin.top})">
            {#each data as d}
                <rect
                    x={0}
                    y={yScale(d.label)}
                    width={xScale(d.value)}
                    height={yScale.bandwidth()}
                    fill={colorScale(d.label)}
                />
            {/each}

            {#if maxBar}
                <!-- highlight outline around the longest bar -->
                <rect
                    x={0}
                    y={yScale(maxBar.label)}
                    width={xScale(maxBar.value)}
                    height={yScale.bandwidth()}
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                />
                <!-- annotation text -->
                <text
                    x={xScale(maxBar.value) + 8}
                    y={yScale(maxBar.label) + yScale.bandwidth() / 2}
                    text-anchor="start"
                    dominant-baseline="middle"
                    class="annotation">
                    Most lines of code
                </text>
            {/if}

            <!-- x-axis label -->
            <text
                x={innerWidth / 2}
                y={innerHeight + margin.bottom - 16}
                text-anchor="middle"
                class="axis-label">
                <tspan x={innerWidth / 2} dy="0">Lines of</tspan>
                <tspan x={innerWidth / 2} dy="0.95em">Code</tspan>
            </text>

            <!-- y-axis label -->
            <text
                x={-(innerHeight / 2)}
                y={-margin.left + 16}
                text-anchor="middle"
                transform="rotate(-90)"
                class="axis-label">
                Language
            </text>
        </g>
    </svg>

    <ul class="legend">
        {#each data as d}
            <li style="--color: {colorScale(d.label)}">
                <span class="swatch"></span>
                {d.label} <em>({d.value.toLocaleString()})</em>
            </li>
        {/each}
    </ul>
</div>

<style>
    svg {
        max-width: 100%;
        height: auto;
        overflow: visible;
    }

    .container {
        display: flex;
        align-items: flex-start;
        gap: 0.75rem;
    }

    .legend {
        list-style: none;
        padding: 0;
        margin: 0;
        display: flex;
        flex-direction: column;
        gap: 0.4rem;
        font-size: 0.8rem;
    }

    .legend li {
        display: flex;
        align-items: center;
        gap: 0.5rem;
    }

    .swatch {
        display: inline-block;
        width: 12px;
        height: 12px;
        border-radius: 2px;
        background-color: var(--color);
        flex-shrink: 0;
    }

    .chart-title {
        font-size: 0.9em;
        font-weight: 600;
        fill: currentColor;
    }

    .axis-label {
        font-size: 0.68em;
        fill: currentColor;
    }

    .annotation {
        font-size: 0.52em;
        fill: black;
        font-style: italic;
    }
</style>
