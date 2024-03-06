<script>
    import { page } from '$app/stores';
    
    let pages = [
        { url: "./", title: "Home" },
        { url: "./projects", title: "Projects" },
        { url: "contact", title: "Contact" },
        { url: "resume", title: "Resume" },
        // add the rest of your pages here
    ];
</script>

<nav>
    {#each pages as p}
    <a href={ p.url } class:current={ "." + $page.route.id === p.url }>{ p.title }</a>
    {/each}
</nav>

<slot />
