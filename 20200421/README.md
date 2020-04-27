# Thoughts on code readability

How big a deal a code readability is?

## What kind of 'readability'?

Code readability can have different definitions. It could be, for an example, a
code hard to follow. Maybe because of hard to understand nested `if else`
clauses. Or because of a lack of dividing code into functions, which names would
be more explanatory. Such code can be understood, but requires focused analysis.
Its readability can be achieved by code refactoring.

But there's also another type of non readable code. It's a code that **fails to
 explain what it does**.

## Reasons

From some time I'm quite sceptic about the statement, that "one of the benefits
of unit tests is code documentation". That "if you don't know 'whats' and 'whys'
of the code, check unit tests for answers". I'm not 100% sceptic: securing
requirements with tests is a good idea to some degree. Anyway: what bothers me,
is the tendency of not caring enough about a code being self explanatory.
Passing that responsibility on unit tests is one of excuses.

Problem is often visible, when working with legacy code. We see a fragment and
it's totally not clear, what's its purpose. So we leave it alone, because we
don't want to mess something up. Besides, people usually don't want to improve
code's readability in legacy code. "I don't have to understand it, someone
probably had a good reason to write it". And a mysterious fragment of a code
lives forever happily after.

Bad code reviews also go hand in hand with producing non readable code. From my
experience, people very often only check, if variables have good names, if
solution is written accordingly to framework's standards, or if it can be
written more clever. They don't pay enough attention, to understand what the
code actually does, they just want to check if it looks good. After that, code
review is marked as 'accepted' and an author of a solution merges it, missing
the last moment for future confusion prevention.

## Example

Some time ago, I've encountered the situation, when during refactoring and
writing missing tests, the non readable code caused the need of an addition
investigation. It was a file's 'dragover' event handler and it performed an
initial check, for dragged file's validity, displaying either a success or an
error. It went something like this:

```js
function someFileDragOverHandler(event) {
  if (event.dataTransfer) {
    if (event.dataTransfer.items && event.dataTransfer.items.length > 0) {
      const draggeedItem = event.dataTransfer.items[0];
      if (isItemOk(draggeedItem)) {
        showSuccess();
      } else {
        showError();
      }
    } else {
      showSuccess();
    }
  }
}
```

I wanted to write unit tests, which would check if `showSuccess()` or
`showError()` are called properly. But first, I had to understand **when**
success or error should be displayed. Basing only on the code, the understanding
should go like this:
- if `event.dataTransfer` is falsy, show neither error, nor success
- if `event.dataTransfer.items` is falsy or has no elements, show success
- otherwise perform check on an item, call either showSuccess() or showError()

I don't remember why, but there was a reason for that particular situation to
check `dataTransfer`. If not available, something should happen, though. Right?
**Where is the 'else' clause?** If something is checked, then obviously it could
happen. What then? In this situation nothing would happen: user would get
neither success nor error.

Why `event.dataTransfer.items` and `event.dataTransfer.items.length` are checked? If a
check failes, why the 'success' is displayed? It turned out, that
[DataTransferItemList](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API/File_drag_and_drop)
is not supported everywhere. More precisely (surprise, surprise) it's not
supported by IE. Project required IE support, so `items` property had to be
checked. Additionaly `items` [could
have](https://developer.mozilla.org/en-US/docs/Web/API/DataTransferItemList)
length of 0. Displaying 'success' was a fallback, assuming that later parsing
code would handle a file being invalid.

## Improving redability

So the code was not readable, because:
- There was no `else` for `dataTransfer` check.
- When it comes to `items`, it could be not obvious, that it's a browser support
  that is checked and not something else.

I refactored the code a little bit, leaving it, in my opinion, more readable:

```js
 function someFileDragOverHandler(event) {
  const dataTransferItems = event.dataTransfer.items;
  const dataTransferItemListSupported =
        (dataTransferItems !== undefined && dataTransferItems.length > 0);
  const draggedItemVaild = dataTransferItemListSupported && isItemValid(dataTransferItems[0]);

  if (draggedItemVaild || !dataTransferItemListSupported) {
    showSuccess();
  } else {
    showError();
  }
}
```

- code could be shorter, but I wanted to show explicitly, that
  `DataTransferItemList` support should be checked
- conditions for showing a success or an error are short and quick to understand

## "Good grief, why there's no 'else' clause?!"

> (...) Good grief, why there's no 'else' clause?! Maybe that's the source of the bug?
> 
> (...) Code is human communication.
> 
> Kyle Simpson [Keep Betting on JavaScript by Kyle Simpson · JSCamp
Barcelona 2018](https://www.youtube.com/watch?v=lDLQA6lQSFg).

Through that simple example, I wanted to demonstrate, how an unreadable code:
- adds an extra time, needed to understand it,
- increases complexity,
- doesn't explain what it actually does,
- is prone to generate errors
