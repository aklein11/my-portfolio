<script>
    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<h1>Abigail Klein</h1>

<!-- <nav>
    <a hreft="index.html" padding-bottom>Home</a>
    <a href="contact/index.html">Contact</a>
    <a href="projects/index.html">Projects</a>
    <a href="https://github.com/aklein11/" target="_blank">GitHub Profile</a>
    <a href="resume/index.html">Resume</a>
</nav> -->

<p>My name is Abigail Klein and I am a senior studying Computation and Cognition at MIT. I enjoy coding and am excited to learn more web development and data visualization techniques through this class.</p>

<img src="images/googleBlurbPicture.jpg" alt="portrait of Abigail" width="450"/>

<div class="projects">
    <h2>Latest projects</h2>
    <div class="projects highlights">
        {#each projects.slice(0, 3) as p}
            <Project info={p} hLevel=3 />
            <article>
                <h2>{p.title}</h2>
                <img src={p.image} alt="">
                <p>{p.description}</p>
            </article>
        {/each}
    </div>
</div>