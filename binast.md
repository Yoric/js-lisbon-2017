# BinAST

David Teller/Yoric, October 2017


---

## What it's about

- Optimizing first byte sent to execution start.
- Designed for multi-megabyte pages.
- Project: mozilla + Facebook + ...

---

### Why?

- Google Sheets, Google Docs, Yahoo!, LinkedIn, Facebook: 3-7Mb+ JS code.
- Updated very often.
- Facebook: 500-900ms JS parsing (Chrome & Firefox).

---

### About jank

- 16ms- (sustained) Smooth
- 40ms+ (sustained) Choppy
- 50ms+ (single) Jank
- 100ms+ (single) Broken ⤶
- 10s+ (single) Stopped responding

---

### JS source text is an inefficient medium

- ambiguous/complex (⇒ slow tokenization/parsing);
- flat + `SyntaxError` semantics (⇒ cannot skip/parallelize);
- repeated (⇒ minimizers).

---

### Can we do better?

Experiments with a binary format suggest we can have:
- 1:1 match with source text;
- parsing speed 3x;
- lazy parsing speed 8x;
- smaller payload than minimized code.

**Not** actual benchmarks.

---

## Alternatives

---

### Minimizers

  - ⊕ simple for web devs;
  - ⊖ not very good;
  - ⊖ kills readability/debuggability.

---

### Lazy loading

  - ⊕ better startup time;
  - ⊖ hard for web devs;
  - ⊖ quickly makes things worse.

---

###  Syntax parsing

  - ⊕ transparent for web devs;
  - ⊖ not good enough.

---

### Google

"Prefetch & compile your code from ServiceWorkers."

- ⊕ simple for **browser** devs
- ⊖ bandwidth, battery, server energy, memory cost;
- ⊖ really complex for web devs.


---

## Towards (binary) AST

---
### The AST

- future compatible?
- size efficient?
- speed efficient?

---
### Future compatibility

```js
function foo() {
    // ...
}
```

```rust
FunctionDeclaration {
  name: Identifier("foo"),
  body: ...,
  args: [],
  async: false,     // ES6
  generator: false, // ES6
}
```

---

### Future compatibility (2)

```typescript
// specs
interface FunctionDeclaration {
    name: Identifier;
    body: BlockStatement;
    args: [Arguments];
    async: bool = false;
    generator: bool = false;
}
```

```js
// header
"FunctionDeclaration": ["name", "body", "args"]
```


---

### Speed efficiency (1)

- No string sniffing.
- Relax `SyntaxError` to allow skipping functions.

---

### Speed efficiency (2)

- Don't **build** proofs, **check** them:

```js
Scope {
    const: ["x", "y"],
    let: ["a"],
    var: [],
    captured: ["z"],
    hasDirectEval: false,
}
```

---

## Towards a binary (AST)

---

### Binary?

- future compatible?
- size efficient?
- speed efficient?

---

### Layout

1. header
2. grammar table;
3. atoms table;
4. tree(s);
5. comments? (TBD)

---

### Size efficiency

- do not repeat strings/atoms/identifiers/properties;
- use grammar table as a compression mechanism
- compress everything.


---

### Speed efficiency (2)

- atoms table may be parsed early/concurrently;
- grammar table may be rejected early;
- (some) subtrees may be skipped/parallelized.

---

## Conclusions

---

### Status

- TC-39: stage 1.
- Reference implementation: iterating.
- SpiderMonkey: almost ready to land.
- Size: not satisfying yet :/
- Speed: cannot measure yet.
- Safety: iterating.

---

### Next steps

- Land JS shell-only version.
- Well-formed AST.
- Measure speed.
- Bind to DOM.
- Limited opt-in test with Facebook.



---

### Want to join?

- IRC: `#binjs`
- Newsgroup/mailing-list: `dev-binast`