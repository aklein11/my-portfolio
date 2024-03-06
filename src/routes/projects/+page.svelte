<script>
    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
    export let hLevel = 2;
</script>

<svelte:head>
	<title>Projects</title>
</svelte:head>


<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
    }

    h1, h2, h3, h4, h5 h6{
        line-height: 1.1;
        text-wrap: balance;
    }

    h2 {
        margin: 0;
        font-size: 120%;
    }   
</style>


<h1>MIT</h1>
<p>Robot in 2.S007, Labs in 6.009, Paper in 9.66</p>
<h1>Internships</h1>
<p>Google in Cambridge, Google in NY, Amazon in Seattle</p>

<h1>Other { projects.length }</h1>
<div class="projects">
    {#each projects as p}

    <Project info={p} />
        <!-- <article>
            <svelte:element this={"h" + hLevel}>{ p.title }</svelte:element>
            <img src={p.image} alt="">
            <p>{p.description}</p>
        </article> -->
    {/each}


    <!-- <article>
        <h2>Vero debitis sapiente voluptate!</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Quis non, modi recusandae optio nesciunt ipsa qui eaque adipisci cumque fugiat veritatis! Sint vitae quisquam eos tempore placeat, officia eveniet, repudiandae corporis reiciendis iste laudantium distinctio quo. Dolorum, voluptate.</p>
    </article>
    <article>
        <h2>Eius voluptas et dolorem.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Quasi iste quod, asperiores voluptatibus architecto earum ducimus sed nulla iusto provident, repellat delectus magni tempore. Molestiae fuga suscipit minus laudantium earum pariatur, ipsum, saepe ipsam ea iste cupiditate dicta.</p>
    </article>
    <article>
        <h2>Deserunt dicta atque nemo.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Unde nemo consequatur quam provident recusandae porro qui exercitationem quis ipsum? Ipsam error laudantium illo nobis reiciendis vitae, eos, incidunt mollitia repellendus quae sunt repellat labore dolores reprehenderit ullam fuga.</p>
    </article>
    <article>
        <h2>Autem ipsa aperiam provident.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Nam laboriosam maxime aliquam eius exercitationem laborum quas! Molestiae corporis natus placeat laboriosam odio reiciendis laudantium corrupti eos itaque facere. Exercitationem quibusdam soluta eveniet, ipsa recusandae autem cum delectus expedita?</p>
    </article>
    <article>
        <h2>Necessitatibus numquam deserunt maxime.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Maxime, explicabo cumque itaque recusandae magni architecto dignissimos velit. Architecto maiores hic recusandae ducimus quo esse autem officiis quae, in, dolorum veniam sequi omnis alias dolorem, culpa numquam aspernatur est.</p>
    </article>
    <article>
        <h2>Cupiditate nesciunt laboriosam illo.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Porro alias ullam repudiandae ducimus exercitationem! Quae consectetur impedit perferendis velit modi odit sequi vel repellat autem? Nesciunt assumenda ex nobis corrupti, similique perferendis? Inventore in officiis quo asperiores vitae.</p>
    </article>
    <article>
        <h2>Illum magni sapiente doloribus!</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Earum quam dicta vel quo exercitationem molestias animi magnam at quis. Velit ad corporis fugit quasi, nobis ipsa id cupiditate nihil at sed magni hic et quo! Non, libero corporis!</p>
    </article>
    <article>
        <h2>Nesciunt ullam quos porro.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Nostrum, esse dolore? Iste maxime esse tempora quisquam praesentium nam voluptas? Atque porro ducimus laborum ipsum qui quisquam! Soluta, quod. Atque rerum dignissimos saepe corrupti iste voluptas expedita eius fuga.</p>
    </article>
    <article>
        <h2>Consequatur repellendus sit incidunt!</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Atque dolores amet eius quo architecto laboriosam corporis quod accusantium sed excepturi, ratione, voluptate labore. Rerum iste, esse, ratione quos suscipit porro beatae voluptates repudiandae, distinctio maxime assumenda explicabo officiis!</p>
    </article>
    <article>
        <h2>Sint excepturi amet ullam.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Architecto sit itaque delectus, temporibus modi, dicta explicabo laboriosam, perferendis illo mollitia recusandae labore eaque vel! Inventore sunt consequuntur quaerat perferendis reprehenderit consequatur blanditiis fuga. Iure, porro. Neque, itaque veritatis.</p>
    </article>
    <article>
        <h2>Numquam, consectetur tenetur. Commodi.</h2>
        <img src="https://vis-society.github.io/labs/2/images/empty.svg" alt="">
        <p>Iste impedit excepturi magnam adipisci maiores blanditiis minus, autem accusamus sequi asperiores alias, ipsa tenetur animi voluptates perferendis temporibus recusandae pariatur nobis! Deserunt saepe tempora facere vero qui eligendi asperiores.</p>
    </article> -->
</div>
