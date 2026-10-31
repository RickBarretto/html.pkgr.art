<h1 align="center">html</h1>

<p align="center">
    <i>Write HTML in Arturo. Skip the string juggling.</i>
    <br><br><br>
    <img
        src="./docs/icons/arturo-light-mode.png#gh-light-mode-only"
        alt="Arturo logo"
        width="20"
    />
    <img
        src="./docs/icons/arturo-dark-mode.png#gh-dark-mode-only"
        alt="Arturo logo"
        width="20"
    />
    <br>
    <b>~ CODE IS ART ~</b>
    <br><br>
</p>

<p align="center">
    <a href="https://www.arturo-lang.io"><img src="https://img.shields.io/badge/language-Arturo-6A156B.svg?&style=flat&colorA=000000&colorB=000000" alt="Language: Arturo"/></a>
    &nbsp;
    <a href="https://github.com/RickBarretto/html.pkgr.art/blob/main/LICENSE"><img src="https://img.shields.io/github/license/RickBarretto/html.pkgr.art?&style=flat&colorA=000000&colorB=000000" alt="License"/></a>
    &nbsp;
    <a href="https://github.com/RickBarretto/html.pkgr.art/actions"><img src="https://img.shields.io/github/actions/workflow/status/RickBarretto/html.pkgr.art/test.yml?branch=main&style=flat&colorA=000000&colorB=000000" alt="CI Status"/></a>
</p>

## At a Glance

```art
import {src/html}!

print html [
  head [title "Hello"]
  body [
    h1.class: "title" "Hello, Arturo!"
    p "HTML without string templates."
  ]
]
```

Tags become functions. Text is escaped for you. The result is compact HTML5,
ready to print or write to a file. Requires **Arturo 0.10.0+**; no runtime dependencies.

[Quick start](#quick-start) · [API reference](API.md) · [Examples](examples/README.md) · [Migration](API.md#migration)

## Installation

```sh
arturo -p install html
```

Then use `import {html}!` in your script. Arturo can also install it on first import.

**Trying this branch?** Clone the repo and use `import {src/html}!` from its root,
as shown here. Registry installs use published releases and may have an older API.

## Quick Start

Save the example above as `page.art` in the repository root, then run:

```sh
arturo page.art > page.html
```

Open `page.html` in your browser. That's it.

### Just need a fragment?

Use `fragment` to group elements without adding a wrapper:

```art
print fragment [
  h2 "Hello <world>"
  p "This text is escaped."
]
; <h2>Hello &lt;world&gt;</h2><p>This text is escaped.</p>
```

### Build a component

Return a fragment from a function. Nest it like any tag:

```art
greeting: function [name][fragment [p name]]

print fragment [
  section [
    greeting "Ana"
    greeting "João"
  ]
]
```

### Add attributes

Use `.name: value`. Underscores become hyphens:

```art
print fragment [
  a.href: "https://arturo-lang.io" "Visit Arturo"
  input #[name: "email" required: true]
  p.data_state: "ready" "All set!"
]
```

Need custom tags, loops, or trusted HTML? Head to the [API reference](API.md).

## Try the Examples

Blog, landing page, chat UI, Tailwind, daisyUI, and Bootstrap — [pick an example](examples/README.md).

```sh
arturo --no-color preview.art
```

Open **http://localhost:18966/**. First run installs Lanx 0.2.0 and its missing
dependencies. Stop with **Ctrl+C**; rerun after edits. Framework examples need
internet access for styling; chat replies are local demos.

To generate the gallery without starting the server or installing Lanx:

```sh
arturo --no-color preview.art build
```

## A Few Rules

- Every tag takes one content argument. For an empty tag, use `[]` or `#[]`.
- Tags return markup objects. Print them, or use `to :string` when you need a string.
- Inside child blocks, tag names such as `div` and `input` take over built-ins.
  See [name collisions](API.md#name-collisions).
- `raw` skips escaping. Use it only for trusted content, including script and style bodies.
  URLs, JavaScript, and CSS are not sanitized.
- This is an HTML5 serializer. HTML4, SVG/MathML, and content-model validation aren't supported.

## Development

Install [Unitt](https://github.com/RickBarretto/unitt) 3.1.1 and make sure its
executable is on `PATH` (`$HOME/.arturo/packages/bin` on Unix):

```sh
arturo -p install unitt 3.1.1
arturo --no-color test.art            # all suites, including example output
arturo --no-color test.art element    # one suite
arturo --no-color exp.art             # small demo
```

The test entry point uses `config.art` and preserves failure status for Unitt's
uncaught-error path on Arturo 0.10.0.

## License

[MIT](LICENSE).
