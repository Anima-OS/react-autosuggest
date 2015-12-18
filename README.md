<a href="https://codeship.com/projects/67868" target="_blank">
  <img src="https://img.shields.io/codeship/41810250-aa07-0132-fbf4-4e62e8945e03/master.svg"
       alt="Build Status" />
</a>
<a href="https://npmjs.org/package/react-autosuggest" target="_blank">
  <img src="https://img.shields.io/npm/v/react-autosuggest.svg"
       alt="NPM Version" />
</a>
<a href="https://npmjs.org/package/react-autosuggest" target="_blank">
  <img src="https://img.shields.io/npm/dm/react-autosuggest.svg"
       alt="NPM Downloads" />
</a>

# React Autosuggest

[WAI-ARIA compliant][wai-aria] React autosuggest component.


## Demo

* <a href="https://moroshko.github.io/react-autosuggest#basic-usage" target="_blank">Basic usage</a>
* <a href="https://moroshko.github.io/react-autosuggest#highlight-matches" target="_blank">Highlight matches</a>
* <a href="https://moroshko.github.io/react-autosuggest#async-example" target="_blank">Async example</a>
* <a href="https://moroshko.github.io/react-autosuggest#multiple-sections" target="_blank">Multiple sections</a>
* <a href="https://moroshko.github.io/react-autosuggest#debounced-example" target="_blank">Debounced example</a>
* <a href="https://moroshko.github.io/react-autosuggest#caching-example" target="_blank">Caching example</a>

## Playground

[Codepen][playground]

## Features

