# HTML DSL API

Write tags inside `html`, `body`, or `fragment` blocks. Use `markup\tag` outside them.
Examples on this page assume this import from the repository root:

```art
import {src/html}!
```

For a published release, use `import {html}!`. See the [README](README.md#installation)
if you're trying this branch for the first time.

[Entry points](#entry-points) · [Children](#children) · [Attributes](#attributes) · [Components](#components-and-control-flow) · [Raw HTML](#raw-html-scripts-and-styles) · [Scope](#name-collisions) · [Tags](#tag-reference) · [Migration](#migration)

## Entry Points

| Call | Result |
| --- | --- |
| `html content` | HTML document, starting with `<!DOCTYPE html>` |
| `body content` | A `<body>` element |
| `fragment content` | Markup with no wrapper |
| `markup\p content` | A tag outside a child block |
| `markup\element "my-widget" content` | A custom tag |
| `markup\raw "<b>Trusted</b>"` | Markup with escaping turned off |
| `to :string node` | A compact HTML string |

`html` and `body` forward attributes to their elements.
`markup\html []` includes the doctype; `markup\element "html" []` doesn't.

```art
print html.lang: "en" [
  head [title "Hello"]
  body [p "Welcome!"]
]
; <!DOCTYPE html><html lang="en"><head><title>Hello</title></head><body><p>Welcome!</p></body></html>
```

## Children

Every tag takes **one** content argument:

| Content | What happens |
| --- | --- |
| `p "Hello"` | Escapes the text |
| `div [h2 "Hello" p "Welcome"]` | Evaluates the block and nests its markup |
| `div fragment [p "Hello"]` | Nests existing markup without escaping it again |
| `div #[class: "empty"]` | Uses the dictionary as attributes; no children |
| `br []` | Emits an empty void tag: `<br>` |

Text escapes `&`, `<`, `>`, and double quotes. Only strings and markup can be
children; `null` is ignored. Convert numbers with `to :string`.

Blocks run **once**, when you create the markup. Rendering it again doesn't rerun them.
`->` is handy when there's only one child expression:

```art
print fragment [article -> div.class: "content" ["Hello"]]
; <article><div class="content">Hello</div></article>
```

Void tags such as `input`, `img`, and `br` still need an argument. Pass `[]`,
`""`, or an attribute dictionary. Nonempty content raises an error.

## Attributes

Pass attributes with `.name: value`, or use a dictionary for an empty element:

```art
print fragment [
  a.href: "https://arturo-lang.io" .class: "link" "Arturo"
  input #[name: "email" required: true]
  p.data_state: "ready" .aria_hidden: false "Hello"
]
; <a class="link" href="https://arturo-lang.io">Arturo</a><input name="email" required><p aria-hidden="false" data-state="ready">Hello</p>
```

| Rule | Example |
| --- | --- |
| Names become lowercase; `_` becomes `-` | `data_state` → `data-state` |
| Call attributes override dictionary attributes after normalization | `input.name: "new" #[name: "old"]` |
| `null` omits an attribute | `p.id: null "Hello"` |
| HTML boolean attributes use bare names for `true`, disappear for `false` | `input.required: true []` → `<input required>` |
| Other logical values stay quoted | `.aria_hidden: false` → `aria-hidden="false"` |
| Attributes sort by normalized name | `.id: "x" .class: "y"` → `class="y" id="x"` |

Values can be strings, literals, integers, floats, or logical values. Values are
escaped just like text. There are no tag-specific shortcuts or positional attributes.

## Components and Control Flow

Return markup from a function, then call it inside a child block:

```art
card: function [name][fragment [article [h2 name]]]

print fragment [
  div [
    "People: "
    loop ["Ana" "João"] 'person [card person]
    if true [p "Welcome"]
    if false [p "Hidden"]
  ]
]
```

The false conditional returns `null`, so it adds nothing. Nested fragments keep
their markup; you don't need `raw` for components.

## Raw HTML, Scripts, and Styles

`raw` means “trust this string.” It skips escaping:

```art
print fragment [
  raw "<b>Trusted HTML</b>"
  style [raw "p { color: navy; }"]
  script [raw "console.log('Hello');"]
  element.data_state: "ready" "my-widget" [p "Hello <world>"]
]
```

Use `markup\raw` and `markup\element` outside child blocks.
Script and style bodies accept only raw markup or empty content. Ordinary strings
raise an error, even when nested in a block.

**Keep untrusted input out of `raw`.** This library doesn't sanitize URLs,
JavaScript, or CSS. Escaping text isn't a substitute for checking a link's URL.

## Name Collisions

Inside child blocks, official tag names take precedence over caller bindings and
built-ins. That includes `div`, `map`, `select`, and `input`.

Need one of those built-ins? Save it under another name before entering the block.
Caller bindings are restored when the block returns.

Blocks also bind `element`, `raw`, and `html`. Assignments follow function scope;
use a shared collection if you need to observe changes outside the block.

## Separate DSL Objects

```art
other: to :htmlDsl []!
print other\p "Hello"
```

Each initialized object has the same helpers and scoped child evaluation.
`markup\tags` contains the canonical tag inventory used to generate helpers;
`html` is provided separately as the document helper.

## Errors and Limits

The DSL rejects malformed tag or attribute names, unsupported values, nonempty
void elements, and ordinary script/style text.

It serializes HTML5. It doesn't check whether an element belongs inside another
element. HTML4, obsolete elements such as `param`, and SVG/MathML serialization
aren't supported. Custom tags use the generic name rules; `element` doesn't add
SVG/MathML behavior or check custom-element naming conventions.

## Migration

Updating from the old API? Start here:

| Before | Now |
| --- | --- |
| `html\p "Hello"` | `markup\p "Hello"` outside blocks; `p "Hello"` inside |
| `html\element "my-widget" []` | `markup\element "my-widget" []` |
| `html\raw "<b>Hello</b>"` | `markup\raw "<b>Hello</b>"` |
| A fragment used as a string | `to :string fragment content` |
| Intentional HTML passed as text | `markup\raw trustedHtml` |

`html` is now a document function; `markup` is the public DSL object. Tags and
fragments return markup objects instead of strings. Print them directly or render
with `to :string`. Nesting preserves markup; text is escaped.

Use official tag names inside child blocks. Legacy backslash shorthand is no
longer the documented interface. There are no aliases or specialized positional
attribute arguments. Void tags have no closing tag; dictionaries no longer
request self-closing tags. HTML4 mode has been removed.

## Tag Reference

All supported tags are listed below, grouped by
[MDN's HTML element categories](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements).
These groups help you find a tag; they don't enforce nesting rules.

Examples are expressions inside `fragment [...]`.

| Content kind | Accepted content |
| --- | --- |
| Normal | String, markup, child block, or attribute dictionary for an empty element |
| Void | Empty string/block or attribute dictionary; no closing tag |
| Trusted | Empty content, raw markup, block of raw values, or attribute dictionary |
| Document | Normal content, with a doctype before the element |

### Main root

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `html` | HTML5 document root | Document | `html [body [p "Hello"]]` |

### Document metadata and sectioning root

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `body` | Document body | Normal | `body []` |
| `head` | Document metadata | Normal | `head []` |

### Web components

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `slot` | Web component insertion point | Normal | `slot "Text"` |
| `template` | Inert reusable template | Normal | `template "Text"` |

### Demarcating edits

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `del` | Deleted text | Normal | `del "Text"` |
| `ins` | Inserted text | Normal | `ins "Text"` |

### Embedded content

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `embed` | External embedded content | Void | `embed #[]` |
| `iframe` | Nested browsing context | Normal | `iframe "Text"` |
| `object` | External resource | Normal | `object "Text"` |
| `picture` | Responsive image container | Normal | `picture []` |
| `source` | Alternative media source | Void | `source #[]` |

### Forms

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `button` | Action button | Normal | `button "Text"` |
| `datalist` | Suggested input options | Normal | `datalist []` |
| `fieldset` | Form control group | Normal | `fieldset "Text"` |
| `form` | Submission form | Normal | `form "Text"` |
| `input` | Input control | Void | `input #[]` |
| `label` | Control label | Normal | `label "Text"` |
| `legend` | Fieldset caption | Normal | `legend "Text"` |
| `meter` | Scalar measurement | Normal | `meter "Text"` |
| `optgroup` | Option group | Normal | `optgroup []` |
| `option` | Selectable option | Normal | `option "Text"` |
| `output` | Calculation result | Normal | `output "Text"` |
| `progress` | Task progress | Normal | `progress "Text"` |
| `select` | Selection control | Normal | `select []` |
| `textarea` | Multiline text control | Normal | `textarea "Text"` |

### Inline text semantics

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `a` | Hyperlink | Normal | `a "Text"` |
| `abbr` | Abbreviation | Normal | `abbr "Text"` |
| `b` | Attention text | Normal | `b "Text"` |
| `bdi` | Bidirectional isolation | Normal | `bdi "Text"` |
| `bdo` | Direction override | Normal | `bdo "Text"` |
| `br` | Line break | Void | `br #[]` |
| `cite` | Work title | Normal | `cite "Text"` |
| `code` | Code fragment | Normal | `code "Text"` |
| `data` | Machine readable value | Normal | `data "Text"` |
| `dfn` | Defined term | Normal | `dfn "Text"` |
| `em` | Stress emphasis | Normal | `em "Text"` |
| `i` | Alternate voice | Normal | `i "Text"` |
| `kbd` | Keyboard input | Normal | `kbd "Text"` |
| `mark` | Highlighted text | Normal | `mark "Text"` |
| `q` | Inline quotation | Normal | `q "Text"` |
| `s` | No longer accurate text | Normal | `s "Text"` |
| `rp` | Ruby fallback | Normal | `rp "Text"` |
| `rt` | Ruby annotation | Normal | `rt "Text"` |
| `ruby` | Ruby annotation group | Normal | `ruby "Text"` |
| `samp` | Sample output | Normal | `samp "Text"` |
| `small` | Side comment | Normal | `small "Text"` |
| `span` | Generic inline container | Normal | `span "Text"` |
| `strong` | Strong importance | Normal | `strong "Text"` |
| `sub` | Subscript | Normal | `sub "Text"` |
| `sup` | Superscript | Normal | `sup "Text"` |
| `time` | Date or time | Normal | `time "Text"` |
| `u` | Non textual annotation | Normal | `u "Text"` |
| `var` | Variable | Normal | `var "Text"` |
| `wbr` | Optional line break | Void | `wbr #[]` |

### Interactive elements

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `details` | Disclosure widget | Normal | `details "Text"` |
| `dialog` | Dialog | Normal | `dialog "Text"` |
| `summary` | Disclosure heading | Normal | `summary "Text"` |

### Document metadata

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `base` | Base URL | Void | `base #[]` |
| `link` | External resource relationship | Void | `link #[]` |
| `meta` | Metadata | Void | `meta #[]` |
| `title` | Document title | Normal | `title "Text"` |
| `style` | Trusted stylesheet | Trusted | `style [raw "/* trusted */"]` |

### Image and multimedia

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `area` | Image map region | Void | `area #[]` |
| `audio` | Audio player | Normal | `audio "Text"` |
| `img` | Image | Void | `img #[]` |
| `map` | Image map | Normal | `map "Text"` |
| `track` | Timed text track | Void | `track #[]` |
| `video` | Video player | Normal | `video "Text"` |

### Scripting

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `canvas` | Script drawing surface | Normal | `canvas "Text"` |
| `noscript` | Scripting fallback | Normal | `noscript "Text"` |
| `script` | Trusted script | Trusted | `script [raw "/* trusted */"]` |

### Content sectioning

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `address` | Contact information | Normal | `address "Text"` |
| `article` | Self contained composition | Normal | `article "Text"` |
| `aside` | Related content | Normal | `aside "Text"` |
| `footer` | Section footer | Normal | `footer "Text"` |
| `header` | Section introduction | Normal | `header "Text"` |
| `h1` | Level 1 heading | Normal | `h1 "Text"` |
| `h2` | Level 2 heading | Normal | `h2 "Text"` |
| `h3` | Level 3 heading | Normal | `h3 "Text"` |
| `h4` | Level 4 heading | Normal | `h4 "Text"` |
| `h5` | Level 5 heading | Normal | `h5 "Text"` |
| `h6` | Level 6 heading | Normal | `h6 "Text"` |
| `hgroup` | Heading group | Normal | `hgroup []` |
| `main` | Main content | Normal | `main "Text"` |
| `nav` | Navigation | Normal | `nav "Text"` |
| `section` | Thematic section | Normal | `section "Text"` |
| `search` | Search region | Normal | `search "Text"` |

### Table content

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `caption` | Table caption | Normal | `caption "Text"` |
| `col` | Table column | Void | `col #[]` |
| `colgroup` | Table column group | Normal | `colgroup []` |
| `table` | Data table | Normal | `table []` |
| `tbody` | Table body | Normal | `tbody []` |
| `td` | Table data cell | Normal | `td "Text"` |
| `tfoot` | Table footer | Normal | `tfoot []` |
| `th` | Table header cell | Normal | `th "Text"` |
| `thead` | Table header | Normal | `thead []` |
| `tr` | Table row | Normal | `tr []` |

### Text content

| Tag | Purpose | Content | Example |
| --- | --- | --- | --- |
| `blockquote` | Block quotation | Normal | `blockquote "Text"` |
| `dd` | Description details | Normal | `dd "Text"` |
| `div` | Generic flow container | Normal | `div "Text"` |
| `dl` | Description list | Normal | `dl []` |
| `dt` | Description term | Normal | `dt "Text"` |
| `figcaption` | Figure caption | Normal | `figcaption "Text"` |
| `figure` | Self contained figure | Normal | `figure "Text"` |
| `hr` | Thematic break | Void | `hr #[]` |
| `li` | List item | Normal | `li "Text"` |
| `menu` | Unordered command list | Normal | `menu []` |
| `ol` | Ordered list | Normal | `ol []` |
| `p` | Paragraph | Normal | `p "Text"` |
| `pre` | Preformatted text | Normal | `pre "Text"` |
| `ul` | Unordered list | Normal | `ul []` |
