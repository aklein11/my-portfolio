<script>
    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
    export let hLevel = 2;
</script>

<svelte:head>
	<title>Projects</title>
</svelte:head>


<h1>MIT</h1>
<p>Robot in 2.S007, Labs in 6.009, Paper in 9.66</p>
<h1>Internships</h1>
<p>Google in Cambridge, Google in NY, Amazon in Seattle</p>

<h1>Other { projects.length }</h1>
<div class="projects">
    {#each projects as p}
        <Project info={p} />
    {/each}

</div>
