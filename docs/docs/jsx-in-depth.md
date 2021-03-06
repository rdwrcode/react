---
id: jsx-in-depth
title: JSX In Depth
permalink: docs/jsx-in-depth.html
---

Fundamentally, JSX just provides syntactic sugar for the `React.createElement(component, props, ...children)` function. The JSX code:

```js
<MyButton color="blue" shadowSize={2}>Click Me</MyButton>
```

compiles into:

```js
React.createElement(MyButton, {color: "blue", shadowSize: 2}, 'Click Me')
```

You can also use the self-closing form of the tag if there are no children. So:

```js
<div className="sidebar" />
```

compiles into:

```js
React.createElement('div', {className: 'sidebar'}, null)
```

If you want to test out how some specific JSX is converted into JavaScript, you can try out [the online Babel compiler](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%2Creact%2Cstage-0&code=function%20hello()%20%7B%0A%20%20return%20%3Cdiv%3EHello%20world!%3C%2Fdiv%3E%3B%0A%7D).

## Specifying The React Element Type

The first part of a JSX tag determines the type of the React element.

Capitalized types indicate that the JSX tag is referring to a React component. These tags get compiled into a direct reference to the named variable, so if you use the JSX `<Foo />` expression, `Foo` must be in scope.

Since JSX compiles into calls to `React.createElement`, the `React` library must also always be in scope from your JSX code.

For example, both of the imports are necessary in this code, even though 'React' and 'CustomButton' are not directly referenced from JavaScript:

```js
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  return <CustomButton color="red" />;
}
```

If you don't use a JavaScript bundler and added React as a script tag, it is already in scope as a React global.

You can also refer to a React component using dot-notation from within JSX. This is convenient if you have a single module that exports many React components. For example, if `MyComponents.DatePicker` is a component, you can use it directly from JSX with:

```js
import React from 'react';

var MyComponents = {
  DatePicker: function(props) {
    return <div>imagine a {props.color} datepicker here</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue"} />;
}
```

When an element type starts with a lowercase letter, it refers to a built-in component like `<div>` or `<span>` and results in a string `'div'` or `'span'` passed to `React.createElement`. Types that start with a capital letter like `<Foo />` compile to `React.createElement(Foo)` and correspond to a component defined or imported in your JavaScript file.

We recommend naming components with a capital letter. If you do have a component that starts with a lowercase letter, assign it to a capitalized variable before using it in JSX.

For example, this code will not run as expected:

```js
import React from 'react';

function hello(props) {
  // This use of <div> is legitimate because div is a valid HTML tag
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // This code attempts to create an HTML <hello> tag and fails
  return <hello toWhat="World" />;
}
```

You cannot use a general expression as the React element type. If you do want to use a general expression to indicate the type of the element, just assign it to a capitalized variable first. This often comes up when you want to render a different component based on a prop:

```js
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: <PhotoStory />,
  video: <VideoStory />,
};

function Story1(props) {
  // Not valid JSX
  return <components[props.story] />;
}

function render2(props) {
  var MyComponent = components[props.story];

  // Valid JSX
  return <MyComponent />;
}
```

## Props in JSX

There are several different ways to specify props in JSX.

### JavaScript Expressions

You can pass any JavaScript expression as a prop, by surrounding it with `{}`. For example, in this JSX:

```js
<MyComponent foo={1 + 2 + 3 + 4} />
```

For `MyComponent`, The value of `props.foo` will be `10` because the expression `1 + 2 + 3 + 4` gets evaluated.

`if` statements and `for` loops are not expressions in JavaScript, so they can't be used in JSX directly. Instead, you can put these in the surrounding code. For example:

```js
function NumberDescriber(props) {
  var description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
```

### String Literals

You can pass a string literal as a prop. These two JSX expressions are equivalent:

```js
<MyComponent message="hello world" />

<MyComponent message={"hello world"} />
```

When you pass a string literal, its value is HTML-unescaped. So these two JSX expressions are equivalent:

```js
<MyComponent message="&lt;3" />

<MyComponent message={"<3"} />
```

This behavior is usually not relevant. It's only mentioned here for completeness.

### Props Default to "True"

If you pass no value for a prop, it defaults to `true`. These two JSX expressions are equivalent:

```js
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

In general, we don't recommend using this because it can be confused with the ES6 object shorthand `{foo}` which is short for {foo: foo} rather than `{foo: true}`. This behavior is just there so that it matches the behavior of HTML.

### Spread Attributes

If you already have `props` as an object, and you want to pass it in JSX, you can use `...` as a "spread" operator to pass the whole props object. These two render functions are equivalent:

```js
function render1() {
  var props = {left: 'ben', right: 'hector'};
  return <MyComponent {...props} />;
}

function render2() {
  return <MyComponent left="ben" right="hector" />;
}
```

Spread attributes can be useful when you are building generic containers. However, they can also make your code messy by making it easy to pass a lot of irrelevant props to components that don't care about them. We recommend that you use this syntax sparingly.

## Children in JSX

In JSX expressions that contain both an opening tag and a closing tag, the content between those tags is passed as a special prop: `props.children`. There are several different ways to pass children:

### String Literals

You can put a string between the opening and closing tags and `props.children` will just be that string. This is useful for many of the built-in HTML elements. For example:

```js
<MyComponent>Hello world!</MyComponent>
```

This is valid JSX, and `props.children` in `MyComponent` will simply be the string `"Hello world!"`. HTML is unescaped, so you can generally write JSX just like you would write HTML in this way:

```html
<div>This is valid HTML &amp; JSX at the same time.</div>
```

JSX removes whitespace at the beginning and ending of a line. It also removes blank lines. New lines adjacent to tags are removed; new lines that occur in the middle of string literals are condensed into a single space. So these all render to the same thing:

```js
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### JSX Children

You can provide more JSX elements as the children. This is useful for displaying nested components:

```js
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

You can mix together different types of children, so you can use string literals together with JSX children. This is another way in which JSX is like HTML, so that this is both valid JSX and valid HTML:

```html
<div>
  Here is a list:
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
```

A React component can't return multiple React elements, but a single JSX expression can have multiple children, so if you want a component to render multiple things you can wrap it in a `div` like this.

### JavaScript Expressions

You can pass any JavaScript expression as children, by enclosing it within `{}`. For example, these expressions are equivalent:

```js
<MyComponent>foo</MyComponent>

<MyComponent>{'foo'}</MyComponent>
```

This is often useful for rendering a list of JSX expressions of arbitrary length. For example, this renders an HTML list:

```js
function Item(props) {
  return <li>{props.message}</li>;
}

function renderTodoList() {
  var todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

JavaScript expressions can be mixed with other types of children. This is often useful in lieu of string templates:

```js
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

Normally, JavaScript expressions inserted in JSX will evaluate to a string, a React element, or a list of those things. However, `props.children` works just like any other prop in that it can pass any sort of data, not just the sorts that React knows how to render. For example, if you have a custom component, you could have it take a callback as `props.children`:

```js
function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}

// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  var items = [];
  for (var i = 0; i < numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>
}
```

Children passed to a custom component can be anything, as long as that component transforms them into something React can understand before rendering. This usage is not common, but it works if you want to stretch what JSX is capable of.

`false`, `null`, 'undefined', and 'true' are valid children. They simply don't render. These JSX expressions will all render to the same thing:

```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{true}</div>
```

This can be useful to conditionally render React elements. This JSX only renders a `<Header />` if `showHeader` is true:

```js
<div>{showHeader && <Header />}<Content /></div>
```
