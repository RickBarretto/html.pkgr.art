# HTML DSL for Arturo

HTML5 documents and fragments for Arturo 0.10.0. No runtime dependencies.

```art
import {src/html}!
print fragment html [
    head [title "Hello"]
    body [
        h1.class: "xl" "Hello, World"
        article -> div.class: "flex-col" ["Children"]
    ]
]
```

Tag helpers return typed markup values. `fragment` combines a DSL block or a
markup value into typed markup without a wrapper element. Print it directly, or use `to :string` for a compact string.
Strings are escaped; nested markup is preserved. Blocks evaluate once.
Use `markup\element "my-widget" "Hello"` for custom tags.
Dictionary arguments supply attributes on empty elements. Dynamic attributes
override dictionary values, underscores become hyphens, and null values are
omitted. HTML boolean attributes accept logical values; ARIA logical values
remain quoted strings.

For reusable components, return a fragment from a function:

```art
import {src/html}!
greeting: function [name][fragment [p name]]
print fragment [greeting "Arturo"]
```

`markup\raw` explicitly trusts a string as markup. Script and style bodies require
raw markup. Only use trusted content: this library does not sanitize URLs,
JavaScript, or CSS. SVG/MathML serialization, HTML4, and content-model validation
are outside this release's supported scope.

Run the introductory example with `arturo --no-color exp.art`.

Runnable [examples](examples/README.md) include a blog, landing page, and
ChatGPT-style chat interface, plus Tailwind, daisyUI, and Bootstrap examples.
Smaller DSL demonstrations live in [examples/basics](examples/basics/). Each
example has a checked-in HTML output in [examples/expected](examples/expected/).

To view the examples in your browser, run from the repository root:

```sh
arturo --no-color preview.art
```

The script renders all examples, then starts **Lanx**, Arturo's static-file
server. Its versioned import automatically installs Lanx 0.2.0 and any missing
dependencies on first use; no separate installation command is needed.

Open **http://localhost:18966/** and choose an example. Keep the command running
while browsing, and press **Ctrl+C** to stop the server. After editing an
example, stop and rerun the command to regenerate the pages.

To generate the gallery without starting Lanx:

```sh
arturo --no-color preview.art build
```

Generated HTML is written to `.preview/`, which is ignored by Git. Build-only
mode does not import or install Lanx. Framework styling requires internet access;
the chat examples provide local demo replies.

You can also render an individual page or run the example output tests:

```sh
arturo --no-color examples/pages/blog.art > /tmp/blog.html
arturo --no-color test.art examples
```

The example suite runs each script in a separate process and checks its exit
status and exact output, including the trailing newline. It also runs as part
of the full test suite and existing CI workflow.

Development requires Unitt 3.1.1 on PATH:

```sh
arturo -p install unitt 3.1.1
arturo --no-color test.art
arturo --no-color test.art element
```

The entry point delegates to Unitt's native runner using `config.art`. It also
preserves failure status for Unitt 3.1.1's uncaught-error path on Arturo 0.10.0.
You can run a test file directly with `arturo --no-color test/element.test.art`.
Null results in DSL blocks are ignored, allowing conditionals with no output.

See [API.md](API.md) for the complete tag reference, scoped name collisions,
entry points, attributes, control flow, and migration examples.

## Migration

The public DSL object is now `markup`; `html` and `body` are callable entry
functions. Replace `html\...` with `markup\...`. Within fragment and element
child blocks, use official tag names directly. These local helpers take
precedence over built-ins and restore caller bindings when evaluation returns.

Tag and fragment results are markup objects rather than strings. Print them or
render with `to :string`. Output is compact and text is escaped. Replace
intentional HTML strings with `markup\raw`. Void tags emit no closing tag;
dictionaries no longer request self-closing tags. HTML4 mode is removed.
