> Development env:
>
> - MacOS 14.x
> - NodeJS 20.x

## Let’s Get Started

### Step 1: Set Up Your Project

First, let’s create a little home for our script. Open your terminal and run:

```bash
mkdir doc-script
cd doc-script
```

This makes a new directory called doc-script and jumps you inside it.

### Step 2: Kick Off an npm Package

Since we’re using Node.js, we’ll set this up as an npm package—it’s the slick way to manage scripts like this. Run:

```bash
npm init -y
```

The -y flag skips all the questions and gives you a basic package.json file. Easy peasy.

### Step 3: Write Your Script

Now, let’s create the actual script. Make a file called doc.js and pop this inside:

```bash
#!/usr/bin/env node

console.log("Welcome to my doc script!");
```

That `#!/usr/bin/env` node line at the top? It’s called a shebang, and it tells your system to run this file with Node.js. The rest just prints a friendly message—feel free to tweak it later if you want your script to do more.

### Step 4: Hook It Up in package.json

We need to tell npm that doc.js is a command we want to use. Open package.json and add this bin field right after the "version" line (or wherever, just inside the main object):

```json
"bin": {
  "doc": "./doc.js"
}
```

Your package.json should now look something like this:

```json
{
  "name": "doc-script",
  "version": "1.0.0",
  "bin": {
    "doc": "./doc.js"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

This bin part says, “Hey, when someone types doc, run doc.js for me.”

### Step 5: Make It Global

Here’s the magic step. In your doc-script directory, run:

```bash
npm link
```

This links your project to your global Node.js setup. It creates a command called doc that you can use anywhere. npm handles all the behind-the-scenes stuff, like sticking a wrapper in /usr/local/bin (or wherever your global bin directory is).

### Step 6: Test It Out

Open a new terminal window (or just keep using the same one) and type:

```bash
doc
```

You should see:
`Welcome to my doc script!`

## Extra Tips

Editing on the Fly: Since we used npm link, any changes you make to doc.js will show up right away when you run doc. No need to re-link.Use `npm unlink` to remove it from your global setup.

Adding More: Want your script to do fancy stuff, like handle arguments? You can use process.argv in doc.js to grab anything you type after doc. For now, though, we’re keeping it simple.

## More

- See my local `docx` cmd as an example.
