# HTML DSL API

Import with `import {src/html}!`. Supported names follow the official names discussed in [issue #2](https://github.com/RickBarretto/html.pkgr.art/issues/2). The inventory below is grouped by [MDN HTML element categories](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements); these are reference groups, not enforced content models.

## Entry points and qualified helpers

`fragment content` returns a compact escaped string. `html content` returns typed document markup with `<!DOCTYPE html>`. `body content` returns typed body markup. Both callable entry functions forward generic attributes. Every element accepts exactly one content argument: a string, typed markup, an eagerly evaluated child block, or an attribute dictionary for an empty element. Void elements still require that argument: use `[]`, `""`, or `#[]`.

`markup` is the public DSL object. `markup\p "Hello"`, `markup\element "my-widget" [span "Hello"]`, and `markup\raw "<b>Trusted</b>"` work outside DSL scopes. `markup\html []` includes the doctype; `markup\element "html" []` serializes only the element. `to :string node` renders typed markup. `fragment node` also renders it. Markup captures its content once; rendering it again does not rerun child blocks.

All fragment blocks and element child blocks bind official tag helpers plus `element` and `raw` locally. Official names take precedence over caller bindings and built-ins such as `div`, `map`, `select`, and `input`. Caller bindings and built-ins are restored when the scope returns. Save any needed built-in behavior under another name before entering the DSL. Assignments in DSL blocks follow function scope; use a shared collection for intentional observable effects.

```art
print fragment [
    body [
        h1.class: "xl" "Hello, World"
        article -> div.class: "flex-col" ["Children"]
    ]
]
```

`->` wraps one expression as a child block. Explicit blocks support multiple children, nested elements, loops, and conditionals. Null results are ignored; other children must be strings or typed markup.

```art
card: function [name][markup\article [h2 name]]
print fragment [
    div [
        "People: "
        loop ["Ana" "João"] 'person [card person]
        if true [p "Welcome"]
        if false [p "Hidden"]
    ]
]
```

## Attributes and trust

Use generic attributes on any helper: `.class: "xl"`, `.id: "intro"`, `.data_state: "ready"`, or `.aria_hidden: false`. Dictionary content supplies attributes for an empty element: `input #[name: "email" required: true]`. Dynamic attributes override dictionary values after names are lowercased and underscores become hyphens. Attributes are sorted by normalized name. Null values are omitted. HTML boolean attributes with logical true are bare and false are omitted; logical values on other attributes are quoted (`aria-hidden="false"`). Strings, literals, integers, floating values, and logical values are supported. Attributes do not introduce tag-specific shorthand or aliases.

Text and attribute values escape `&`, `<`, `>`, and double quotes. Nested typed markup is preserved without double escaping. `raw` bypasses escaping and requires trusted input. Script/style children must be empty or raw values:

```art
print fragment [
    style [raw "p { color: navy; }"]
    script [raw "console.log('Hello');"]
    element.data_state: "ready" "my-widget" [p "Hello <world>"]
]
```

The library does not sanitize JavaScript, CSS, or URLs. It rejects malformed tag/attribute names, unsupported children/attribute values, nonempty void content, and ordinary script/style strings.

## Migration and scope

Replace the old public object `html` with `markup`: `html\p` becomes `markup\p`, `html\element` becomes `markup\element`, and `html\raw` becomes `markup\raw`. `html` is now a callable document entry function. Prefer unqualified names inside child blocks. Legacy backslash shorthand is no longer the documented interface. Tag results are typed markup rather than plain strings. No aliases or specialized positional attribute arguments are provided.

Independent initialized objects can be created with `other: to :htmlDsl []!`; they expose the same qualified methods and scoped child evaluation. Helpers are generated from the same canonical inventory (`markup\tags`).

HTML4, obsolete elements (including `param`), SVG/MathML serialization, and HTML content-model validation remain excluded. Custom names serialize through `element`; this does not enable SVG/MathML semantics or validate custom element naming conventions beyond the generic name syntax.

## Complete tag reference

Examples below are child expressions inside `fragment [...]`. **Normal** means string, scoped child block, or attribute dictionary (empty content). **Void** means empty string/block or attribute dictionary, with no closing tag. **Trusted** means an empty value, raw value, block of raw values, or attribute dictionary. **Document** accepts the same forms as a normal element and adds the doctype.

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
