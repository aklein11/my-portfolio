<script>

    import * as d3 from "d3";
    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';


    let data = [];
    export let commits = [];
    let width = 1000, height = 600;
    const margin = { top: 20, right: 20, bottom: 20, left: 40 };

    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;

    // Extract the minimum and maximum dates from your data
    let dateExtent;
    $: dateExtent = d3.extent(commits, d => d.datetime);

    $: xScale = d3.scaleTime()
        .domain(dateExtent)
        .range([usableArea.left, usableArea.right])
        .nice(); // Extends the domain to nice values
    
    $: yScale = d3.scaleLinear()
        .domain([0, 24])
        .range([usableArea.bottom, usableArea.top])

    let xAxis, yAxis, yAxisGridlines;
    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
        
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
            .tickFormat("")
            .tickSize(-usableArea.width)
        );
    }

    

    let loc = 0;

    let hoveredIndex = -1;
    $: hoveredCommit = commits[hoveredIndex] ?? {};
    let commitTooltip;
    let showTooltip = false;
    let tooltipPosition = {x: 0, y: 0};

    // Function to compute tooltip position
    async function computeTooltipPosition(hoveredDot) {
        return await computePosition(hoveredDot, commitTooltip, {
            strategy: "fixed", 
            middleware: [
                offset(5), 
                autoPlacement()
            ],
        });
    }


    async function dotInteraction (index, evt) {
        const hoveredDot = evt.target;
        const eventType = evt.type;

        if (eventType === "mouseenter" || eventType === "focus") {
            // dot hovered
            tooltipPosition = await computeTooltipPosition(hoveredDot);
            hoveredIndex = index;
            showTooltip = true;
        }
        else if (eventType === "mouseleave" || eventType === "blur") {
            // dot unhovered
            // hoveredIndex = -1;
            showTooltip = false;
        }
        else if (eventType === "click" || (eventType === "keyup" && evt.key === "Enter")) {
            selectedCommits = [commits[index]]
        }
    }

    
    function brushed (evt) {
        let brushSelection = evt.selection;
        selectedCommits = !brushSelection ? [] : commits.filter(commit => {
            let min = {x: brushSelection[0][0], y: brushSelection[0][1]};
            let max = {x: brushSelection[1][0], y: brushSelection[1][1]};
            let x = xScale(commit.date) + usableArea.left;
            let y = yScale(commit.hourFrac);
            return (x >= min.x) && (x <= max.x) && (y >= min.y) && (y <= max.y);
        })
    }

    let svg;
    $: {
        d3.select(svg).call(d3.brush().on("start brush end", brushed));
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    
    function isCommitSelected (commit) {
        return selectedCommits.includes(commit);
    }

    let hasSelection, selectedLines, languageBreakdown;
    export let selectedCommits = [];
    $: hasSelection = selectedCommits.length > 0;
    $: selectedLines = (hasSelection ? selectedCommits : commits).flatMap(d => d.lines);
    $: languageBreakdown = d3.rollups(selectedLines, v => d3.max(v, v => v.line), d => d.type);

    

</script>



<h3>Commits by time of day</h3>

<svg viewBox="0 0 {width} {height}" bind:this={svg}>
	<!-- scatterplot will go here -->
    <g class="dots">
        {#each commits as commit, index (commit.id) }
            <circle
                cx={ xScale(commit.datetime) }
                cy={ yScale(commit.hourFrac) }
                r="15"
                fill="steelblue"
                tabindex={0}
                aria-describedby="commit-tooltip"
                role="tooltip"
                aria-haspopup="true"
                on:mouseenter={evt => dotInteraction(index, evt)}
                on:mouseleave={evt => dotInteraction(index, evt)}
                on:focus={evt => dotInteraction(index, evt)} 
                on:blur={evt => dotInteraction(index, evt)}
                on:click={evt => dotInteraction(index, evt)}
                on:keyup={evt => dotInteraction(index, evt)}
                class:selected={isCommitSelected(commit)}
            />
        {/each}
    </g>

    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />

</svg>


<dl id="commit-tooltip" class="info tooltip" hidden={!showTooltip} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"  bind:this={commitTooltip}>
    <dt>Commit</dt>
    <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id } </a></dd>

    <dt>Date</dt>
    <dd>{ hoveredCommit.datetime?.toLocaleString("en", {date: "full"}) }</dd>

    <dt>Time</dt>
    <dd>{ hoveredCommit.datetime?.toLocaleString("en", {time: "short"}) }</dd>

    <dt>Author</dt>
    <dd>{ hoveredCommit.author }</dd>

    <dt>Lines Edited</dt>
    <dd>{ hoveredCommit.totalLines }</dd> 
</dl>

<p>{hasSelection ? selectedCommits.length : "No"} commits selected</p>



<style>
	svg {
		overflow: visible;
	}

    .gridlines {
        stroke-opacity: .2;
    }

    dl.info {
        display: grid;
        grid-template-columns: repeat(auto-fill);
        gap: 0; /* Remove default gap */
        
        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }

    dl.info dt {
        font-weight: bold; /* Make labels bold */
        color: #4a0000; /* Make labels less prominent */
    }

    dl.info dd {
        font-weight: bold; /* Make labels bold */
        color: #001a4a; /* Make labels less prominent */
    }

    .tooltip {
        position: fixed;
        right: 1em; /* Adjust as needed */
        z-index: 1000; /* Ensure tooltip appears above other content */
        background-color: #9d9d9dda;
        border: 1px solid #ccc;
        padding: 0.5em;
        border-radius: 4px;
        box-shadow: 0 2px 4px rgba(213, 213, 213, 0.555);
        backdrop-filter: blur(4px);
        max-width: max-content; /* Adjust as needed */
        top: 0;
        left: 0;
        overflow-wrap: break-word; /* Allow word wrapping */
        /* color: #001a4a; */
    }

    circle {
        transition: 200ms;
        transform-origin: center;
        transform-box: fill-box;

        &:hover {
            transform: scale(1.5);
        }

        @starting-style {
            r: 0;
        }
    }

    .selected {
        fill: orange; 
    }

</style>
