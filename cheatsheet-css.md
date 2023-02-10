# The Box Model
![Gallery Analogy for the Terminology](images/css-box-model.avif)

- The content may overflow out of the box.
- Usually the size refers to the content box by default (`box-sizing: content-box;`), unless it's set to `box-sizing: border-box;`
- Browsers have their own user agent stylesheets, but they can be reset with reset and normalizer css files with (arguably) more sensible defaults.
E.g. to use the `border-box` sizing model,
```css
html {
  box-sizing: border-box;
}
*,
*::before,
*::after {
  box-sizing: inherit;
}

```
- Debugging tip: borders may affect the size of the box, so you could use `outline` instead -- takes the space of the margin, and does not affect the box size.
- The `display` type affects how box properties are respected:
  - `block` boxe uses all of the behaviors defined;
  - `inline` boxes use _some_ behaviors -- `width` and `height` are ignored; vertical padding/border/margin don't push away other contents; horizontal ones are respected;
  - `inline-block` works like `block`, but does not break into a new line.
- `Margin` values can be positive or negative; they may collapse by taking the max (if both positive), min (if both negative), or subtract (one of them negative), given additional margin collapsing rules.

# Selectors
- Simple

| Name                        | Looks Like                                                                   | Explanation                                                                                                   |
|-----------------------------|------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|
| universal/wildcard selector | `*`                                                                          | matches every element                                                                                         |
| type selector               | `section`                                                                    | matches the html tag                                                                                          |
| class selector              | `.my-class`                                                                  | also matches an element with multiple classes like `<div class="my-class another-class a-third-class"></div>` |
| id selector                 | `#my-id`                                                                     | id is unique to each element so it's rarely used                                                              |
| attribute selector          | `[data-type='primary']` (matches value), or `[data-type]` (matches presence) | can also specify the attribute value's case sensitivity, contains/starts with/ends with                       |
| group selector              | `.my-class, [lang]`                                                          | can group multiple selectors with comma in between them                                                       |
| pseudo-classes              | `a:hover`                                                                    | pseudo-classes are element in particular states; e.g. are interacted with                                     |
| pseudo-elements             | `.my-element::before`                                                        | pseudo-elements act as if they are inserting a new element with CSS                                           |
