<script>
    import projects from '$lib/projects.json';
    import Project from '$lib/Project.svelte';
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<h1>{ projects.length } Projects</h1>
<nav>
  <a href=".">Home</a>
  <a class="current" href="projects">Projects</a>
  <a href="contact">Contact</a>
  <a href="resume">Resume</a>
  <a href="https://github.com/gozeld4" target="_blank">GitHub</a>
</nav>
    
    <div class="projects">
        {#each projects as project}
        <Project data={project} />
        {/each}
    </div>
    

    <p></p>
