# Reproduction of error _JSXStyle is not defined

This repo represents a minimal reproduction of one scenario in which I have encountered the error `_JSXStyle is not defined`.

The conditions that seem to be necessary to reproduce are:
- using babel to transpile a react component that includes a `<style jsx>` element
- `@babel/preset-env` is included in the babel presets
- the component uses object destructuring in the props argument.

## Steps to reproduce

```
npm install
npx babel Component.js
```

### Input (Component.js)
```javascript
export const Component = ({ className }) => (
  <div className={className}>
    <style jsx>{`
      div {
        background: red
      }
    `}
    </style>
  </div>
);
```

### Output

```javascript
"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.Component = void 0;

var _style = _interopRequireDefault(require("styled-jsx/style"));

function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }

var Component = function Component(_ref) {
  var className = _ref.className;
  return /*#__PURE__*/React.createElement("div", {
    className: "jsx-2776096334" + " " + (className || "")
  }, /*#__PURE__*/React.createElement(_JSXStyle, {
    id: "2776096334"
  }, "div.jsx-2776096334{background:red;}"));
};

exports.Component = Component;
```

Note that `styled-jsx/style` is imported as:

```javascript
var _style = _interopRequireDefault(require("styled-jsx/style"));
```

but then it is referred to later in the file as `_JSXStyle` which causes the error:

```javascript
React.createElement(_JSXStyle, {
    id: "2776096334"
})
```

## Changes which fix the issue

Any one of the following change on its own resolves the issue, resulting in the `styled-jsx/style` import being referenced correctly. ie.

```javascript
React.createElement(_style, {
    id: "2776096334"
})
```

1. Don't use object destructuring in the props argument
2. Don't include the `@babel/preset-env` preset in the babel config
3. Downgrading to `styled-jsx@3.3.2`. The error occurrs in `3.3.3` and `3.4.4`. I have not tested other versions.
4. Exclude the `transform-parameters` module from `@babel/preset-env` eg.
```json
{
  "presets": [
    [
      "@babel/preset-env", {
        "exclude": [
          "transform-parameters"
        ]
      }
    ],
    "@babel/preset-react"
  ],
  "plugins": [
    "styled-jsx/babel"
  ]
}
```
