<script>
    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<h1>Abigail Klein</h1>

<p>My name is Abigail Klein and I am a senior studying Computation and Cognition at MIT. I enjoy coding and am excited to learn more web development and data visualization techniques through this class.</p>

<img src="images/googleBlurbPicture.jpg" alt="portrait of Abigail" width="450"/>


<h2>Latest projects</h2>
<div class="projects highlights">
    {#each projects.slice(0, 3) as p}
        <Project info={p} hLevel=3 />
        
    {/each}
</div>
