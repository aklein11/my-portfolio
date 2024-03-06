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

<style>
    nav {
        --border-color: oklch(50% 10% 200 / 40%);
        
        display: flex;
        margin-bottom: 20px;
        border-bottom-width: 1px;
        border-bottom-style: solid;
        border-bottom-color: oklch(80% 3% 200 / 40%);
    }

    nav a {
        text-decoration: none;
        color: inherit;
        text-align: center;
        padding: .5em;
    }

    nav a:hover {
        border-bottom-width: 1px;
        border-bottom-style: solid;
        border-bottom-color: var(--color-accent);
        /* background-color: oklch(from var(--color-accent) 95% 5% h) */
        background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
    }
</style>

<nav>
    {#each pages as p}
        <a href={ p.url } class:current={ "." + $page.route.id === p.url } target={ p.url.startsWith("http") ? "_blank" : null }>
            { p.title }
        </a>
    {/each}
</nav>

<slot />
