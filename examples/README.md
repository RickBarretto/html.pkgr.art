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

## Run and preview

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
arturo --no-color test.art examples
arturo --no-color test.art
```

The example suite executes every listed script in its own process and compares
stdout byte for byte with its expected HTML, including whitespace and the final
newline. It also checks exit status. These tests run offline and are included in
the existing CI suite. They verify serialization; they do not load CDN assets or
exercise browser layout or JavaScript events.

When changing an example, review its rendered HTML and update the corresponding
expected file deliberately. Add new example paths to `test/examples.test.art`.
