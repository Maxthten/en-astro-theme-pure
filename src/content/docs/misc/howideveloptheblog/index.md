---
title: "Dev Log: Building a Bilingual Blog from Scratch"
publishDate: 2025-12-22 22:41:00
description: "Ramblings && The Hardships of Blogging"
tags: ["Astro", "Cloudflare", "Pitfalls", "Tinkering"]
language: "English"
---

If you are reading this article, it means I succeeded. 🎉

Looking at this smooth-running blog with its seamless English-Chinese switching, you might not imagine that just a few hours ago, I was staring at a screen full of error logs, doubting my life choices.

This article isn't about deep tech; it's purely a record of how I modded the **Astro Pure** theme into a **bilingual (Chinese/English) independent site deployed on Cloudflare**. If you want to build a similar bilingual blog, read on.

## 1. Architectural Vision: Splitting in Two

From the start, I decided not to mess with complex internationalization routing (i18n routing); it's too troublesome. My requirement was simple and crude:

- **Chinese Site**: `zh.maxtonniu.com`
- **English Site**: `en.maxtonniu.com`
- **Deployment**: Cloudflare Pages (Free, fast, and no ICP filing required)

So I created two GitHub repositories corresponding to the two sites. Sounds perfect, right? That's when the nightmare began.

## 2. Pitfall #1: Isomorphic Route Jumping (Magic Switch)

With two domains, the biggest problem was: **If I'm looking at the `/about` page on the Chinese site and click the "English" button, how do I automatically jump to `/about` on the English site instead of stupidly jumping back to the English homepage?**

Since the Astro Pure theme's Header component is hidden deep (or rather, to avoid breaking the source code), I didn't want to modify the component code.

**Solution: The Secret Signal Interception Method**

I filled the switch button's link in `site.config.ts` with a "secret signal": `#switch-lang`.

Then, I injected a bit of magic script at the bottom of the global layout file `BaseLayout.astro`:

```javascript
// Script logic: find the secret signal link, automatically append current path
const targetDomain = '[https://en.maxtonniu.com](https://en.maxtonniu.com)'; // Fill this with the English site URL

document.addEventListener('DOMContentLoaded', () => {
  const langBtn = document.querySelector('a[href="#switch-lang"]');
  if (langBtn) {
    // Automatically replace #switch-lang with [https://en.maxtonniu.com/current_path](https://en.maxtonniu.com/current_path)
    langBtn.href = targetDomain + window.location.pathname;
  }
});


```

This way, no matter which page I'm on, clicking the button allows for a precise traversal to the parallel universe.

## 3. Pitfall #2: Mode Switching

Solved the jumping, but here comes a new problem: Browser `localStorage` is isolated by domain.

I turned on "Dark Mode" on the Chinese site, jumped to the English site, and—blinded. The English site was still in the default "Light Mode". This experience was too disjointed.

**Solution: URL Parameter Relay**

I modified the script above to secretly tag a parameter onto the URL tail during the jump: `?sync_theme=dark`.

Then, at the very earliest stage of page load (inside the `<head>` tag), I wrote a script to "catch the ball":

```javascript
// Check if URL has theme parameter
const params = new URLSearchParams(window.location.search);
const theme = params.get('sync_theme');

if (theme) {
  // Force write to local storage
  localStorage.setItem('theme', theme);
  // Immediately add dark class to html tag to prevent flashing
  document.documentElement.classList.add('dark');
}
```

Now, the theme passes between the two domains like a baton, silky smooth.

## 4.Pitfall #3: "Ghost Files"

Running `npm run dev` locally was fine, but pushing to Cloudflare resulted in an error:

> `[vite]: Rollup failed to resolve import "@/assets/tools/zotero.svg?raw"`

The reason was that I deleted the theme's built-in example image `zotero.svg`, but forgot to delete the reference in the code. **Lesson**: The local development environment is "lazy loaded", so no error doesn't mean no problem; online builds do a "carpet search" and tolerate no grit in their eyes. If you delete a resource, remember to delete the reference!

5. Final Boss: The Grey 404

---------------------------

After going through eighty-one trials, it finally showed `Success: Uploaded`. I excitedly opened the URL, and the result:

> **404 Not Found** (Astro's default grey page)

But I definitely uploaded the code! Why is the homepage empty?

After troubleshooting for ages, I found a dazzling line in the console logs: `[build] adapter: @astrojs/vercel`

**Case Closed!** The configuration code I copied actually still contained `adapter: vercel()`. This is equivalent to packaging the code into a **Vercel-specific** format (Serverless Functions) and then forcing it onto **Cloudflare Pages**. Cloudflare couldn't understand the code, couldn't find `index.html`, and could only shrug.

**Ultimate Fix:** Modify `astro.config.mjs`, delete the Vercel adapter, and return to purity:

```javascript
export default defineConfig({
  // Delete adapter: vercel()
  output: 'static', // Tell Astro I want a pure static webpage!
  // ...
});
```

Pushed to GitHub, Cloudflare rebuilt, and the green `Success` lit up again. This time, the page appeared.

> The above was lovingly written by Gemini on my behalf.
