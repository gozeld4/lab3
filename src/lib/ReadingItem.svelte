<script>
    export let data = {};
</script>

<article>
    <img src={data.image} alt={data.title}>
    <h2>{data.title}</h2>
    <p class="author">{data.author}</p>
</article>

<style>
article {
    display: flex;
    flex-direction: row;
    gap: 1em;
    align-items: flex-start;
}

article img {
    width: 4em;
    aspect-ratio: 2 / 3;
    object-fit: cover;
    flex-shrink: 0;
}

article h2 {
    margin: 0;
    font-size: 1rem;
}

.author {
    margin: 0;
    color: oklch(50% 0 0);
    font-style: italic;
}
</style>
