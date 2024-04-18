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
    let label; 
    function toggleWedge (index, event) {
        if (!event.key || event.key === "Enter") {
            selectedIndex = (selectedIndex === index ? -1 : index);
        }
        label = Object.keys(wedges)[index];
    }

    let pieData, arcData, arcs;
    let sliceGenerator = d3.pie().value(d => d.value).sort(null);
    let wedges = {};
    let oldData = [];
    
    export let transitionDuration = 300;

    $: {
        oldData = pieData;
        // set up copy of data
        pieData = data.map(d => ({...d})); 
        // modify copy
        pieData = d3.sort(pieData, d => d.label);
        arcData = sliceGenerator(pieData);
        arcs = arcData.map(d => arcGenerator(d));

        pieData = pieData.map((d, i) => ({...d, ...arcData[i], arc: arcs[i]}));
        transitionArcs()
    }

    function sameArc(d_old, d) {
        if (!d_old || !d) {
            return !d_old && !d; 
        }
        return d_old.startAngle === d.startAngle && d_old.endAngle === d.endAngle; // of if they have the same start and end angles
    }

    function transitionArc (wedge, label) {
        label ??= Object.entries(wedges).find(([label, w]) => w === wedge)[0];
        wedge = this;
        let d = pieData.find(d => d.label === label);
        let d_old = oldData.find(d => d.label === label);
        if (!d || !d_old) {
            return;
        }
        if (sameArc(d_old, d)) {
            return null;
        }
        // Calculations that will only be done once per element go here
        // Always clone objects first, see note in https://d3js.org/d3-interpolate/value#interpolateObject
        let from = d_old ? {...d_old} : getEmptyArc(label, oldData);
        let to = d ? {...d} : getEmptyArc(label);
        let angleInterpolator = d3.interpolate(from, to);
        let interpolator = t => `path("${ arcGenerator(angleInterpolator(t)) }")`;
        let type = d ? d_old ? "update" : "in" : "out";
        return {d, d_old, from, to, interpolator, type};
    }

    function arc (wedge) {
        // Calculations that will only be done once per element go here
        let transition = transitionArc(wedge);
        if (!transition) {
            return;
        }
        return {
            duration: transitionDuration,
            css: (t, u) => {
                const easedT = d3.easeCubic(t);
                return `d: ${transition.interpolator(transition.type === "out" ? u : easedT)}`;
            }
        }
    }

    function getEmptyArc (label, data = pieData) {
        // Union of old and new labels in the order they appear
        let labels = d3.sort(new Set([...oldData, ...pieData].map(d => d.label)));
        let labelIndex = labels.indexOf(label);
        let sibling;
        for (let i = labelIndex - 1; i >= 0; i--) {
            sibling = data.find(d => d.label === labels[i]);
            if (sibling) {
                break;
            }
        }
        let angle = sibling?.endAngle ?? 0;
        return {startAngle: angle, endAngle: angle};
    }

    function transitionArcs() {
        let wedgeElements = Object.values(wedges);
        d3.selectAll(wedgeElements).transition("arc")
            .duration(transitionDuration)
            .styleTween("d", function (_, index) {
                let wedge = this;
                let label = Object.keys(wedges)[index];
                let transition = transitionArc(wedge, label);
                return transition?.interpolator;
            });
    }

</script>


<div class="container">
    <svg viewBox="-50 -50 100 100">
        {#each pieData as d, i (d.label)}
            <path d={d.arc} fill={colors(d.id ?? d.label)} 
                bind:this={ wedges[d.label] }
                class:selected={selectedIndex === i}
                on:click={e => toggleWedge(i, e)} 
                on:keyup={e => toggleWedge(i, e)}
                transition:arc
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

<!-- <input type=number bind:value={transitionDuration}> -->

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
        transition-property: transform, opacity, fill;
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