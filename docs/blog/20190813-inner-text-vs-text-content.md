# innerText vs textContent

There are some
[interesting differences](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent#Differences_from_innerText)
between the two.
Most noticeable IMO that `innerText` triggers a reflow, thus, whenever possible use `textContent`.
And I found a surprising one on top:

```javascript
someNode.innerText =
    `some multi
     line ${ true ? "text" : void }
     !`;
```

Expecting the content of `someNode` to have three lines, maybe with line breaks in the code,
but to render as simple text in one line - ignoring line breaks as usual.

[But wow, try it!](/tests/playground.html#inner-text-vs-text-content.html)

Turns out, line breaks get converted to `<br>`!
Very unexpected... and IMO very wrong!
But `textContent` to the rescue!
It behaves as expected: line breaks yes, `<br>` tags no.
Another reason for `textContent`.

**The final results for `innerText` vs `textContent` is 0:2 - `textContent` the clear winner of this match.**
