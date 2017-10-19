# BinAST

David Teller/Yoric, October 2017


---

## What it's about

- Optimizing first byte sent to execution start.
- In particular multi-megabyte pages.
- Project: mozilla + Facebook + ...

---

### Why?

- Google Sheets, Google Docs, Yahoo!, LinkedIn, Facebook: 3-7Mb+ JS code.
- Updated *very* often.
- Facebook: 500-900ms just building the JS AST (Chrome & Firefox).

---

### About jank

- 16ms- (sustained) Smooth
- 40ms+ (sustained) Choppy
- 50ms+ (single) Jank
- 100ms+ (single) Broken **⤶**
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

### Future compatibility (3)

Amending the standard:

- **add** a new virtual or concrete interface;
- **subclass** an existing interface with a new interface;
- **extend** an interface to add with new fields;
- **generalize** a sum of interfaces to add more interfaces;
- **extend** an enum with new cases.

This assumes that the language can only grow, without backwards-incompatible syntax changes.

---

### Speed efficiency (1)

- No string sniffing.
- Delay `SyntaxError` to function execution (allows lazy/concurrent parsing).

---

### Speed efficiency (2)

- Don't **build** proofs, **check** them.
- Remove reasons to need to walk subtrees to collect all data.
- e.g. scope:
```js
Scope {
    // Variables declared in this scope.
    const: ["x", "y"],
    let: ["a"],
    var: [],

    // Captured by some inner function.
    captured: ["x"],

    // Cannot optimize due to `eval` in the subtree.
    hasDirectEval: false,
}
```
- e.g. tree wellf-formedness.

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
5. comments, sourcemaps? (TBD)


---


### Size efficiency

- do not repeat strings/atoms/identifiers/properties;
- use grammar table as a compression mechanism;
- compress everything.


---

### Speed efficiency (2)

- atoms table may be parsed/checked/interned early/concurrently;
- identifiers may be checked a single time;
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