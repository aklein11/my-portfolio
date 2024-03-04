<svelte:head>
	<title>Contact me</title>
</svelte:head>

<h1>Email</h1>
<p>aklein11@mit.edu</p>
<form action="mailto:aklein11@mit.edu" method="POST" enctype="text/plain">
    Email: <input name="email" type="email" placeholder="put email here">
    Subject: <input name="subject" type="text">
    Body: <textarea name="body"></textarea>
    <button>Submit</button>
</form>
