# Aim

Aim is a UI library for ReasonML in the ** View-Model-Update ** pattern.

You can declare a View and update it whenever you want.

Aim does not use virtual DOM.

Instead, specify the dynamic part and reflect it to the real DOM at the same time as the state is updated.

# Counter app

Here is an example of a simple counter application.

`` `javascript
open Webapi.Dom;

module Counter = {
  let increment = count => count + 1;

  type actions =
    | Increment;

  let make = () => {
    Aim.
      // View
      component (
        [
          html (
            "div",
            [],
            [
              html ("span", [], [text_ (count => Js.Int.toString (count))]),
              html (
                "button",
                [event ("click", (e, dispatch) => dispatch (Increment))],
                [text ("+")],
              ),
            ],
          ),
        ],
        // The view is updated by passing the state to the update function argument.
        update => update (0),
        (action, update, count) => {
          // Called when the dispatch function is executed, such as in a click event
          switch (action) {
          | Increment => update (count + 1)
          }
        },
      )
    );
  };
};

let body = Document.querySelector ("body", document);

switch (body) {
| Some (body) =>
  let counter = Counter.make ();

  Aim.render (counter, body);
| None =>
  exception Body_NotFound;
  raise (Body_NotFound);
};
`` `

# How to use

## `render` and` component`

`render` draws a DOM on an HTMLElement using the result of the` component` function.

`` `javascript
render (
  component (
    [html ("div", [], [],
    _ => (),
    (_, _, _) => (),
  ),
  // Dom.element
  container
);
`` `

The first argument of the `component` function is an array of elements.

Describes how to define elements.

## Element definition

### `html (tagName, attributes, children)`

The `html` function is for defining HTMLElement.

### `text (content)`

The `text` function is for defining a TextNode.

### `attr (name, value)`

The `attr` function defines an attribute.

Here is an example using these functions.

`` `javascript
render (
  component (
    [html ("button", [attr ("type", "button")], [text ("button")])],
    _ => (),
    (_, _, _) => (),
  ),
  container
);
`` `

## logical attributes

### `boolAttr (name, trueOrFalse)`

Define logical attributes such as `checked` and` selected`.

`` `javascript
render (
  component (
    [html ("input", [attr ("type", "checkbox"), boolAttr ("checked", true)], [])],
    _ => (),
    (_, _, _) => (),
  ),
  container
);
`` `

## State dynamic elements and initialization

Here is an example using the `text_` function to define a dynamic` TextNode` for a state.

In this example, `text_` takes a function with the signature` int => string` by type inference.

`` `javascript
render (
  component (
    [
      html ("button", [], [text ("button")]),
      html ("span", [], [
        // function for state
        text_ (count => Js.Int.toString (count))
      ]),
    ],
    // Update view with initial value 0 using `update` function
    update => update (0),
    (_, _, _) => (),
  ),
  container
);
`` `

The function passed as the second argument of the `component` function is called only once at the beginning.

It takes a `update` function that performs a partial DOM update as an argument.

## Event

### `event (name, handler)`

The `event` function defines an event.

Aim's event handler can always receive the `dispatch` function and dispatch * action *.

* action * handling is done with the third argument of the `component` function.

`` `javascript
type actions =
  | Increment;

render (
  component (
    [
      html (
        "button",
        // receive the `dispatch` function as the second argument of the event handler
        [event ("click", (e, dispatch) => dispatch (Increment))],
        [text ("button")],
      ),
      html ("span", [], [text_ (count => Js.Int.toString (count))]),
    ],
    update => update (0),
    // The `action` that was dispatched is passed.
    // The `update` function updates the view,` current` is the current state value.
    (action, update, current) => {
      switch (action) {
      | Increment => update (current + 1)
      }
    },
  ),
  container
);
`` `

This section describes the definition of dynamic elements for states other than `text_`.

## State dynamic elements

### `attr_ (name, state => value)`

Make attributes dynamic.

In the following example, the display is switched between Boolean and hidden.

`` `javascript
type actions =
  | Switch;

render (
  component (
    [
      html (
        "button",
        [event ("click", (e, dispatch) => dispatch (Switch))],
        [text ("button")],
      ),
      html (
        "span",
        // the style attribute is dynamically switched
        [attr _ ("style", show => show? "": "display: none")],
        [text ("text")],
      ),
    ],
    update => update (true),
    (action, update, current) => {
      switch (action) {
      | Switch => update (! Current)
      }
    },
  ),
  container
);
`` `

### `boolAttr_ (name, state => trueOrFalse)`

Make logical attributes dynamic.

In the example below, pressing the button toggles the check box on and off.

`` `javascript
type actions =
  | Switch;

render (
  component (
    [
      html (
        "button",
        [event ("click", (e, dispatch) => dispatch (Switch))],
        [text ("button")],
      ),
      html (
        "input",
        [
          attr ("type", "checkbox"),
          // dynamic logical attributes
          boolAttr _ ("checked", checked => checked),
        ],
        [],
      ),
    ],
    update => update (true),
    (action, update, checked) => {
      switch (action) {
      | Switch => update (! Checked)
      }
    },
  ),
  container
);
`` `

### `node_ (iterator, keySelector, nodeSelector)`

Finally, let's talk about the `node_` function.

Use this function to render a dynamic number of elements.

`iterator` is a function that creates an array of states.

`keySelector` is a function that creates a unique key for each of the arrays created by` iterator`.

`nodeSelector` is a function that creates an element for each of the arrays created by` iterator`.

In this example, the number of elements decreases every second.

`` `javascript
render (
  component (
    <>
      {nodes_ (
         // increment `count` every second
         ction, update, current) => {
      switch (action) {
      | Switch => update (! Current)
      }
    },
  ),
  container
);
`` `

### `boolAttr_ (name, state => trueOrFalse)`

Make logical attributes dynamic.

In the example below, pressing the button toggles the check box on and off.

`` `javascript
type actions =
  | Switch;

render (
  component (
    [
      html (
        "button",
        [event ("click", (e, dispatch) => dispatch (Switch))],
        [text ("button")],
      ),
      html (
        "input",
        [
          attr ("type", "checkbox"),
          // dynamic logical attributes
          boolAttr _ ("checked", checked => checked),
        ],
        [],
      ),
    ],
    update => update (true),
    (action, update, checked) => {
      switch (action) {
      | Switch => update (! Checked)
      }
    },
  ),
  container
);
`` `

### `node_ (iterator, keySelector, nodeSelector)`

Finally, let's talk about the `node_` function.

Use this function to render a dynamic number of elements.

`iterator` is a function that creates an array of states.

`keySelector` is a function that creates a unique key for each of the arrays created by` iterator`.

`nodeSelector` is a function that creates an element for each of the arrays created by` iterator`.

In this example, the number of elements decreases every second.

`` `javascript
render (
  component (
    <>
      {nodes_ (
         // increment `count` every second
