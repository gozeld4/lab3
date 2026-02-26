<svelte:head>
  <title>Contact me</title>
</svelte:head>

<h1>Contact</h1>
<nav>
  <a href=".">Home</a>
  <a href="projects">Projects</a>
  <a class="current" href="contact">Contact</a>
  <a href="resume">Resume</a>
  <a href="https://github.com/gozeld4" target="_blank">GitHub</a>
</nav>
    
    <form action="mailto:gozeld4@gmail.com" method="GET">
        <label for="email">
            Email:
            <input type="email" name="email" id="email">
        </label>
        
        <label for="subject">
            Subject:
            <input type="text" name="subject" id="subject">
        </label>
        
        <label for="body">
            Message:
            <textarea name="body" id="body"></textarea>
        </label>
        
        <button type="submit">Send</button>
    </form>
