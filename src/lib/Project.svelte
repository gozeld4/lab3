<script>
    export let data = {};
</script>

<article>
    <h2>{data.title}</h2>
    <img src={data.image} alt={data.title}>
    <p class="year">{data.year}</p>
    <p>{data.description}</p>
</article>

<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 4;
    }

    article .year {
        margin: 0;
        font-size: 0.85rem;
        color: #888;
        font-style: italic;
    }
    
    article h2 {
        margin: 0;
    }


    article img {
        width: 100%;
        aspect-ratio: 16 / 9;
        object-fit: cover;
    }
</style>