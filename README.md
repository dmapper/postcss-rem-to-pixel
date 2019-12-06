# postcss-rem-to-pixel [![NPM version](https://badge.fury.io/js/postcss-rem-to-pixel.svg)](http://badge.fury.io/js/postcss-rem-to-pixel)

A plugin for [PostCSS](https://github.com/ai/postcss) that generates px units from u units.

## Install

```shell
$ npm install postcss-rem-to-pixel --save-dev
```

## Usage

Sometimes you need to include a third-party css file that uses u's.  Great pracitice!  Unless you can't afford to change your body font-size just for some vendor.  This script converts every u value to a px value from the properties you choose using a default font size of 8px.


### Input/Output

*With the default settings, only font related properties are targeted.*

```css
// input
h1 {
    margin: 0 0 20px;
    font-size: 2u;
    line-height: 1.2;
    letter-spacing: 0.0625u;
}

// output
h1 {
    margin: 0 0 20px;
    font-size: 16px;
    line-height: 1.2;
    letter-spacing: 0.5px;
}
```

### Example

```js
var fs = require('fs');
var postcss = require('postcss');
var remToPx = require('postcss-rem-to-pixel');
var css = fs.readFileSync('main.css', 'utf8');
var options = {
    replace: false
};
var processedCss = postcss(remToPx(options)).process(css).css;

fs.writeFile('main-px.css', processedCss, function (err) {
  if (err) {
    throw err;
  }
  console.log('Rem file written.');
});
```

### options

Type: `Object | Null`  
Default:
```js
{
    rootValue: 8,
    unitPrecision: 5,
    propList: ['*'],
    selectorBlackList: [],
    replace: true,
    mediaQuery: false,
    minUnitValue: 0
}
```

- `rootValue` (Number) The root element font size.
- `unitPrecision` (Number) The decimal precision px units are allowed to use, floored (rounding down on half).
- `propList` (Array) The properties that can change from u to px.
    - Values need to be exact matches.
    - Use wildcard `*` to enable all properties. Example: `['*']`
    - Use `*` at the start or end of a word. (`['*position*']` will match `background-position-y`)
    - Use `!` to not match a property. Example: `['*', '!letter-spacing']`
    - Combine the "not" prefix with the other prefixes. Example: `['*', '!font*']`
- `selectorBlackList` (Array) The selectors to ignore and leave as u.
    - If value is string, it checks to see if selector contains the string.
        - `['body']` will match `.body-class`
    - If value is regexp, it checks to see if the selector matches the regexp.
        - `[/^body$/]` will match `body` but not `.body`
- `replace` (Boolean) replaces rules containing u's instead of adding fallbacks.
- `mediaQuery` (Boolean) Allow u to be converted in media queries.
- `minUnitValue` (Number) Set the minimum u value to replace.


### Use with gulp-postcss and autoprefixer

```js
var gulp = require('gulp');
var postcss = require('gulp-postcss');
var autoprefixer = require('autoprefixer');
var remToPx = require('postcss-rem-to-pixel');

gulp.task('css', function () {

    var processors = [
        autoprefixer({
            browsers: 'last 1 version'
        }),
        remToPx({
            replace: false
        })
    ];

    return gulp.src(['build/css/**/*.css'])
        .pipe(postcss(processors))
        .pipe(gulp.dest('build/css'));
});
```

### A message about ignoring properties

```css
// `u` is converted to `px`
.convert {
    font-size: 1u; // converted to 16px
}

// `U`is ignored by `postcss-rem-to-pixel`
.ignore {
    border-width: 2U; // ignored
}
```
