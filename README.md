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

The problem here is that `\u` is the start of a unicode escape, but ES grammar forces it to be of the form `\u00FF` or `\u{42}`
and considers the token `\unicode` illegal.
Similarly `\x` is the start of a hex escape like `\xFF` but `\xerxes` is illegal. Octal literal escapes have the same problem; `\0100` is illegal.

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
