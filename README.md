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

Lifting the restriction raises the question of how to handle cooked template values that contain illegal escape sequences. Currently, cooked template values are supposed to replace escape sequences with the "Unicode code point represented by the escape sequence" but this can't happen if the escape sequence is not valid.

One solution is to set the cooked value to `undefined` for template values that contain illegal escape sequences. The raw value is still accessible via `.raw` so embedded DSLs that might contain `undefined` cooked values can just use the raw string:

```js
function tag(strs) {
  strs[0] === undefined
  strs.raw[0] === "\\unicode and \\u{55}";
}
tag`\unicode and \u{55}`
```

Another potential solution would be to have valid escape sequences replaced but leave invalid escape sequences alone.

```js
function tag(strs) {
  strs[0] === "\\unicode and U"
  strs.raw[0] === "\\unicode and \\u{55}";
}
tag`\unicode and \u{55}`
```

This seems like a bad solution. Ostensibly, any DSL that needs `\unicode` or similar to be valid will not care about *any* escape sequences and will just use the raw value.

Having inconsistent escaping behavior is a potential footgun. If programmer is building a DSL that relies on `\unicode` cooking literally, they might be surprised to discover to find `\uface` is cooked into `龜`.

It seems like the best choice is just to set the cooked value to `undefined` when an invalid escape sequence is encountered and force DSLs writers who care about this to use the raw value.
