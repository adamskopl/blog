# Color themes with CSS-variables

<img src="https://raw.githubusercontent.com/adamskopl/blog/posts/20200703/cover.png">

## The problem

- You're writing HTML and its default styles.
- How could You organize a code, so that it's easy to expand styles with color themes? Without changing original HTML or CSS every time.

## Color Themes demo using CSS variables

It appears it's easy to do it, by using [CSS variables](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties). I prepared a simple demo. Let's review it, step by step.

### 1. Basic Setup

```html
<div id="main">
  <div class="someBox">
    SOME BOX
  </div>
</div>
```
```css
body {
  margin: 0;
}

#main {
    width: 100vw;
    height: 100vh;

    display: flex;
    justify-content: center;
    align-items: center;

    background-color: white;
}

.someBox {
    width: 30vh;
    height: 30vh;
    background-color: red;
    color: black;
}
```
[CodePen](https://codepen.io/adamskopl/pen/ZEQvmgV)

- We've added the main area containing a whole space.
- There's a red box in the center.
- For now, there are 3 main colors: white, black, red.
  - These will be colors that will be changed when changing color themes.

### 2. Adding theme selection

```html
<div class="themesControls">
  <input type="radio" name="themesInput" id="themeDefault"> <label for="themeDefault">default</label>
  <input type="radio" name="themesInput" id="themeOne"> <label for="themeOne">One</label>
  <input type="radio" name="themesInput" id="themeTwo"> <label for="themeTwo">Two</label>
</div>
```
```css
.themesControls {
  position: fixed;
}

label {
  color: black;
}
```
```js
Array.from(document.getElementsByTagName("input")).forEach(
  addEventListenerSettingThemeClass
);

function addEventListenerSettingThemeClass(el) {
  el.addEventListener("change", function (e) {
    alert(`choice: ${e.target.id}`);
  });
}
```
[CodePen](https://codepen.io/adamskopl/details/XWXVojv)

- We've added 3 radio buttons for future themes selection. Radio buttons change is handled in Javascript.
- Buttons' container has a fixed position so that it will not interfere with a layout.
- Buttons' labels are using one of the 3 main colors.

### 3. Adding 'theme' class to the 'body' element

```html
  <input type="radio" name="themesInput" id="themeDefault" checked> <label for="themeDefault">default</label>
```
```js
Array.from(document.getElementsByTagName("input")).forEach(
  addEventListenerSettingThemeClass
);

Array.from(document.getElementsByTagName("input")).forEach(
  setBodyIfElementChecked
);

function setBodyIfElementChecked(el) {
  if (el.checked) {
    clearClass(getBody());
    addClass(getBody(), el.id);
  }
}

function addEventListenerSettingThemeClass(el) {
  el.addEventListener("change", function (e) {
    const eventEl = e.target;
    clearClass(getBody());
    addClass(getBody(), eventEl.id);
  });
}

function getBody() {
  return document.getElementsByTagName("body")[0];
}

function clearClass(el) {
  el.className = "";
}

function addClass(el, className) {
  el.classList.add(className);
}
```
[CodePen](https://codepen.io/adamskopl/pen/zYrpyJr)

- The way of applying color themes will be by adding 'theme' classes like 'themeOne' to the `<body>` element.
- A listener for a button's change adds a given color theme to the body (`addEventListenerSettingThemeClass`). Theme's class name is the same as an input's id.
- One of the inputs is set to be 'checked'. So that there's a checked input at the beginning, thus theme being chosen.
  - In the beginning, the checked input will set the initial color theme with `setBodyIfElementChecked` (by adding mentioned theme class).

### 4. Using CSS variables

```css
body {
  --primaryColor: white;
  --secondaryColor: black;
  --teritaryColor: red;
}

#main {
  background-color: var(--primaryColor);
}

label {
  color: var(--secondaryColor);
}

.someBox {
  background-color: var(--teritaryColor);
  color: var(--secondaryColor);
}

/* COLOR THEMES SECTION */

body.themeOne {
  --primaryColor: yellow;
  --secondaryColor: orange;
  --teritaryColor: purple;
}

body.themeTwo {
  --primaryColor: pink;
  --secondaryColor: blue;
  --teritaryColor: lime;
}
```
[CodePen](https://codepen.io/adamskopl/pen/JjGMmZL)

- First, we're declaring 3 themes variables with their initial values. They should be available in the whole `<body>`, hence the place of declaration. From now on, `--primaryColor`, `--secondaryColor` and `teritaryColor` can be used instead of values.
- We're using these 3 variables in `#main` `label` and `.someBox` by using `var(--variableName)` syntax. At this point nothing is changed: we've only replaced values with variables.
- CSS variables don't have fixed values and can be changed, as seen in the `/*COLOR THEMES SECTION*/`.
  - `<body>` combined with given theme class changes colors values used in our demo. Hence `body.themeOne` has a higher [specificity](http://www.standardista.com/css3/css-specificity/), than `body`, values from `body.themeOne` are used.
  - Notice, that to add another color theme, only a new `body.themeX` definition has to be added. No changes in original CSS and HTML required.
- Voilà! :art:
