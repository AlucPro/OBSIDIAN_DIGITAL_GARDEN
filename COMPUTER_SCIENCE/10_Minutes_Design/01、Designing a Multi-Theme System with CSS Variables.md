## Introduction

Supporting light and dark themes enhances user experience, especially when aligned with system preferences like Google Chrome’s dark and light modes. This post explains how to use CSS variables and media queries to design a multi-theme system that switches automatically or via user selection.

## CSS Variables for Theming

CSS variables (custom properties) enable reusable style values. For theming, define variables for UI elements like colors:

```css
:root {
  --background-color: #fff;
  --text-color: #333;
}

body {
  background-color: var(--background-color);
  color: var(--text-color);
}
```

## Defining Themes

Set default variables for the light theme and override them for dark mode using the `prefers-color-scheme` media query:

```css
:root {
  --background-color: #fff;
  --text-color: #333;
}

@media (prefers-color-scheme: dark) {
  :root {
    --background-color: #333;
    --text-color: #fff;
  }
}
```

`@media (prefers-color-scheme: dark) {}` mean that the theme is dark when the user's system settings are dark.

This automatically adjusts the theme based on the user’s system settings.

## Manual Theme Selection

To allow manual overrides, apply classes to the `<html>` element and refine the media query:

```css
:root {
  --background-color: #fff;
  --text-color: #333;
}

@media (prefers-color-scheme: dark) {
  :root:not(.light-theme) {
    --background-color: #333;
    --text-color: #fff;
  }
}

:root.dark-theme {
  --background-color: #333;
  --text-color: #fff;
}

:root.light-theme {
  --background-color: #fff;
  --text-color: #333;
}
```

- **Automatic**: No class; follows system preference via media query.
- **Light**: `.light-theme` class enforces light mode.
- **Dark**: `.dark-theme` class enforces dark mode.

## Switching Themes with JavaScript

Use JavaScript to toggle classes and persist preferences with `localStorage`:

```html
<select id="theme-select">
  <option value="automatic">Automatic</option>
  <option value="light">Light</option>
  <option value="dark">Dark</option>
</select>
```

```javascript
const htmlElement = document.documentElement;
const themeSelect = document.getElementById("theme-select");

function setTheme(theme) {
  if (theme === "automatic") {
    htmlElement.classList.remove("light-theme", "dark-theme");
    localStorage.removeItem("theme");
  } else {
    htmlElement.classList.remove("light-theme", "dark-theme");
    htmlElement.classList.add(`${theme}-theme`);
    localStorage.setItem("theme", theme);
  }
}

// Load saved theme
const savedTheme = localStorage.getItem("theme");
if (savedTheme) {
  setTheme(savedTheme);
  themeSelect.value = savedTheme;
} else {
  themeSelect.value = "automatic";
}

// Handle selection
themeSelect.addEventListener("change", (e) => {
  setTheme(e.target.value);
});
```

## Conclusion

This approach combines CSS variables, media queries, and JavaScript to create a flexible theming system that adapts to Chrome’s modes automatically or respects user choices, ensuring a seamless and maintainable design.