* [WAI-ARIA accessible][wai-aria] (including ARIA attributes and keyboard interactions)
* Supports flux architecture (see [redux examples][examples])
* Supports [react-themeable](https://github.com/markdalgleish/react-themeable) for flexible styling
* Supports [multiple sections][multiple-sections] as well as [plain list][basic-usage] of suggestions
* Full control over [suggestion rendering](#renderSuggestionProp) (you can display extra data, images, whatever you want)
* Full control over [when to show the suggestions](#shouldRenderSuggestionsProp) (e.g. when user types 2 or more characters)
* [Pass through props to the input field](#inputPropsProp) (e.g. placeholder, type, onChange, onBlur)
* [onSuggestionSelected](#onSuggestionSelectedProp) hook
* Thoroughly tested

The following are not part of react-autosuggest, but can be easily implemented:

* Delayed requests management (if request comes back after user types another letter, it will be ignored). [Example][async-example]
* In-memory caching (suggestions for a given input are retrieved only once). [Example][caching-example]

## Installation

```shell
npm install react-autosuggest --save
```

## Basic Usage

This example doesn't use flux architecture. If you would like to connect Autosuggest to your redux app, check out the [redux examples][examples].

```js
import Autosuggest from 'react-autosuggest';

const languages = [{
  name: 'C',
  year: 1972
}, {
  name: 'Elm',
  year: 2012
}, {
  name: 'Javascript',
  year: 1995
}, {
  name: 'Python',
  year: 1991
}];

function getMatchingLanguages(value) {
  const escapedValue = escapeRegexCharacters(value.trim()); // See: https://github.com/moroshko/react-autosuggest/blob/master/demo/src/components/utils/utils.js#L2-L4
  const regex = new RegExp('^' + escapedValue, 'i');

  return languages.filter(language => regex.test(language.name));
}

function getSuggestionValue(suggestion) { // when suggestion selected, this function tells
  return suggestion.name;                 // what should be the value of the input
}

function renderSuggestion(suggestion) {
  return (
    <span>{suggestion.name}</span>
  );
}

class Example extends React.Component {
  constructor() {
    super();

    this.state = {
      value: '',
      suggestions: getMatchingLanguages('')
    };

    this.onChange = this.onChange.bind(this);
    this.onSuggestionSelected = this.onSuggestionSelected.bind(this);
  }

  onChange(event, { newValue, method }) {
    if (method === 'type') {
      this.setState({
        value: newValue,
        suggestions: getMatchingLanguages(newValue)
      });
    } else {
      this.setState({
        value: newValue
      });
    }
  }

  // When suggestion is selected, we need to update `suggestions` so that,
  // if user presses Up or Down to reveal suggestions,
  // they would see the updated list.
  onSuggestionSelected(event, { suggestionValue }) {
    this.setState({
      suggestions: getMatchingLanguages(suggestionValue)
    });
  }

  render() {
    const { value, suggestions } = this.state;
    const inputProps = {
      placeholder: 'Type a programming language',
      value,
      onChange: this.onChange
    };

    return (
      <Autosuggest suggestions={suggestions}
                   getSuggestionValue={getSuggestionValue}
                   renderSuggestion={renderSuggestion}
                   inputProps={inputProps}
                   onSuggestionSelected={this.onSuggestionSelected} />
    );
  }
}
```

## Props

* [`suggestions`](#suggestionsProp)
* [`getSuggestionValue`](#getSuggestionValueProp)
* [`renderSuggestion`](#renderSuggestionProp)
* [`inputProps`](#inputPropsProp)
* [`shouldRenderSuggestions`](#shouldRenderSuggestionsProp)
* [`multiSection`](#multiSectionProp)
* [`renderSectionTitle`](#renderSectionTitleProp)
* [`getSectionSuggestions`](#getSectionSuggestionsProp)
* [`onSuggestionSelected`](#onSuggestionSelectedProp)
* [`theme`](#themeProp)

<a name="suggestionsProp"></a>
#### suggestions (required)

Arbitrary array of suggestions to display.

For plain list of suggestions, you could have:

```js
const suggestions = [{
  text: 'Apple'
}, {
  text: 'Banana'
}, {
  text: 'Cherry'
}, {
  text: 'Grapefruit'
}, {
  text: 'Lemon'
}];
```

You could also have [multiple sections](#multiSectionProp), in which case `suggestions` would be an array of sections. For example:

```js
const suggestions = [{
  title: 'A',
  suggestions: [{
    id: '100',
    text: 'Apple'
  }, {
    id: '101',
    text: 'Apricot'
  }]
}, {
  title: 'B',
  suggestions: [{
    id: '102',
    text: 'Banana'
  }]
}, {
  title: 'C',
  suggestions: [{
    id: '103',
    text: 'Cherry'
  }]
}];
```

**Note:**

* It's totally up to you what shape suggestions take!
* The initial value of `suggestions` should match the initial value of `inputProps.value`. This is to make sure that, if input has a non-empty initial value, and it's focused, the right suggestions are displayed.

<a name="getSuggestionValueProp"></a>
#### getSuggestionValue (required)

When user navigates the suggestions using the Up and Down keys, [the input should display the highlighted suggestion][wai-aria]. You design how suggestion is modelled. Therefore, it's your responsibility to tell Autosuggest how to map suggestions to input values.

This function gets:

* `suggestion` - The suggestion in question

It should return a string. For example:

```js
function getSuggestionValue(suggestion) {
  return suggestion.text;
}
```

<a name="renderSuggestionProp"></a>
#### renderSuggestion (required)

Use your imagination to define how suggestions are rendered.

This function gets:

* `suggestion` - The suggestion to render
* `value` - The current value of the input
* `valueBeforeUpDown` - The value of the input prior to Up/Down interactions. If user didn't interact with Up/Down yet, it will be `null`. It is useful if you want to highlight input's value in the suggestion (a.k.a the match), for example.

It should return a `ReactElement`. For example:

```js
function renderSuggestion(suggestion, value, valueBeforeUpDown) {
  return (
    <span>{suggestion.text}</span>
  );
}
```

<a name="inputPropsProp"></a>
#### inputProps (required)

Autosuggest is a [controlled component][controlled-component]. Therefore, you should pass at least a `value` and an `onChange` callback to the input field. You can pass additional props as well. For example:

```js
const inputProps = {
  value: inputValue,   // `inputValue` usually comes from application state
  onChange: onChange   // `onChange` will be called when input value changes
  type: 'search',
  placeholder: 'Enter city or postcode'
};
```

`onChange` has the following signature:

```js
function onChange(event, { newValue, method })
```

where:

* `newValue` - the new value of the input field
* `method` - string describing how the change occurred. The possible values are:
  * `'down'` - user pressed Down
  * `'up'` - user pressed Up
  * `'escape'` - user pressed Esc
  * `'click'` - user clicked on suggestion
  * `'type'` - none of the methods above (usually means that user typed something, but can also be that they pressed Backspace, pasted something into the field, etc.)

<a name="shouldRenderSuggestionsProp"></a>
#### shouldRenderSuggestions (optional)

By default, suggestions are rendered when input field isn't blank. Feel free to override this behaviour.

This function gets:

* `value` - The current value of the input

It should return a boolean.

For example, to display suggestions only when input is at least 3 characters long, do:

```js
function shouldRenderSuggestions(value) {
  return value.trim().length > 2;
}
```

<a name="multiSectionProp"></a>
#### multiSection (optional)

By default, Autosuggest renders a plain list of suggestions.

If you'd like to have multiple sections (with optional titles), set `multiSection={true}`.

<a name="renderSectionTitleProp"></a>
#### renderSectionTitle (optional)

When rendering [multiple sections](#multiSectionProp), you need to tell Autosuggest how to render a section title.

This function gets:

* `section` - The section to render (this would be an item in the [suggestions](#suggestionsProp) array


It should return a `ReactElement`. For example:

```js
function renderSectionTitle(section) {
  return (
    <strong>{section.title}</strong>
  );
}
```

<a name="getSectionSuggestionsProp"></a>
#### getSectionSuggestions (optional)

When rendering [multiple sections](#multiSectionProp), you need to tell Autosuggest where to find the suggestions for a given section.

This function gets:

* `section` - The section to render (this would be an item in the [suggestions](#suggestionsProp) array


It should return an array of suggestions to render in the given section. For example:

```js
function getSectionSuggestions(section) {
  return section.suggestions;
}
```

<a name="onSuggestionSelectedProp"></a>
#### onSuggestionSelected (optional)

This function is called when suggestion is selected. It has the following signature:

```js
function onSuggestionSelected(event, { suggestion, suggestionValue, method })
```

where:

* `suggestion` - the selected suggestion
* `suggestionValue` - the value of the selected suggestion (equivalent to `getSuggestionValue(suggestion)`)
* `method` - string describing how user selected the suggestion. The possible values are:
  * `'click'` - user clicked on the suggestion
  * `'enter'` - user selected the suggestion using Enter

<a name="themeProp"></a>
#### theme (optional)

Autosuggest comes with no styles, and it supports [react-themeable](https://github.com/markdalgleish/react-themeable).

This means you can use [CSS Modules](https://github.com/css-modules/css-modules), [Radium](http://projects.formidablelabs.com/radium), [React Style](https://github.com/js-next/react-style), [JSS](https://github.com/jsstyles/jss), inline styles, or even global CSS to style your Autosugest component.

For example, to style the Autosuggest using CSS Modules, do:

```css
/* theme.css */

.container { ... }
.input { ... }
.items-container { ... }
.item { ... }
.item--focused { ... }
...
```

```js
import theme from 'theme.css';
```
```xml
<Autosuggest theme={theme} ... />
```

When not specified, `theme` defaults to:

```js
{
  'container':               'react-autosuggest__container',
  'container--open':         'react-autosuggest__container--open',
  'input':                   'react-autosuggest__input',
  'items-container':         'react-autosuggest__suggestions-container',
  'item':                    'react-autosuggest__item',
  'item--focused':           'react-autosuggest__item--focused',
  'section-container':       'react-autosuggest__section-container',
  'section-title':           'react-autosuggest__section-title',
  'section-items-container': 'react-autosuggest__section-suggestions-container'
}
```

An example of styling an Autosuggest using CSS Modules can be found [here](https://github.com/moroshko/react-autosuggest/blob/master/demo/src/components/theme.less).

The following diagrams illustrate how `theme` is structured.

##### No sections

    +--| container |-----------------+
    |                                |
    |  +--| input |---------------+  |
    |  |                          |  |
    |  +--------------------------+  |
    |                                |
    |  +--| items-container |-----+  |
    |  |                          |  |
    |  |  +--| item |----------+  |  |
    |  |  |                    |  |  |
    |  |  +--------------------+  |  |
    |  |                          |  |
    |  +--------------------------+  |
    |                                |
    +--------------------------------+


##### Multiple sections


    +--| container |----------------------------------+
    |                                                 |
    |  +--| input |--------------------------------+  |
    |  |                                           |  |
    |  +-------------------------------------------+  |
    |                                                 |
    |  +--| items-container |----------------------+  |
    |  |                                           |  |
    |  |  +--| section-container |--------------+  |  |
    |  |  |                                     |  |  |
    |  |  |  +--| section-title |------------+  |  |  |
    |  |  |  |                               |  |  |  |
    |  |  |  +-------------------------------+  |  |  |
    |  |  |                                     |  |  |
    |  |  |  +--| section-items-container |--+  |  |  |
    |  |  |  |                               |  |  |  |
    |  |  |  |  +--| item |---------------+  |  |  |  |
    |  |  |  |  |                         |  |  |  |  |
    |  |  |  |  +-------------------------+  |  |  |  |
    |  |  |  |                               |  |  |  |
    |  |  |  +-------------------------------+  |  |  |
    |  |  |                                     |  |  |
    |  |  +-------------------------------------+  |  |
    |  |                                           |  |
    |  +-------------------------------------------+  |
    |                                                 |
    +-------------------------------------------------+

## Development

```shell
npm install
npm start
```

Now, open `http://localhost:3000/demo/dist/index.html` and start hacking!

## Running Tests

```shell
npm test
```

## License

[MIT](http://moroshko.mit-license.org)

[wai-aria]: https://www.w3.org/TR/wai-aria-practices/#autocomplete
[controlled-component]: https://facebook.github.io/react/docs/forms.html#controlled-components
[examples]: https://github.com/moroshko/react-autosuggest/tree/master/demo/src/components/App/components
[basic-usage]: https://moroshko.github.io/react-autosuggest#basic-usage
[multiple-sections]: https://moroshko.github.io/react-autosuggest#multiple-sections
[async-example]: https://moroshko.github.io/react-autosuggest#async-example
[caching-example]: https://moroshko.github.io/react-autosuggest#caching-example
[playground]: http://codepen.io/moroshko/pen/BjKmQL

---

**TODO:**

* Examples:
  * No results
  * Navigate on selection
* Write upgrade guide
* Release 3.0
  * Update deps
  * Publish to npm
  * Publish to gh-pages
* Add support for scrollbar

