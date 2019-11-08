# Do..while extended lexical scope

Include the condition inside of `while(...)` expression into the lexical scope of `do {...}` block in a manner that allows to white in following manner:
```javascript
do {
  const shouldQuit = getShouldIQuit();
} while (!shouldQuit)
```

## Status

Author(s): Vsevolod Rodionov

Stage: -1

## Motivation

*Why is this important to have in the JavaScript language?*

Unlike `for()`, `for...of` and `for...in` blocks, who are iterating over known entity, `while` and `do...while` blocks are commonly used when break condition is unclear.
Often the break condition is actually defined inside the block; however, there is no common way to declare the exit condition that is defined inside the block clearly.
2 most common approaches are:

```javascript
let shouldQuit = false;
do {
  shouldQuit = getShouldIQuit()
} while (!shouldQuit)
```
and
```javascript
while (true) {
  const shouldQuit = getShouldIQuit();
  if (shouldQuit) {
    break;
  }
}
```

Both of them have issues:

Option 1 is mutating the external state, is verbose and is can pollute the higher-level lexical scope with variable only needed for inner scope. This is still declarative approach: code states, that it has a computation block, and it has an break post-condition.

Option 2 has 3 issues:
- despite the fact that it has post-condition, it is declared in pre-condition manner (`while () {}` instead of `do {} while ()`)
- it is using `break`. Usage of this statement is controversial and discussed a lot, e.g.: 
[1](https://softwareengineering.stackexchange.com/questions/58237/are-break-and-continue-bad-programming-practices), 
[2](https://stackoverflow.com/questions/3922599/is-it-a-bad-practice-to-use-break-in-a-for-loop), 
[3](https://www.quora.com/Is-there-anything-bad-about-using-a-while-true-loop-and-exiting-it-using-a-break-statement), 
[4](https://www.reddit.com/r/learnjava/comments/3mc1lb/why_is_it_bad_practice_to_use_the_break_or/)
- its approach us redundantly imperative: it increases cognitive complexity; instead of clearly stated break condition the one who sees this block for a first time has to look though it to discover the `break` control flow statements

## Use cases

[angular](https://github.com/angular/angular/blob/5d86e4a9b166f9a9b9f521b9fa8a83f1187d76df/packages/compiler/src/ml_parser/lexer.ts#L418-L426)
```javascript
// before
while (true) {
  const tagCloseStart = this._cursor.clone();
  const foundEndMarker = endMarkerPredicate();
  this._cursor = tagCloseStart;
  if (foundEndMarker) {
    break;
  }
  parts.push(this._readChar(decodeEntities));
}
// after
do {
  const tagCloseStart = this._cursor.clone();
  const foundEndMarker = endMarkerPredicate();
  this._cursor = tagCloseStart;
  if (!foundEndMarker) {
    parts.push(this._readChar(decodeEntities));
  }
} while (!foundEndMarker)
```

[typescript](https://github.com/microsoft/TypeScript/blob/d7c83f023eeb16027c5942b8156207e9f068b367/src/typingsInstallerCore/typingsInstaller.ts#L48-L55)
```typescript
// before
while (true) {
    command = `${npmPath} install --ignore-scripts ${(toSlice === packageNames.length ? packageNames : packageNames.slice(sliceStart, sliceStart + toSlice)).join(" ")} --save-dev --user-agent="typesInstaller/${tsVersion}"`;
    if (command.length < 8000) {
        break;
    }

    toSlice = toSlice - Math.floor(toSlice / 2);
}
// after
do {
    command = `${npmPath} install --ignore-scripts ${(toSlice === packageNames.length ? packageNames : packageNames.slice(sliceStart, sliceStart + toSlice)).join(" ")} --save-dev --user-agent="typesInstaller/${tsVersion}"`;
    toSlice = toSlice - Math.floor(toSlice / 2);
} while (command.length >= 8000)
```
[yargs-parser](https://github.com/yargs/yargs-parser/blob/master/index.js#L922-L938)
```javascript
// before
var change = true
while (change) {
  change = false
  for (var i = 0; i < aliasArrays.length; i++) {
    for (var ii = i + 1; ii < aliasArrays.length; ii++) {
      if (intersect.length) {
        change = true
      }
    }
  }
}
// after
do {
  let change = false;
  for (var i = 0; i < aliasArrays.length; i++) {
    for (var ii = i + 1; ii < aliasArrays.length; ii++) {
      if (intersect.length) {
        change = true
      }
    }
  }
} while (change)
```

## Implementations

### Polyfill/transpiler implementations

Babel plugin for this proposal can be implemented in a short period of time.
