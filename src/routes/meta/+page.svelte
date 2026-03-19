<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';

  let locData = [];

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
      ...row,
      line: Number(row.line),
      length: Number(row.length),
      depth: Number(row.depth)
    }));
  });

  $: locByLanguage = d3.rollups(locData, v => v.length, d => d.type)
    .map(([lang, count]) => ({ label: lang, value: count }))
    .sort((a, b) => b.value - a.value);
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>

<section>
  <h2>Lines of Code by Language</h2>
  <BarHorizontal data={locByLanguage} />
</section>


