# Thoughts on code readability

## Unit tests as documentation?

From some time I'm quite sceptic about the statement, that "one of the benefits
of unit tests is code documentation". That "if you don't know 'whats' and 'whys'
of the code, check unit tests for answers". I'm not 100% sceptic: securing
requirements with tests is a good idea to some degree. Anyway: what bothers me,
is the tendency of not caring enough about a code readability. Passing that
responsibility on unit tests is one of excuses.

## Legacy code

Problem is often visible, when working with legacy code. We see a fragment and
it's totally not clear, what's its purpose. So we leave it alone, because we
don't want to mess something up. Besides, people usually don't want to improve
code's readability in legacy code. 'I don't have to understand it, someone
probably had a good reason to write it'. And a mysterious fragment of a code
lives forever happily after.

## Weak code reviews

Bad code reviews also go hand in hand with producing non readable code. From my
experience, people very often only check, if variables have good names, if
solution is written accordingly to framework's standards, or if it can be
written more clever. They don't pay enough attention, to understand what the
code actually does, they just want to check if it looks good. After that, code
review is marked as 'accepted' and an author of a solution is more than happy,
to merge it, and no one 

## Some example?

Some time ago, I've encountered the situation, when during refactoring and
writing missing tests, the non readable code caused some trouble. It went
something like this:

```js
function someFileDragOverHandler(userChoice, event) {
  if (event.dataTransfer) {
    if (event.dataTransfer.items && event.dataTransfer.items.length > 0) {
      const type = event.dataTransfer.items[0].type;
      if (userChoice === 'A' && type !== 'text/html') {
        showError();
      } else if (userChoice === 'B' && type !== 'image/jpeg') {
        showError();
      } else if (type !== 'text/html' && type !== 'image/jpeg') {
        showSuccess();
      }
    }
  } else {
    showSuccess();
  }
}
```

I wanted to understand what exactly is going on. Two things caused confusion:
- ```if else``` clauses checking types
- when the ```showSuccess()``` and ```showError()``` should be called?

### the first 'if'

I wanted to write unit test, checking if ```showSuccess()``` is called properly.
Why it was called, when ```event.dataTransfer``` is falsy? And that's another
thing. It could be ```null```, ```undefined```, ```0```? How someone should
know? Why no direct comparison to e.g. ```undefined```? That produces questions
already! And ```dataTransfer``` is available always... why would someone check
something, which will not ever happen and call ```showSuccess()```? Success on
something, that at first glance seems like an error?

### types checking

