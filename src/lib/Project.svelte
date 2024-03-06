<script>export let info={}; let p = info;</script>
<article>
    <h2>{p.title}</h2>
    <img src={p.image} alt="">
    <p>{p.description}</p>
</article>

<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
    }

    h1, h2, h3, h4, h5 h6{
        line-height: 1.1;
        text-wrap: balance;
    }

    h2 {
        margin: 0;
        font-size: 120%;
    }   
</style>