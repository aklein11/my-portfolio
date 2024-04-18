<script>
    import * as d3 from 'd3';

    export let selectedIndex = -1;
    export let data = [];

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    
    // let pieData = [
    //     { value: 1, label: "apples" },
    //     { value: 2, label: "oranges" },
    //     { value: 3, label: "mangos" },
    //     { value: 4, label: "pears" },
    //     { value: 5, label: "limes" },
    //     { value: 5, label: "cherries" }
    // ];

    export let colors = d3.scaleOrdinal(d3.schemeTableau10);

    function toggleWedge (index, event) {
        if (!event.key || event.key === "Enter") {
            selectedIndex = (selectedIndex === index ? -1 : index);
        }
    }

    let pieData, arcData, arcs;
    let sliceGenerator = d3.pie().value(d => d.value);

    $: {
        // set up copy of data
        pieData = data.map(d => ({...d})); 
    };

    $: {
        // modify 
        arcData = sliceGenerator(pieData);
        arcs = arcData.map(d => arcGenerator(d));
        pieData = pieData.map((d, i) => ({...d, ...arcData[i], arc: arcs[i]}));
    }
</script>


<div class="container">
    <svg viewBox="-50 -50 100 100">
        {#each pieData as d, i}
            <path d={d.arc} fill={colors(d.id ?? d.label)} 
                class:selected={selectedIndex === i}
                on:click={e => toggleWedge(i, e)} 
                on:keyup={e => toggleWedge(i, e)}  
                tabindex="0" role="button" aria-label="pie wedge"
                style="--start-angle: { d.startAngle }rad;
	                    --end-angle: { d.endAngle }rad;"/>
        {/each}
    </svg>

    <ul class="legend">
        {#each pieData as d, i}
            <li class="legend-item" style="--color: { colors(d.label) }">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
                <!-- class:selected={selectedIndex === index} -->
            </li>
        {/each}
    </ul>
</div>


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
        outline: grey solid 3px;
        margin: auto;
        max-width: 150px;
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

    svg:has(path:hover, path:focus-visible) {
        path:not(:hover, :focus-visible) {
            opacity: 50%;
        }
    }

    path {
        transform: rotate(var(--mid-angle))
	           translateY(0)
	           rotate(calc(-1 * var(--mid-angle)));

        transition: 300ms;
        outline: none;
        --angle: calc(var(--end-angle) - var(--start-angle));
	    --mid-angle: calc(var(--start-angle) + var(--angle) / 2);

        &.selected {
            transform: rotate(var(--mid-angle))
                    translateY(-6px)
                    rotate(calc(-1 * var(--mid-angle)));
       }


    }

    .selected {
        --color: oklch(60% 45% 0) !important;

        &:is(path) {
            fill: var(--color);
        }
    }

</style>