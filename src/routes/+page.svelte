<script>
    import projects from '$lib/projects.json';
    import Project from '$lib/Project.svelte';
    import reading from '$lib/reading.json';
    import ReadingItem from '$lib/ReadingItem.svelte';
</script>

<h1>Gozel Dovranova</h1>
<nav>
    <a class="current" href=".">Home</a>
    <a href="projects">Projects</a>
    <a href="contact">Contact</a>
    <a href="resume">Resume</a>
    <a href="https://github.com/gozeld4" target="_blank">GitHub</a>
</nav>

<p>I am senior majoring in Chemical Engineering and AI. I am passionate about AI for Science.</p>
<img src="images/1.jpg" alt="This is me">

<h2>Latest Projects</h2>
<div class="projects highlights">
    {#each projects.slice(0, 3) as p}
    <Project data={p} />
    {/each}
</div>

<h2>What I'm Reading</h2>
<div class="reading">
    {#each reading as item}
    <ReadingItem data={item} />
    {/each}
</div>

