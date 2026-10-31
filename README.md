# HTML DSL for Arturo

HTML5 documents and fragments for Arturo 0.10.0. No runtime dependencies.

```art
import {src/html}!
print fragment [
    \html [
        \head [\title "Hello"]
        \body [\h1 "Hello, Arturo!" \input.required: true #[name: "user"]]
    ]
]
```

Tag helpers return typed markup values. `fragment` renders a DSL block or a
markup value to a compact string; `to :string` also renders a markup value.
Strings are escaped; nested markup is preserved. Blocks evaluate once.
Use `html\element "my-widget" "Hello"` for custom tags.
Dictionary arguments supply attributes on empty elements. Dynamic attributes
override dictionary values, underscores become hyphens, and null values are
omitted. HTML boolean attributes accept logical values; ARIA logical values
remain quoted strings.

For reusable components, return markup from a function:

```art
import {src/html}!
greeting: function [name][html\element "p" name]
print fragment [greeting "Arturo"]
```

`html\raw` explicitly trusts a string as markup. Script and style bodies require
raw markup. Only use trusted content: this library does not sanitize URLs,
JavaScript, or CSS. SVG/MathML serialization, HTML4, and content-model validation
are outside this release's supported scope.

Run the example with `arturo --no-color exp.art`.

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

## Migration

Tag results are markup objects rather than strings. Render with `fragment` or
explicit string conversion. Output is compact and text is escaped. Replace
intentional HTML strings with `html\raw`. Void tags emit no closing tag;
dictionaries no longer request self-closing tags. HTML4 mode is removed.
