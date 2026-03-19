<script>
    import * as d3 from 'd3';

    let width = 500;
    let height = 300;

    export let data = [];

    // Extra left margin to fit language name labels on the y-axis
    let margin = { top: 40, right: 180, bottom: 50, left: 80 };
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
        d3.select(yAxis).call(d3.axisLeft(yScale));
        d3.select(xAxis).call(
            d3.axisBottom(xScale)
                .tickFormat(d => Number.isInteger(d) ? d : "")
        );
    }
</script>

<div class="container">
    <svg viewBox="0 0 {width} {height}">
        <!-- Chart title -->
        <text
            x={margin.left + innerWidth / 2}
            y={margin.top / 2}
            text-anchor="middle"
            class="chart-title">
            Lines of Code by Language
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
                <!-- leader line from end of bar -->
                <line
                    x1={xScale(maxBar.value)}
                    y1={yScale(maxBar.label) + yScale.bandwidth() / 2}
                    x2={xScale(maxBar.value) + 10}
                    y2={yScale(maxBar.label) + yScale.bandwidth() / 2}
                    stroke="currentColor"
                    stroke-width="1"
                />
                <!-- annotation text -->
                <text
                    x={xScale(maxBar.value) + 15}
                    y={yScale(maxBar.label) + yScale.bandwidth() / 2}
                    dominant-baseline="middle"
                    class="annotation">
                    Most lines of code
                </text>
            {/if}

            <!-- x-axis label -->
            <text
                x={innerWidth / 2}
                y={innerHeight + margin.bottom - 10}
                text-anchor="middle"
                class="axis-label">
                Lines of Code
            </text>

            <!-- y-axis label -->
            <text
                x={-(innerHeight / 2)}
                y={-margin.left + 20}
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
        gap: 1rem;
    }

    .legend {
        list-style: none;
        padding: 0;
        margin: 0;
        display: flex;
        flex-direction: column;
        gap: 0.4rem;
        font-size: 0.9rem;
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
        font-size: 1em;
        font-weight: bold;
        fill: currentColor;
    }

    .axis-label {
        font-size: 0.8em;
        fill: currentColor;
    }

    .annotation {
        font-size: 0.7em;
        fill: black;
        font-style: italic;
    }
</style>
