<script>

    import * as d3 from "d3";
    import { onMount } from "svelte";
    import Pie from "$lib/Pie.svelte";
    import CommitScatterplot from "./Scatterplot.svelte";
    import FileLines from "./FileLines.svelte";
    import Scrolly from "svelte-scrolly";


    let colors = d3.scaleOrdinal(d3.schemeTableau10);
    let commits = [];
    let data = []; 
    let loc;

    onMount(async () => {
        data = await d3.csv("loc.csv", row => ({
            ...row,
            line: Number(row.line), // or just +row.line
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));

        loc = data.length
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

        console.log("commits: ", commits)
    });

    $: maxDepth = data.length > 0 ? d3.max(data, d => d.depth) : "no data";
    $: averageDepth = data.length > 0  ? d3.mean(data, d => d.depth).toFixed(1) : "no data";
    $: numberFiles = data.length > 0 ? d3.group(data, d => d.file).size : "no data";

    let selectedCommits = [];
    let hasSelection, selectedLines, languageBreakdown;
    $: hasSelection = selectedCommits.length > 0;
    $: selectedLines = (hasSelection ? selectedCommits : commits).flatMap(d => d.lines);
    $: languageBreakdown = d3.rollups(selectedLines, v => d3.max(v, v => v.line), d => d.type);

    let commitProgress = 100;
    $: timeScale = d3.scaleTime()
        .range([0, 100]) // what you input into the function
        .domain(d3.extent(commits, d => d.datetime)); // what you get out of the function
    $: commitMaxTime = timeScale.invert(commitProgress);

    $: filteredCommits = commits.filter(commit => {
        return commit.datetime <= commitMaxTime;
    })

    $: filteredLines = data.filter(d => {
        return d.datetime <= commitMaxTime;
    })

</script>


<h1>Meta</h1>

<section>
    <h2>GitHub Stats from March</h2>
    <dl class="stats">
        <dt>Total <abbr title="Lines of code">LOC</abbr></dt><dd>{ data.length }</dd>
        <dt>Maximum Depth: </dt><dd>{ maxDepth }</dd>
        <dt>Average Depth: </dt><dd>{ averageDepth }</dd>
        <dt>Number of Files: </dt><dd>{ numberFiles }</dd>
    </dl>
</section>

<!-- <label for="dateSlider"> Show commits until: </label>
<input type="range" id="dateSlider" min={0} max={100} step="1" bind:value={commitProgress}>
<time id="selectedDateTime">{commitMaxTime.toLocaleString('en')}</time> -->

<Scrolly bind:progress={ commitProgress }>
	{#each commits as commit, index }
	<p>
        Before this class I did not know how to use git or Github. Id ddidn't even know that they were different things. 
        Then I worked through the first lab, and began getting familiar with the tools. 
		On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"})},
		I made <a href="{commit.url}" target="_blank">{ index > 0 ? 'another glorious commit' : 'my first commit, and it was glorious' }</a>.
		I edited {commit.totalLines} lines across { d3.rollups(commit.lines, D => D.length, d => d.file).length } files.
		Then I looked over all I had made, and I saw that it was very good.
	</p>
{/each}

	<svelte:fragment slot="viz">
		<CommitScatterplot commits={filteredCommits} bind:selectedCommits={selectedCommits} />
        {#each languageBreakdown as [language, filteredLines] }
            <div>
                <dl class="stats">
                    <dt>
                        {language}
                    </dt>
                    <dd>
                        {filteredLines} lines
                    </dd>
                </dl>
            </div>
        {/each}

        <Pie data={Array.from(languageBreakdown).map(([language, filteredLines]) => ({label: language, value: filteredLines}))} colors={colors}/>

	</svelte:fragment>
</Scrolly>


<Scrolly bind:progress={commitProgress} throttle={100} debounce={300} --scrolly-layout="viz-first" --scrolly-viz-width="1.5fr">
	{#each commits as commit, index }
        <p>
            Before this class I did not know how to use git or Github. Id ddidn't even know that they were different things. 
            Then I worked through the first lab, and began getting familiar with the tools. 
            On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"})},
            I made <a href="{commit.url}" target="_blank">{ index > 0 ? 'another glorious commit' : 'my first commit, and it was glorious' }</a>.
            I edited {commit.totalLines} lines across { d3.rollups(commit.lines, D => D.length, d => d.file).length } files.
            Then I looked over all I had made, and I saw that it was very good.
        </p>
    {/each}

	<svelte:fragment slot="viz">
		<FileLines lines={filteredLines} colors={colors}/>
	</svelte:fragment>
</Scrolly>


<style>

    :global(body) {
        max-width: min(120ch, 80vw);
    }

   .stats {
        display: grid;  
        /* grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); */
        flex-direction: column;  /* Arrange items horizontally */
        justify-content: space-between;  /* Distribute items evenly along the main axis */
        align-items: center;  /* Align items vertically */
    }

    .stats dt {
        margin-right: 10px;  /* Add some margin between dt and dd */
        grid-row: 1;  /* Place dt in the first row */
        /* display: inline-block; */
    }

    .stats dd {
        grid-row: 2;  /* Place dd in the second row */
    }

</style>
