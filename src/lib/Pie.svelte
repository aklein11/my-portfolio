<script>
    import * as d3 from 'd3';

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);

    export let data = [];
    
    // let pieData = [
    //     { value: 1, label: "apples" },
    //     { value: 2, label: "oranges" },
    //     { value: 3, label: "mangos" },
    //     { value: 4, label: "pears" },
    //     { value: 5, label: "limes" },
    //     { value: 5, label: "cherries" }
    // ];


    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData = sliceGenerator(data);
    let arcs = arcData.map(d => arcGenerator(d));

    let colors = d3.scaleOrdinal(d3.schemeTableau10);

</script>


<style>
    svg {
	max-width: 20em;
	margin-block: 2em;
	overflow: visible; /* Do not clip shapes outside the viewBox */
    }

    .container {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;
        align-items: horizontal;
        gap: 1em;
    }

    .legend {
        list-style: none;
        padding: 1em;
        margin: 1em;
        flex: 1;
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        outline: grey solid 1em;
        margin: auto;
    }

    .ledgend li {
        display: flex;
        align-items: center;
        gap: 1em;
    }

    .swatch {
        width: 1em;
        height: 1em;
        background-color: var(--color);
        display:inline-block;
        margin-right: 5px;
        border-radius: 50%;
        /* aspect-ratio: 1 / 1;  */
    }

</style>


<div class="container">
    <svg viewBox="-50 -50 100 100">
        {#each arcs as arc, i}
            <path d={arc} fill={colors(i)} />
        {/each}
    </svg>

    <ul class="legend">
        {#each data as d, index}
            <li class="legend-item" style="--color: { colors(index) }">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>

