# Runnable examples

Run commands from the repository root. Each script prints a complete HTML page
or fragment to stdout; the matching output lives under `expected/` with the same
relative path.

| Folder | Example | Demonstrates |
| --- | --- | --- |
| `basics/` | `scoped.art` | Local tag helpers and `->` nesting |
| `basics/` | `document.art` | Metadata, forms, tables, and mixed text |
| `basics/` | `components.art` | Reusable components, loops, and conditionals |
| `basics/` | `attributes.art` | Generic attributes, overrides, escaping, and custom elements |
| `basics/` | `trusted-content.art` | Explicitly trusted markup, scripts, and styles |
| `pages/` | `blog.art` | Responsive blog with post components, dates, and an author sidebar |
| `pages/` | `landing.art` | Responsive product landing page with features and contact links |
| `pages/` | `chat.art` | ChatGPT-style conversation layout with a composer and local demo replies |
| `tailwind/` | `landing.art` | Landing page using Tailwind utility classes |
| `daisyui/` | `chat.art` | Chat interface using daisyUI buttons, navigation, and form controls |
| `bootstrap/` | `blog.art` | Blog using Bootstrap's responsive grid and cards |

## Preview gallery with Lanx

From the repository root:

```sh
arturo --no-color preview.art
```

The script imports Lanx 0.2.0, automatically installing it and any missing
dependencies on first use. No separate installation step is required.

Open **http://localhost:18966/** and choose an example. Stop the server with
**Ctrl+C**. Rerun the command after editing an example to regenerate its HTML.
Lanx serves the generated `.preview/` folder, which is ignored by Git.
It is a development server; it may listen on network interfaces beyond localhost.

To generate the gallery without installing Lanx or starting a listener:

```sh
arturo --no-color preview.art build
```

Preview generation and output tests share the paths in `examples/catalog.txt`.
Only HTML output goes into the served directory; the repository source is not
served. Lanx is a development dependency and does not change the DSL's runtime
dependencies.

## Run and preview individual pages

```sh
arturo --no-color examples/basics/scoped.art
arturo --no-color examples/pages/blog.art > /tmp/blog.html
arturo --no-color examples/pages/landing.art > /tmp/landing.html
arturo --no-color examples/pages/chat.art > /tmp/chat.html
arturo --no-color examples/tailwind/landing.art > /tmp/tailwind-landing.html
arturo --no-color examples/daisyui/chat.art > /tmp/daisyui-chat.html
arturo --no-color examples/bootstrap/blog.art > /tmp/bootstrap-blog.html
```

Open the generated `.html` file in a browser. You can also preview the checked-in
HTML under `expected/` directly. Plain pages include their CSS and work offline.
The chat examples support sending messages and resetting the conversation using
**New chat**. Replies use a fixed local JavaScript demonstration; no AI model,
API key, backend, or conversation persistence is included. User messages are
inserted with `textContent`, keeping input such as `<script>` as displayed text.
The form also supports the browser's standard required-field validation.

The framework examples load their assets from a CDN, so styling requires an
internet connection:

- [Tailwind browser CDN](https://tailwindcss.com/docs/installation/play-cdn):
  Tailwind 4, using the browser script for a build-free example. For production,
  use Tailwind's compiled CSS workflow.
- [daisyUI CDN](https://daisyui.com/docs/cdn/): daisyUI 5.7.50 CSS alongside
  Tailwind 4 utilities, with the light theme selected on the document root.
- [Bootstrap CDN](https://getbootstrap.com/docs/5.3/getting-started/introduction/):
  Bootstrap 5.3.8 CSS. This example uses CSS components without Bootstrap JS.

## Verify output

```sh
unitt --suffix:examples.test.art
unitt
```

The example suite executes every listed script in its own process and compares
stdout byte for byte with its expected HTML, including whitespace and the final
newline. It also checks exit status. These tests run offline and are included in
the existing CI suite. They verify serialization; they do not load CDN assets or
exercise browser layout or JavaScript events.

When changing an example, review its rendered HTML and update the corresponding
expected file deliberately. Add new example paths to `examples/catalog.txt` to include them in both
the gallery and output tests.
