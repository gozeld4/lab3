<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import {
    computePosition,
    autoPlacement,
    offset
  } from '@floating-ui/dom';
  import BarHorizontal from '$lib/BarHorizontal.svelte';
  import LineChart from '$lib/LineChart.svelte';

  let locData = [];
  let commits = [];
  let linesByDate = [];
  let width = 1000;
  let height = 600;
  let margin = { top: 20, right: 20, bottom: 30, left: 40 };
  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
  };
  usableArea.width = usableArea.right - usableArea.left;
  usableArea.height = usableArea.bottom - usableArea.top;
  let svg;
  let xAxis, yAxis, yAxisGridlines;
  let hoveredIndex = -1;
  let hoveredCommit = {};
  let clickedCommits = [];
  let commitTooltip;
  let tooltipPosition = { x: 0, y: 0 };

  $: brushSelection = null;

  function brushed(evt) {
    brushSelection = evt.selection;
  }

  function isCommitBrushed(commit) {
    if (!brushSelection) {
      return false;
    }
    let [[x0, y0], [x1, y1]] = brushSelection;
    let x = xScale(commit.datetime);
    let y = yScale(commit.hourFrac);
    return x >= x0 && x <= x1 && y >= y0 && y <= y1;
  }

  async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;

    if (evt.type === 'mouseenter') {
      hoveredIndex = index;
      tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
        strategy: 'fixed',
        middleware: [
          offset(5),
          autoPlacement()
        ]
      });
    } else if (evt.type === 'mouseleave') {
      hoveredIndex = -1;
    } else if (evt.type === 'click') {
      let commit = commits[index];

      if (!clickedCommits.includes(commit)) {
        clickedCommits = [...clickedCommits, commit];
      } else {
        clickedCommits = clickedCommits.filter(c => c !== commit);
      }

      console.log(clickedCommits);
    }
  }

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
      ...row,
      line: Number(row.line),
      depth: Number(row.depth),
      length: Number(row.length),
      date: new Date(row.date + 'T00:00' + row.timezone),
      datetime: new Date(row.datetime)
    }));

    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
      let first = lines[0];
      let { author, date, time, timezone, datetime } = first;

      return {
        id: commit,
        url: 'https://github.com/vis-society/lab-7/commit/' + commit,
        author,
        date,
        time,
        timezone,
        datetime,
        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
        totalLines: lines.length,
        lines
      };
    });
    commits = d3.sort(commits, d => -d.totalLines);

    console.log(commits);
  });

  $: {
    let rolled = d3.rollups(locData, v => v.length, d => d3.timeDay.floor(d.datetime))
      .map(([date, count]) => ({ date, count }));
    let [minDate, maxDate] = d3.extent(rolled, d => d.date);
    let allDays = minDate && maxDate ? d3.timeDays(minDate, d3.timeDay.offset(maxDate, 1)) : [];
    linesByDate = allDays.map(date => ({
      date,
      count: rolled.find(d => d.date.getTime() === date.getTime())?.count ?? 0
    }));
  }

  $: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];
  $: selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));
  $: selectedLines = (selectedCommits.length > 0 ? selectedCommits : commits).flatMap(d => d.lines);
  $: selectedCounts = d3.rollup(
    selectedLines,
    v => v.length,
    d => d.type
  );
  $: allTypes = Array.from(new Set(locData.map(d => d.type)));
  $: barData = allTypes
    .map(type => ({ label: String(type), value: selectedCounts.get(type) || 0 }))
    .sort((a, b) => b.value - a.value);
  $: minDate = d3.min(commits, d => d.datetime);
  $: maxDate = d3.max(commits, d => d.datetime);
  $: maxDatePlusOne = maxDate ? d3.timeDay.offset(maxDate, 1) : null;
  $: linesExtent = d3.extent(commits, d => d.totalLines);
  $: xScale = d3.scaleTime()
    .domain(minDate && maxDatePlusOne ? [minDate, maxDatePlusOne] : [new Date(), new Date()])
    .range([usableArea.left, usableArea.right])
    .nice();
  $: yScale = d3.scaleLinear()
    .domain([0, 24])
    .range([usableArea.bottom, usableArea.top]);
  $: rScale = d3.scaleSqrt()
    .domain(linesExtent[0] != null && linesExtent[1] != null ? linesExtent : [0, 1])
    .range([5, 30]);
  $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};
  $: {
    d3.select(svg).call(d3.brush()
      .extent([[usableArea.left, usableArea.top], [usableArea.right, usableArea.bottom]])
      .on("start brush end", brushed));
    d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
  }

  $: if (xAxis && yAxis && yAxisGridlines) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxisGridlines).call(
      d3.axisLeft(yScale)
        .tickFormat(() => '')
        .tickSize(-usableArea.width)
    );
    d3.select(yAxis).call(
      d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, '0') + ':00')
    );
  }
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>

