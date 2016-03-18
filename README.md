# Template Literals Are Broken

## Introduction

Template literals should allow the embedding of languages (DSLs etc.). But restrictions on escape sequences make this problematic.

For example, consider making a latex processor with templates:

```js
function latex(strings) {
  // ...
}

let document = latex`
\newcommand{\fun}{\textbf{Fun!}}  // works just fine
\newcommand{\unicode}{\textbf{Unicode!}} // Illegal token!
\newcommand{\xerxes}{\textbf{King!}} // Illegal token!

Breve over the h goes \u{h}ere // Illegal token!
`
```

A Unicode escape like `\u00FF` is allowed but `\unicode` is illegal.

A Hex escape like `\xFF` is allowed but `\xerxes` is illagal.

Octal literal escapes are not allowed.

```js
`bin = \0100` // Illegal token!
```

## Proposal

Remove the restriction on escape sequences.

But then what about the cooked strings?

```js
function tag(strs) {
  // proposed change
  strs[0] === undefined
}
tag`\unicode`
```

## Future

Better API?
