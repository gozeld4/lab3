<script>
    export let data = {};
</script>

<article>
    <h2>{data.title}</h2>
    <img src={data.image} alt={data.title}>
    <p>{data.description}</p>
</article>

<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
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