<section>
  <h2>Commits by Time of Day</h2>
  <svg
    viewBox={`0 0 ${width} ${height}`}
    style="max-width: 100%; height: auto; overflow: visible;"
    bind:this={svg}
  >
    <g
      class="gridlines"
      transform={`translate(${usableArea.left}, 0)`}
      bind:this={yAxisGridlines}
    />
    <g transform={`translate(0, ${usableArea.bottom})`} bind:this={xAxis} />
    <g transform={`translate(${usableArea.left}, 0)`} bind:this={yAxis} />
    {#each commits as commit, index}
      <circle
        class:selected={selectedCommits.includes(commit)}
        on:mouseenter={evt => dotInteraction(index, evt)}
        on:mouseleave={evt => dotInteraction(index, evt)}
        on:click={evt => dotInteraction(index, evt)}
        on:keydown={evt => {
          if (evt.key === 'Enter' || evt.key === ' ') {
            evt.preventDefault();
            dotInteraction(index, { ...evt, type: 'click' });
          }
        }}
        role="button"
        tabindex="0"
        aria-pressed={selectedCommits.includes(commit)}
        aria-label={`Commit ${commit.id} by ${commit.author}, ${commit.totalLines} edited lines`}
        cx={xScale(commit.datetime)}
        cy={yScale(commit.hourFrac)}
        r={rScale(commit.totalLines)}
        fill="steelblue"
        fill-opacity="0.65"
      />
    {/each}
  </svg>

  <dl
    class="info tooltip"
    bind:this={commitTooltip}
    hidden={hoveredIndex === -1}
    style={`top: ${tooltipPosition.y}px; left: ${tooltipPosition.x}px;`}
  >
    <dt>Commit</dt>
    <dd><a href={hoveredCommit.url} target="_blank" rel="noreferrer">{hoveredCommit.id}</a></dd>

    <dt>Date</dt>
    <dd>{hoveredCommit.datetime?.toLocaleString('en', { dateStyle: 'full' })}</dd>

    <dt>Time</dt>
    <dd>{hoveredCommit.datetime?.toLocaleString('en', { timeStyle: 'short' })}</dd>

    <dt>Author</dt>
    <dd>{hoveredCommit.author}</dd>

    <dt>Lines edited</dt>
    <dd>{hoveredCommit.totalLines}</dd>
  </dl>
</section>

<p>{selectedCommits.length || 'No'} commits selected</p>

<section>
  <h2>Lines of Code by Language</h2>
  <BarHorizontal
    data={barData}
    title={selectedCommits.length > 0 ? 'Selected Commits Breakdown' : 'Website Breakdown'}
  />
</section>

<LineChart data={linesByDate} />

<style>
  circle {
    transition: 200ms;
  }

  circle:hover {
    fill: darkgreen;
  }

  .selected {
    fill: var(--color-accent);
  }

  dl.info {
    display: grid;
    grid-template-columns: max-content 1fr;
    gap: 0.25rem 0.75rem;
    margin: 0;
    transition-duration: 500ms;
    transition-property: opacity, visibility;
  }

  dl.info[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
  }

  dl.info dt,
  dl.info dd {
    margin: 0;
  }

  dl.info dt {
    color: #64748b;
    font-weight: 500;
  }

  .tooltip {
    position: fixed;
    top: 1em;
    left: 1em;
    padding: 0.75rem 1rem;
    background-color: oklch(100% 0 0 / 0.8);
    border-radius: 0.5rem;
    box-shadow: 0 8px 24px rgb(15 23 42 / 0.16);
    backdrop-filter: blur(8px);
    z-index: 1;
  }

  .gridlines :global(line) {
    stroke: #cbd5e1;
    stroke-opacity: 0.5;
  }

  .gridlines :global(path) {
    display: none;
  }

  @keyframes marching-ants {
    to {
      stroke-dashoffset: -8; /* 5 + 3 */
    }
  }

  svg :global(.selection) {
    fill-opacity: 10%;
    stroke: black;
    stroke-opacity: 70%;
    stroke-dasharray: 5 3;
    animation: marching-ants 2s linear infinite;
  }
</style>

