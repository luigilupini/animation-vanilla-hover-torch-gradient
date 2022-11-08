## Hovering torch gradient

> Here applied is a very cool torch like gradient effect when hovering a card.

![alt text](./capture.png)

Featuring:

- In these cards a pseudo circle gradient background lights on mouse position.

```html
<div id="cards">
  <div class="card"></div>
  <div class="card"></div>
  <div class="card"></div>
  <div class="card"></div>
  <div class="card"></div>
  <div class="card"></div>
</div>
```

- Dynamically produce a `""` pseudo gradient `::before/::after` each card.

This pseudo gradient is absolutely positioned "relative" to the parent card. The
element is relative to the card, instead of the default surrounding page.

```css
.card::before,
.card::after {
  content: "";
  position: absolute;
  border-radius: inherit;
  height: 100%;
  width: 100%;
  top: 0px;
  left: 0px;
  /* Remove these two to observe a bug when the gradient remains on/in card */
  opacity: 0;
  transition: opacity 500ms;
}

.card::before {
  /* JS provides variable for gradient origin: */
  background: radial-gradient(
    800px circle at var(--coords-x) var(--coords-y),
    rgba(255, 255, 255, 0.06),
    transparent 40%
  );
  z-index: 3;
}
.card::after {
  /* JS provides variable for gradient origin: */
  background: radial-gradient(
    600px circle at var(--coords-x) var(--coords-y),
    rgba(255, 255, 255, 0.4),
    transparent 40%
  );
  z-index: 1;
}

/* Apply hover effect here 🧙‍♂️ */
.card:hover::before {
  opacity: 1;
}
```

- Within our script we iterate a listener to each card of a HTML collection.

The star of the show is making use of [getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect).

![alt text](./element-box-diagram.png)

This helpful method is used to determine the pointer position in each card. Each
card is going to produce a coordinate value that we `setProperty`. This produces
a CSS variable used to update our `before/after` gradient, origin position.

```js
const cards = document.querySelectorAll(".card");

const handleMouseMove = (event) => {
  const { currentTarget: card, clientX, clientY } = event;
  // The `getBoundingClientRect()` method returns a DOMRect object provide
  // info about the element size & its position, relative to the viewport.
  const { left: cardLeft, top: cardTop } = card.getBoundingClientRect(),
    // But we use it to calc our mouse position relative to each card.
    coordX = clientX - cardLeft,
    coordY = clientY - cardTop;
  // ...
  console.table({
    card: { clientX, clientY },
    boundary: { cardLeft, cardTop },
    coordinates: { coordX, coordY }, // coords in each
  });
  //...
  // We use the above to setup custom css properties for each card :)
  card.style.setProperty("--coords-x", `${coordX}px`);
  card.style.setProperty("--coords-y", `${coordY}px`);
  // Back in our radial-gradient we can use this variables!
};

for (let card of cards) {
  card.addEventListener("mousemove", handleMouseMove);
}
```

- Extending the hovering effect on the card and sibling card borders.

Because you can't provide coordinate color to different section of a border. Add
an additional inner child to each card `after` giving (cloudy/opaque) background
that appears as a border space, due to the inner child with a 100% - 4px (h/w).

```css
.card-inner {
  position: absolute;
  background: var(--card-inner);
  border-radius: inherit;
  margin: 2px;
  width: calc(100% - 4px);
  height: calc(100% - 4px);
  z-index: 2;
}
```
