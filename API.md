# HTML DSL API

Write HTML elements inside `html`, `body`, or `fragment` blocks. Use `markup\p` outside them.
Examples on this page assume this import from the repository root:

```art
import {src/html}!
```

For a published release, use `import {html}!`. See the [README](README.md#installation)
if you're trying this branch for the first time.

[Entry points](#entry-points) · [Children](#children) · [Attributes](#attributes) · [Components](#components-and-control-flow) · [Raw HTML](#raw-html-scripts-and-styles) · [Scope](#name-collisions) · [✨ Modern & emerging HTML](#-modern--emerging-html) · [Progress](#implementation-progress) · [HTML Elements](#html-elements-reference) · [Migration](#migration)

## Entry Points

| Call | Result |
| --- | --- |
| `html content` | HTML document, starting with `<!DOCTYPE html>` |
| `body content` | A `<body>` element |
| `fragment content` | Markup with no wrapper |
| `markup\p content` | An HTML element outside a child block |
| `markup\element "my-widget" content` | A custom element |
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

Every element takes **one** content argument:

| Content | What happens |
| --- | --- |
| `p "Hello"` | Escapes the text |
| `div [h2 "Hello" p "Welcome"]` | Evaluates the block and nests its markup |
| `div fragment [p "Hello"]` | Nests existing markup without escaping it again |
| `div #[class: "empty"]` | Uses the dictionary as attributes; no children |
| `br []` | Emits an empty void element: `<br>` |

Text escapes `&`, `<`, `>`, and double quotes. Only strings and markup can be
children; `null` is ignored. Convert numbers with `to :string`.

Blocks run **once**, when you create the markup. Rendering it again doesn't rerun them.
`->` is handy when there's only one child expression:

```art
print fragment [article -> div.class: "content" ["Hello"]]
; <article><div class="content">Hello</div></article>
```

Void elements such as `input`, `img`, and `br` still need an argument. Pass `[]`,
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
escaped just like text. There are no element-specific shortcuts or positional attributes.

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

Inside child blocks, official element names take precedence over caller bindings and
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
`markup\tags` contains the canonical element inventory used to generate helpers;
`html` is provided separately as the document helper.

## Errors and Limits

The DSL rejects malformed element or attribute names, unsupported values, nonempty
void elements, and ordinary script/style text.

It serializes HTML5. It doesn't check whether an element belongs inside another
element. HTML4, obsolete or deprecated HTML APIs and elements such as `font`,
`center`, and `param`, and SVG/MathML serialization aren't supported.
The superseded `permission` experiment is an explicit exception, documented below.
Custom elements use the generic name rules; `element` doesn't add SVG/MathML behavior,
check custom-element naming conventions, or reject obsolete names. Emitting a
name through `element` or `raw` does not imply support for its legacy behavior.

## ✨ Modern & Emerging HTML

**Build with the web's next ideas.** These helpers put modern interface features
within the same familiar Arturo syntax. ✨

| Helper | Interface feature | Status / reference |
| --- | --- | --- |
| `selectedcontent` | Mirrors the selected option inside a customizable select | [HTML Living Standard](https://html.spec.whatwg.org/multipage/form-elements.html#the-selectedcontent-element) |
| `geolocation` | Browser-controlled location sharing | [Capability element](https://developer.chrome.com/blog/geolocation-html-element); browser availability varies |
| `usermedia` | Camera and microphone access controls | [Emerging capability element](https://developer.chrome.com/blog/usermedia-html-element) |
| `install` | Browser-controlled web app installation | [Experimental implementation](https://developer.chrome.com/blog/install-element-ot) |
| `permission` | Generic permission control | [Superseded experiment](https://developer.chrome.com/blog/geolocation-html-element#from-generic-permission-to-specific-geolocation), retained intentionally |

Inside `fragment [...]`, for example:

```art
select [
  button [selectedcontent []]
  option.value: "sparkles" "✨ Sparkles"
  option.value: "stars" "🌟 Stars"
]
geolocation [button "Use my location"]
usermedia [button "Enable camera and microphone"]
install [a.href: "/app" "Open the app"]
```

These helpers serialize markup; browser support, styling, JavaScript event
handling, and any required experimental flags or trial enrollment determine the
resulting interface. Customizable selects also require the appropriate CSS.
Child content in the capability controls can provide a fallback for browsers
that don't recognize the element; a working fallback action needs its own
implementation.

For implementations of the earlier permission experiment:

```art
permission.type: "camera microphone" []
```

`permission` evolved into the specialized `geolocation` and `usermedia` controls.
It is retained for experimentation, rather than as a supported legacy browser API.
The helpers also appear in the [forms](#forms) and
[interactive elements](#interactive-elements) references below.

## Migration

Updating from the old API? Start here:

| Before | Now |
| --- | --- |
| `html\p "Hello"` | `markup\p "Hello"` outside blocks; `p "Hello"` inside |
| `html\element "my-widget" []` | `markup\element "my-widget" []` |
| `html\raw "<b>Hello</b>"` | `markup\raw "<b>Hello</b>"` |
| A fragment used as a string | `to :string fragment content` |
| Intentional HTML passed as text | `markup\raw trustedHtml` |

`html` is now a document function; `markup` is the public DSL object. Elements and
fragments return markup objects instead of strings. Print them directly or render
with `to :string`. Nesting preserves markup; text is escaped.

Use official element names inside child blocks. Legacy backslash shorthand is no
longer the documented interface. There are no aliases or specialized positional
attribute arguments. Void elements have no closing tag; dictionaries no longer
request self-closing tags. HTML4 mode has been removed.

## Implementation Progress

![HTML elements implemented: 117 of 117 (100%)](docs/element-coverage.svg)

**117 / 117 documented HTML element helpers implemented (100%).** This measures the
[HTML Elements Reference](#html-elements-reference) against the helpers in [src/html.art](src/html.art),
including the separate `html` document helper and four experimental capability
controls. It measures markup serialization; browser support and full HTML
standard coverage are outside this count.

Implemented DSL features:

- [x] Documents, fragments, and custom elements
- [x] Nested child blocks and reusable components
- [x] Text and attribute escaping
- [x] Attribute dictionaries, normalization, and boolean attributes
- [x] Void elements and trusted script/style content
- [x] Scoped element helpers and separate DSL objects

## HTML Elements Reference

All supported HTML elements are listed below, grouped by
[MDN's HTML element categories](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements).
These groups help you find an element; they don't enforce nesting rules.

References link to MDN, Google's web.dev guides or Chrome developer articles,
and the WHATWG HTML Living Standard. Google guides may cover a group of related
elements. A dash means no element reference is available in that source;
experimental proposals may not be part of the HTML Living Standard.

Examples are expressions inside `fragment [...]`.

| Content kind | Accepted content |
| --- | --- |
| Normal | String, markup, child block, or attribute dictionary for an empty element |
| Void | Empty string/block or attribute dictionary; no closing tag |
| Trusted | Empty content, raw markup, block of raw values, or attribute dictionary |
| Document | Normal content, with a doctype before the element |

### Main root

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `html` | HTML5 document root | Document | `html [body [p "Hello"]]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/html) · [Google](https://web.dev/learn/html/document-structure) · [WHATWG](https://html.spec.whatwg.org/multipage/semantics.html#the-html-element) |

### Document metadata and sectioning root

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `body` | Document body | Normal | `body []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/body) · [Google](https://web.dev/learn/html/document-structure) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-body-element) |
| `head` | Document metadata | Normal | `head []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/head) · [Google](https://web.dev/learn/html/document-structure) · [WHATWG](https://html.spec.whatwg.org/multipage/semantics.html#the-head-element) |

### Web components

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `slot` | Web component insertion point | Normal | `slot "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/slot) · [Google](https://web.dev/learn/html/template) · [WHATWG](https://html.spec.whatwg.org/multipage/scripting.html#the-slot-element) |
| `template` | Inert reusable template | Normal | `template "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template) · [Google](https://web.dev/learn/html/template) · [WHATWG](https://html.spec.whatwg.org/multipage/scripting.html#the-template-element) |

### Demarcating edits

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `del` | Deleted text | Normal | `del "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/del) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/edits.html#the-del-element) |
| `ins` | Inserted text | Normal | `ins "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ins) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/edits.html#the-ins-element) |

### Embedded content

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `embed` | External embedded content | Void | `embed #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/embed) · [Google](https://web.dev/learn/html/apis) · [WHATWG](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-embed-element) |
| `iframe` | Nested browsing context | Normal | `iframe "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/iframe) · [Google](https://web.dev/learn/html/apis) · [WHATWG](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element) |
| `object` | External resource | Normal | `object "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/object) · [Google](https://web.dev/learn/html/apis) · [WHATWG](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-object-element) |
| `picture` | Responsive image container | Normal | `picture []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/picture) · [Google](https://web.dev/learn/html/images) · [WHATWG](https://html.spec.whatwg.org/multipage/embedded-content.html#the-picture-element) |
| `source` | Alternative media source | Void | `source #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/source) · [Google](https://web.dev/learn/html/images) · [WHATWG](https://html.spec.whatwg.org/multipage/embedded-content.html#the-source-element) |

### Forms

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `button` | Action button | Normal | `button "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/button) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-button-element) |
| `datalist` | Suggested input options | Normal | `datalist []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/datalist) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-datalist-element) |
| `fieldset` | Form control group | Normal | `fieldset "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/fieldset) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-fieldset-element) |
| `form` | Submission form | Normal | `form "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/form) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/forms.html#the-form-element) |
| `input` | Input control | Void | `input #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/week) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/input.html#the-input-element) |
| `label` | Control label | Normal | `label "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/label) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/forms.html#the-label-element) |
| `legend` | Fieldset caption | Normal | `legend "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/legend) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-legend-element) |
| `meter` | Scalar measurement | Normal | `meter "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meter) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-meter-element) |
| `optgroup` | Option group | Normal | `optgroup []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/optgroup) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-optgroup-element) |
| `option` | Selectable option | Normal | `option "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/option) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-option-element) |
| `output` | Calculation result | Normal | `output "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/output) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-output-element) |
| `progress` | Task progress | Normal | `progress "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/progress) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-progress-element) |
| `select` | Selection control | Normal | `select []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/select) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-select-element) |
| `selectedcontent` | Selected option content in a customizable select | Normal | `selectedcontent []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/selectedcontent) · [Google](https://developer.chrome.com/blog/a-customizable-select) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-selectedcontent-element) |
| `textarea` | Multiline text control | Normal | `textarea "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/textarea) · [Google](https://web.dev/learn/html/forms) · [WHATWG](https://html.spec.whatwg.org/multipage/form-elements.html#the-textarea-element) |

### Inline text semantics

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `a` | Hyperlink | Normal | `a "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a) · [Google](https://web.dev/learn/html/links) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-a-element) |
| `abbr` | Abbreviation | Normal | `abbr "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/abbr) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-abbr-element) |
| `b` | Attention text | Normal | `b "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/b) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-b-element) |
| `bdi` | Bidirectional isolation | Normal | `bdi "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/bdi) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-bdi-element) |
| `bdo` | Direction override | Normal | `bdo "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/bdo) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-bdo-element) |
| `br` | Line break | Void | `br #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/br) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-br-element) |
| `cite` | Work title | Normal | `cite "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/cite) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-cite-element) |
| `code` | Code fragment | Normal | `code "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/code) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-code-element) |
| `data` | Machine readable value | Normal | `data "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/data) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-data-element) |
| `dfn` | Defined term | Normal | `dfn "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dfn) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-dfn-element) |
| `em` | Stress emphasis | Normal | `em "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/em) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-em-element) |
| `i` | Alternate voice | Normal | `i "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/i) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-i-element) |
| `kbd` | Keyboard input | Normal | `kbd "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/kbd) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-kbd-element) |
| `mark` | Highlighted text | Normal | `mark "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/mark) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-mark-element) |
| `q` | Inline quotation | Normal | `q "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/q) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-q-element) |
| `s` | No longer accurate text | Normal | `s "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/s) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-s-element) |
| `rp` | Ruby fallback | Normal | `rp "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/rp) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-rp-element) |
| `rt` | Ruby annotation | Normal | `rt "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/rt) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-rt-element) |
| `ruby` | Ruby annotation group | Normal | `ruby "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ruby) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-ruby-element) |
| `samp` | Sample output | Normal | `samp "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/samp) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-samp-element) |
| `small` | Side comment | Normal | `small "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/small) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-small-element) |
| `span` | Generic inline container | Normal | `span "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/span) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-span-element) |
| `strong` | Strong importance | Normal | `strong "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/strong) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-strong-element) |
| `sub` | Subscript | Normal | `sub "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/sub) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-sub-and-sup-elements) |
| `sup` | Superscript | Normal | `sup "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/sup) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-sub-and-sup-elements) |
| `time` | Date or time | Normal | `time "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/time) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-time-element) |
| `u` | Non textual annotation | Normal | `u "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/u) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-u-element) |
| `var` | Variable | Normal | `var "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/var) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-var-element) |
| `wbr` | Optional line break | Void | `wbr #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/wbr) · [Google](https://web.dev/learn/html/inline-text) · [WHATWG](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-wbr-element) |

### Interactive elements

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `details` | Disclosure widget | Normal | `details "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/details) · [Google](https://web.dev/learn/html/details) · [WHATWG](https://html.spec.whatwg.org/multipage/interactive-elements.html#the-details-element) |
| `dialog` | Dialog | Normal | `dialog "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog) · [Google](https://web.dev/learn/html/dialog) · [WHATWG](https://html.spec.whatwg.org/multipage/interactive-elements.html#the-dialog-element) |
| `geolocation` | Location sharing control (experimental) | Normal | `geolocation []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/geolocation) · [Google](https://developer.chrome.com/blog/geolocation-html-element) · WHATWG: — |
| `install` | Web app installation control (experimental) | Normal | `install []` | MDN: — · [Google](https://developer.chrome.com/blog/install-element-ot) · WHATWG: — |
| `permission` | Generic permission control (superseded experimental proposal) | Normal | `permission.type: "camera microphone" []` | MDN: — · [Google](https://developer.chrome.com/blog/geolocation-html-element#from-generic-permission-to-specific-geolocation) · WHATWG: — |
| `summary` | Disclosure heading | Normal | `summary "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/summary) · [Google](https://web.dev/learn/html/details) · [WHATWG](https://html.spec.whatwg.org/multipage/interactive-elements.html#the-summary-element) |
| `usermedia` | Camera and microphone control (experimental) | Normal | `usermedia []` | MDN: — · [Google](https://developer.chrome.com/blog/usermedia-html-element) · WHATWG: — |

The earlier experimental `permission` proposal evolved into the specialized
`geolocation` and `usermedia` controls. Its helper is provided for experiments
with implementations of that proposal; emitting the element does not enable browser
support. See [Chrome's capability elements background](https://developer.chrome.com/blog/geolocation-html-element).

### Document metadata

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `base` | Base URL | Void | `base #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/base) · [Google](https://web.dev/learn/html/metadata) · [WHATWG](https://html.spec.whatwg.org/multipage/semantics.html#the-base-element) |
| `link` | External resource relationship | Void | `link #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/link) · [Google](https://web.dev/learn/html/metadata) · [WHATWG](https://html.spec.whatwg.org/multipage/semantics.html#the-link-element) |
| `meta` | Metadata | Void | `meta #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta/http-equiv) · [Google](https://web.dev/learn/html/metadata) · [WHATWG](https://html.spec.whatwg.org/multipage/semantics.html#the-meta-element) |
| `title` | Document title | Normal | `title "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/title) · [Google](https://web.dev/learn/html/metadata) · [WHATWG](https://html.spec.whatwg.org/multipage/semantics.html#the-title-element) |
| `style` | Trusted stylesheet | Trusted | `style [raw "/* trusted */"]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/style) · [Google](https://web.dev/learn/html/metadata) · [WHATWG](https://html.spec.whatwg.org/multipage/semantics.html#the-style-element) |

### Image and multimedia

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `area` | Image map region | Void | `area #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/area) · [Google](https://web.dev/learn/html/images) · [WHATWG](https://html.spec.whatwg.org/multipage/image-maps.html#the-area-element) |
| `audio` | Audio player | Normal | `audio "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/audio) · [Google](https://web.dev/learn/html/audio-video) · [WHATWG](https://html.spec.whatwg.org/multipage/media.html#the-audio-element) |
| `img` | Image | Void | `img #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img) · [Google](https://web.dev/learn/html/images) · [WHATWG](https://html.spec.whatwg.org/multipage/embedded-content.html#the-img-element) |
| `map` | Image map | Normal | `map "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/map) · [Google](https://web.dev/learn/html/images) · [WHATWG](https://html.spec.whatwg.org/multipage/image-maps.html#the-map-element) |
| `track` | Timed text track | Void | `track #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/track) · [Google](https://web.dev/learn/html/audio-video) · [WHATWG](https://html.spec.whatwg.org/multipage/media.html#the-track-element) |
| `video` | Video player | Normal | `video "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/video) · [Google](https://web.dev/learn/html/audio-video) · [WHATWG](https://html.spec.whatwg.org/multipage/media.html#the-video-element) |

### Scripting

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `canvas` | Script drawing surface | Normal | `canvas "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/canvas) · [Google](https://web.dev/learn/html/apis) · [WHATWG](https://html.spec.whatwg.org/multipage/canvas.html#the-canvas-element) |
| `noscript` | Scripting fallback | Normal | `noscript "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/noscript) · [Google](https://web.dev/learn/html/apis) · [WHATWG](https://html.spec.whatwg.org/multipage/scripting.html#the-noscript-element) |
| `script` | Trusted script | Trusted | `script [raw "/* trusted */"]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script/type/speculationrules) · [Google](https://web.dev/learn/html/apis) · [WHATWG](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element) |

### Content sectioning

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `address` | Contact information | Normal | `address "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/address) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-address-element) |
| `article` | Self contained composition | Normal | `article "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/article) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-article-element) |
| `aside` | Related content | Normal | `aside "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/aside) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-aside-element) |
| `footer` | Section footer | Normal | `footer "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/footer) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-footer-element) |
| `header` | Section introduction | Normal | `header "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/header) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-header-element) |
| `h1` | Level 1 heading | Normal | `h1 "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements) |
| `h2` | Level 2 heading | Normal | `h2 "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements) |
| `h3` | Level 3 heading | Normal | `h3 "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements) |
| `h4` | Level 4 heading | Normal | `h4 "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements) |
| `h5` | Level 5 heading | Normal | `h5 "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements) |
| `h6` | Level 6 heading | Normal | `h6 "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements) |
| `hgroup` | Heading group | Normal | `hgroup []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/hgroup) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-hgroup-element) |
| `main` | Main content | Normal | `main "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/main) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-main-element) |
| `nav` | Navigation | Normal | `nav "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/nav) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-nav-element) |
| `section` | Thematic section | Normal | `section "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/section) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/sections.html#the-section-element) |
| `search` | Search region | Normal | `search "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/search) · [Google](https://web.dev/learn/html/headings-and-sections) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-search-element) |

### Table content

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `caption` | Table caption | Normal | `caption "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/caption) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-caption-element) |
| `col` | Table column | Void | `col #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/col) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-col-element) |
| `colgroup` | Table column group | Normal | `colgroup []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/colgroup) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-colgroup-element) |
| `table` | Data table | Normal | `table []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/table) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-table-element) |
| `tbody` | Table body | Normal | `tbody []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tbody) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-tbody-element) |
| `td` | Table data cell | Normal | `td "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/td) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-td-element) |
| `tfoot` | Table footer | Normal | `tfoot []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tfoot) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-tfoot-element) |
| `th` | Table header cell | Normal | `th "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/th) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-th-element) |
| `thead` | Table header | Normal | `thead []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/thead) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-thead-element) |
| `tr` | Table row | Normal | `tr []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tr) · [Google](https://web.dev/learn/html/tables) · [WHATWG](https://html.spec.whatwg.org/multipage/tables.html#the-tr-element) |

### Text content

| HTML Element | Purpose | Content | Example | References |
| --- | --- | --- | --- | --- |
| `blockquote` | Block quotation | Normal | `blockquote "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/blockquote) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-blockquote-element) |
| `dd` | Description details | Normal | `dd "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dd) · [Google](https://web.dev/learn/html/lists) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-dd-element) |
| `div` | Generic flow container | Normal | `div "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/div) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-div-element) |
| `dl` | Description list | Normal | `dl []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dl) · [Google](https://web.dev/learn/html/lists) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-dl-element) |
| `dt` | Description term | Normal | `dt "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dt) · [Google](https://web.dev/learn/html/lists) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-dt-element) |
| `figcaption` | Figure caption | Normal | `figcaption "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/figcaption) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-figcaption-element) |
| `figure` | Self contained figure | Normal | `figure "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/figure) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-figure-element) |
| `hr` | Thematic break | Void | `hr #[]` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/hr) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-hr-element) |
| `li` | List item | Normal | `li "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/li) · [Google](https://web.dev/learn/html/lists) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-li-element) |
| `menu` | Unordered command list | Normal | `menu []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/menu) · [Google](https://web.dev/learn/html/lists) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-menu-element) |
| `ol` | Ordered list | Normal | `ol []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ol) · [Google](https://web.dev/learn/html/lists) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-ol-element) |
| `p` | Paragraph | Normal | `p "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/p) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-p-element) |
| `pre` | Preformatted text | Normal | `pre "Text"` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/pre) · [Google](https://web.dev/learn/html/text-basics) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-pre-element) |
| `ul` | Unordered list | Normal | `ul []` | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul) · [Google](https://web.dev/learn/html/lists) · [WHATWG](https://html.spec.whatwg.org/multipage/grouping-content.html#the-ul-element) |
