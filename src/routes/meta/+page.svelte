<script>

    import * as d3 from "d3";
    import { onMount } from "svelte";
    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';
    import Pie from "$lib/Pie.svelte";


    let data = [];
    let commits = [];
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

    console.log('unMounted Data', data )

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
    
    
    onMount(async () => {
        data = await d3.csv("loc.csv", row => ({
            ...row,
            line: Number(row.line), // or just +row.line
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));

        commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/vis-society/lab-7/commit/" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length
            };

            Object.defineProperty(ret, "lines", {
                value: lines,
                configurable: true,
                writable: true,
                enumerable: false,
            });

	        return ret;
        });

        console.log(commits)
    });

    $: maxDepth = data.length > 0 ? d3.max(data, d => d.depth) : "no data";
    $: averageDepth = data.length > 0  ? d3.mean(data, d => d.depth).toFixed(1) : "no data";
    $: numberFiles = data.length > 0 ? d3.group(data, d => d.file).size : "no data";

    let hoveredIndex = -1;
    $: hoveredCommit = commits[hoveredIndex] ?? {};
    let cursor = {x: 0, y: 0};
    let commitTooltip;
    let tooltipPosition = {x: 0, y: 0};

    function dotInteraction (index, evt) {
        if (evt.type === "mouseenter" || evt.type === "focus") {
            // dot hovered
        }
        else if (evt.type === "mouseleave" || evt.type === "blur") {
            // dot unhovered
        }
    }

    function brushed (evt) {
        console.log(evt);
        if (evt.selection) {
            brushSelection = evt.selection;
        } else {
            brushSelection = null;
        }
    }

    let svg;
    $: {
        d3.select(svg).call(d3.brush().on("start brush end", brushed));
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    let brushSelection; 
    function isCommitSelected (commit) {
        if (!brushSelection) {
            return false;
        }
        let min = {x: brushSelection[0][0], y: brushSelection[0][1]};
        let max = {x: brushSelection[1][0], y: brushSelection[1][1]};
        let x = xScale(commit.date);
        let y = yScale(commit.hourFrac);
        return x >= min.x && x <= max.x && y >= min.y && y <= max.y;
    }

    let selectedCommits, hasSelection, selectedLines, languageBreakdown;
    $: selectedCommits = brushSelection ? commits.filter(isCommitSelected) : [];
    $: hasSelection = brushSelection && selectedCommits.length > 0;
    $: selectedLines = (hasSelection ? selectedCommits : commits).flatMap(d => d.lines);
    $: languageBreakdown = d3.rollups(selectedLines, v => d3.max(v, v => v.line), d => d.type);

</script>


<h1>Meta</h1>


<section>
    <h2>My GitHub Stats</h2>
    <main>
        {#await fetch("https://api.github.com/users/aklein11") }
            <p>Loading...</p>
        {:then response}
            {#await response.json()}
                <p>Decoding...</p>
            {:then data}
                <!-- <p>The data is { JSON.stringify(data) }</p> -->
                <dl class="stats">
                    <dt>Total <abbr title="Lines of code">LOC</abbr></dt><dd>{ data.length }</dd>
                    <dt>Followers: </dt><dd>{ data.followers }</dd>
                    <dt>Following: </dt><dd>{ data.following }</dd>
                    <dt>Number Public Repos: </dt><dd>{ data.public_repos }</dd>
                    <dt>Maximum Depth: </dt><dd>{ maxDepth }</dd>
                    <dt>Average Depth: </dt><dd>{ averageDepth }</dd>
                    <dt>Number of Files: </dt><dd>{ numberFiles }</dd>
                </dl>
            {:catch error}
                <p class="error">
                    Something went wrong: {error.message}
                </p>
            {/await}
        {:catch error}
            <p class="error">
                Something went wrong: {error.message}
            </p>
        {/await}
    </main>
</section>

<h3>Commits by time of day</h3>

<svg viewBox="0 0 {width} {height}" bind:this={svg}>
	<!-- scatterplot will go here -->
    <g class="dots">
        {#each commits as commit, index }
            <circle
                cx={ xScale(commit.datetime) }
                cy={ yScale(commit.hourFrac) }
                r="5"
                fill="steelblue"

                on:mouseenter={evt => {
                    hoveredIndex = index;
                    cursor = {x: evt.x, y: evt.y};
                    }
                }
                on:mouseleave={evt => hoveredIndex = -1}

                class:selected={isCommitSelected(commit)}
            />
        {/each}
    </g>

    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />

</svg>


<dl id="commit-tooltip" class="info tooltip" hidden={hoveredIndex === -1} style="top: {cursor.y}px; left: {cursor.x}px"  bind:this={commitTooltip}>
    <dt>Commit</dt>
    <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

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


    {#each languageBreakdown as [language, lines] }
        <div>
            <dt>
                {language}
                {lines} lines {selectedLines.length}
                
            </dt>
        </div>
    {/each}
{hasSelection}


<Pie data={Array.from(languageBreakdown).map(([language, lines]) => ({label: language, value: lines}))} />



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
        color: #ff7676; /* Make labels less prominent */
    }

    .tooltip {
        position: fixed;
        bottom: 1em; 
        right: 1em; /* Adjust as needed */
        z-index: 1000; /* Ensure tooltip appears above other content */
        background-color: #ffffff87;
        border: 1px solid #ccc;
        padding: 0.5em;
        border-radius: 4px;
        box-shadow: 0 2px 4px rgba(213, 213, 213, 0.555);
        backdrop-filter: blur(4px);
        max-width: 200px; /* Adjust as needed */
        max-height: 200px;
        overflow-wrap: break-word; /* Allow word wrapping */
    }

    circle {
        transition: 200ms;
        transform-origin: center;
        transform-box: fill-box;

        &:hover {
            transform: scale(1.5);

        }
    }

    .selected {
        fill: orange; 
    }

</style>
