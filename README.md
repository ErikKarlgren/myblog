# myblog

A small personal site. Two files do the work, there is no build step, no
package manager, and no JavaScript.

## The files

| File | What it is | Do you edit it? |
|---|---|---|
| `index.html` | The page: your text, your links | Yes, constantly |
| `style.css` | The whole look: colours, fonts, sizes | Yes, to change the design |
| `README.md` | This file | Rarely |
| `LICENSE` | Which licence covers what | Once, to put your name in |
| `LICENSE-MIT` | Licence text for the code | No |
| `LICENSE-CC-BY-SA-4.0` | Licence text for the writing | No |

That is the entire structure. Nothing is generated, so what you see in these
files is exactly what the browser receives.

## Previewing it

Double-click `index.html`. It works straight off your disk — no server, no
install. Everything uses relative paths (`style.css`, not `/style.css`) so it
behaves the same locally and on a real host.

If you want to see it over HTTP the way a visitor will:

```sh
python3 -m http.server 8000
# then open http://localhost:8000
```

## Making it yours

Every placeholder is tagged with the word `EDIT`. To list them all:

```sh
grep -n "EDIT:" index.html
```

The things to fill in:

- `Erik Karlgren Domercq` — in three places: `<title>`, `<h1>`, and the footer
- the tagline under your name
- the two paragraphs under **About**
- the four bullets under **What I work on**
- the three links under **Links** (delete any you don't use)
- your email under **Contact**

Useful `grep` trick for later: `grep -rn "YOUR" .` finds anything unfilled.

## Changing the look

Open `style.css`. Section 1 is a single block called **design tokens**, and it
is the only place you normally need to touch — everything else refers to those
names, so changing a value there changes it everywhere at once.

| I want to change… | Edit this | Section |
|---|---|---|
| Background, text, link colours (light) | `--bg`, `--text`, `--accent`, … | 1 |
| Colours (dark mode) | the same names | 2 |
| The size of your name | `--h1` | 1 |
| The size of section headings | `--h2` | 1 |
| Body text size | `--text` | 1 |
| How wide the column is | `--measure` | 1 |
| Vertical breathing room | `--gap` | 1 |
| The body font | `--font-sans` | 1 |
| The heading font | `--font-mono` | 1 |
| Anything about `h1`, `h2`, `h3` specifically | the `h1 { … }` blocks | 5 |

The sizes use `clamp(minimum, preferred, maximum)`. Each one slides smoothly
between the first and last number as the window widens, which is why there are
no media queries for screen size in the file. To make `h2` bigger, change the
last number:

```css
--h2: clamp(1.10rem, 1.03rem + 0.35vw, 1.30rem);
/*                                        ^^^^^ make this bigger */
```

### One gotcha: the browser bar colour

`index.html` contains two `theme-color` meta tags. Those are what tint the
browser's own bar on a phone, and they must be written as literal colours —
they cannot use the variables from `style.css`. If you change `--bg`, change
the matching `theme-color` too or the browser bar will be a slightly wrong
shade:

```html
<meta name="theme-color" content="#fdfdfc" media="(prefers-color-scheme: light)">
<meta name="theme-color" content="#101216" media="(prefers-color-scheme: dark)">
```

### Fonts

No font files are downloaded, and there is no Google Fonts request. The page
asks the operating system for its own UI font (`system-ui`) and its own
monospace font (`ui-monospace`). That is why it looks native on every platform
and why typography costs zero bytes. The long lists of names in `--font-sans`
and `--font-mono` are fallbacks for older systems; the final `sans-serif` /
`monospace` is a generic name every browser knows, so the site can never end
up without a usable font.

To use the system font everywhere instead of monospace headings, delete
`font-family: var(--font-mono);` from section 5.

## Adding the blog later

The groundwork is deliberately not built. There is a commented-out `<nav>`
block in `index.html` waiting for the day you want it. The recipe:

1. Make a `blog/` directory and copy `index.html` into it as
   `blog/index.html`.
2. In `blog/index.html`, change the stylesheet line to point one level up:
   ```html
   <link rel="stylesheet" href="../style.css">
   ```
   That is the only path that needs to change, and it is the reason a single
   shared `style.css` was worth splitting out from the start: every future
   page reuses one cached file instead of carrying its own copy.
3. In the **root** `index.html`, delete the two `<!--` / `-->` lines around
   the `<nav class="site-nav">` block.
4. The nav in the root file uses `href="./"` for About and `href="blog/"` for
   Blog. In a file inside `blog/`, use `href="../"` for About and `href="./"`
   for Blog, and move `aria-current="page"` onto whichever link is the current
   page.
5. For an article, copy `blog/index.html` to something like
   `blog/i2c-bus-hangs.html` and write. Same `href="../style.css"`.

Keep filenames lowercase with dashes. `<code>` and `<pre>` are already styled
in section 8 of `style.css`, so code samples in articles will look right
without any extra work.

## When to split files

You asked where the line is, so, concretely:

- **Now, keep 2 files.** For one page, splitting the CSS into pieces costs you
  extra HTTP requests and extra files to hold in your head, in exchange for
  nothing.
- **Split `style.css` when** it passes roughly 500 lines, or when a second page
  needs genuinely different rules. Split it *by purpose* — `tokens.css`,
  `layout.css` — not by page.
- **Don't** split into `about.css`, `blog.css`. Each page-specific file ends up
  redefining the same things and the two drift apart.
- **Do** make each article its own `.html` file. That is content, not
  structure, and content is always one file per thing.
- **Don't add a build step** until the annoyance of not having one is bigger
  than the annoyance of maintaining it. For a site this size it never will be.

## Why it is fast

| | Raw | Gzipped (what your phone downloads) |
|---|---|---|
| Everything, as written | 16.9 KB | 6.4 KB |
| With comment blocks stripped | 6.3 KB | 2.6 KB |

Roughly two thirds of that is the comment blocks you are reading for guidance.
A server with gzip or brotli enabled — which is every host worth using —
compresses them down to a few kilobytes, so they are worth keeping while you
are learning the file. If you ever want the absolute minimum, delete the
comment blocks; the site will not change appearance at all.

There are exactly **two network requests**: the HTML and the CSS. No fonts, no
images, no icon file (the favicon is embedded in the HTML as a `data:` URL), no
JavaScript, and no third-party anything. Once cached, a repeat visit is
effectively free.

## Putting it online

Upload the directory to any static host — GitHub Pages, Netlify, Cloudflare
Pages, Codeberg Pages, or a webroot on your own server. There is nothing to
build and no server-side anything, so no configuration is required.

Because all paths are relative, the site works identically at the root of a
domain and in a subdirectory, so you do not have to decide now.

## Licence

The code (`index.html`, `style.css`) is MIT. The writing is
CC BY-SA 4.0. See `LICENSE` for the summary and `LICENSE-MIT` /
`LICENSE-CC-BY-SA-4.0` for the actual texts.

Two placeholders remain in `LICENSE` and `LICENSE-MIT`: replace `Erik Karlgren Domercq`
and, if you like, add a year.
