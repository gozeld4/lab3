<script>
    import projects from '$lib/projects.json';
    import Project from '$lib/Project.svelte';
    import reading from '$lib/reading.json';
    import ReadingItem from '$lib/ReadingItem.svelte';
    import { onMount } from "svelte";

let githubData = null; // This will eventually hold our Github stats
let loading = true; // This will be true *until* the fetch's promise resolves to a value
let error = null; // If the API call resulted in an error, it will go into this variable



let retrieveGithubData = async () => {
    try {
        let response = await fetch("https://api.github.com/users/gozeld4");
        githubData = await response.json();
    } catch (err) {
        error = err;
    }
    loading = false;
}

    onMount(retrieveGithubData);
</script>

<h1>Gozel Dovranova</h1>


<p>I am senior majoring in Chemical Engineering and AI. I am passionate about AI for Science.</p>
<img src="images/1.jpg" alt="This is me">



{#if loading}
    <p>Loading...</p>
{:else if error}
    <p>Something went wrong: {error.message}</p>
{:else}
    <section>
        <h2>My GitHub Stats</h2>
        <dl>
            <dt>Followers</dt>
            <dd>{githubData.followers}</dd>
            <dt>Following</dt>
            <dd>{githubData.following}</dd>
            <dt>Public Repositories</dt>
            <dd>{githubData.public_repos}</dd>
        </dl>
    </section>
{/if}



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

<style>
dt {
        grid-row: 2;
        color: oklch(60% 5% 200);
        text-transform: uppercase;
        font-size: 0.75em;
        letter-spacing: 0.05em;
    }
    dl {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        border: 1px solid oklch(50% 10% 200 / 40%);
        border-radius: 0.5rem;
        padding: 1rem;
        gap: 0.5rem;
        text-align: center;
    }

    

    dd {
        grid-row: 1;
        font-size: 2.5em;

        margin: 0;
    }
</style>