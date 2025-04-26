**Record a Quick 3-Minute Process for Publishing an NPM Package**

### 1. Project Preparation
- Local development, following the guide: [[01. Build a Global CLI Tool with Node.js]].
- Pay special attention to the `package.json` file. Refer to this [package.json](https://github.com/AlucPro/note-tool/blob/main/package.json) as an example:

```JSON
{
  "name": "@alucpro/note-tool",
  "version": "0.0.2",
  "description": "A NodeJS script that deals with markdown notes.",
  "type": "module",
  "main": "src/index.js",
  // Sometimes, the npm homepage might not properly detect the README file, so you need to manually specify it
  "readme": "README.md",
  // Specify that this is a public package
  "publishConfig": {
    "access": "public"
  },
  // If this is a CLI package, specify the bin command and corresponding JS file
  "bin": {
    "note": "src/index.js"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/AlucPro/note-tool.git"
  },
  "keywords": [
    "note",
    "obsidian",
    "flomo",
    "logseq",
    "markdown",
    "roamresearch"
  ],
  "author": "Ming (alucpro)",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/AlucPro/note-tool/issues"
  },
  "homepage": "https://github.com/AlucPro/note-tool#readme"
}
```

### 2. Publishing
```bash
# Log in to npm locally
npm login

# Bump the version number
npm version patch

# Publish as a public package
npm publish --access public

# ------
# Others
# Check if the documentation is recognized correctly
npm docs
```

### Notes and Common Issues:
- If the `name` field in `package.json` is too similar to an existing package, the publication will be rejected. → You can use a scoped package name like `@alucpro/packagename` to avoid conflicts.
- Packages with a scope are private by default, requiring a paid account. Therefore, for free accounts, you must add `--access public` when publishing.
- Sometimes, the npm package page cannot detect the `README.md` file automatically. To fix this, specify the `readme` field explicitly in the `package.json`.
