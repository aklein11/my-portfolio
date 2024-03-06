<slot />

<script>
    let pages = [
        {url: "./", title: "Home"},
        {url: "./projects", title: "Projects"},
        // add the rest of your pages here
    ];
</script>

<nav>
	{#each pages as p }
		<!-- Link here, using {p.url} for the URL and {Home.title} -->
        <!-- Link here, using {p.url} for the URL and {Projects.title} -->
        <!-- Link here, using {p.url} for the URL and {Contact.title} -->
        <!-- Link here, using {p.url} for the URL and {Resume.title} -->
	{/each}
</nav>