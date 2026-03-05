<svelte:head>
  <title>Contact me</title>
</svelte:head>

<h1>Contact</h1>

    
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
