# 🕛 Twelvety

[See the demo site](https://twelvety.netlify.app)

Twelvety is a pre-configured Eleventy starter project built to be fast. It includes:

- Component architecture
- CSS pipeline using Sass, PostCSS and CleanCSS
- JS pipeline using Browserify, Babel and Uglify
- Page-specific CSS and JS
- Inline critical CSS and defer non-critical CSS
- Minified HTML, CSS and JS
- Responsive picture shortcode with WebP support
- Content hash of assets

Write components like this:

```html
<main class="home">
  <h1 class="home__title">Twelvety</h1>
</main>

{% stylesheet 'scss' %}
  @import 'mixins';

  .home {
    @include container;

    &__title {
      color: red;
    }
  }
{% endstylesheet %}

{% javascript %}
  console.log('Super fast 💨')
{% endjavascript %}
```

## Deploy to netlify

To quickly deploy your own instance of Twelvety to Netlify, just click the button below and follow the instructions.

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/gregives/twelvety)

**What will happen when I click this button?** Netlify will clone the Twelvety git repository to your GitHub account (it will ask your permission to do this), add the new repository to your netlify account and deploy it!

## Run Locally

You'll need [Node.js](https://nodejs.org) and npm (included with Node.js). To install the required packages, run

```sh
npm install
```

### Commands

- Run `npm run serve` to run a development server and live reload
- Run `npm run build` to build for production
- Run `npm run clean` to clean the output folder and Twelvety cache

The brains of Twelvety live in the `utils` folder: if you just want to make a website, then you don't need to touch anything inside `utils`. However, if you want to change any of the shortcodes, have a look around!

## Features

Twelvety sets up transforms, shortcodes and some sensible Eleventy options. Click the features below to learn how they work.

<details>
<summary><code>stylesheet</code> paired shortcode</summary>
<br>

Use the `stylesheet` paired shortcode to include your Sass. You can import Sass files from your `styles` directory (defined in `.twelvety.js`) and from `node_modules`. The Sass will be rendered using [node-sass](https://github.com/sass/node-sass), passed into [PostCSS](https://github.com/postcss/postcss) (with [PostCSS Preset Env](https://github.com/csstools/postcss-preset-env) and [Autoprefixer](https://github.com/postcss/autoprefixer) for compatability) and either minified using [clean-css](https://github.com/jakubpawlowicz/clean-css) or beautified by [JS Beautifier](https://github.com/beautify-web/js-beautify) (in production and development respectively).

```html
{% stylesheet 'scss' %}
  @import 'normalize.css/normalize';
  @import 'mixins';

  .home {
    @include container;

    color: $color--red;
  }
{% endstylesheet %}
```

By default, Twelvety does **not** use indented Sass so you need to use semicolons. If you want to change this, have a look in the `renderStyles` function near the top of `utils/shortcodes/stylesheet.js`. In the future, Twelvety should probably use the `language` parameter of the paired shortcode (which currently does nothing) to infer this.

The `stylesheet` paired shortcode also has a third parameter, which by default is set to `page.url`, the URL of the current page being rendered. This means that only the required CSS is included in each page. You can make your own 'chunk' of CSS using this parameter, for example, a CSS file common to all pages of your website.

</details>

<details>
<summary><code>styles</code> shortcode</summary>
<br>

The `styles` shortcode collects together all Sass written in `stylesheet` paired shortcodes for the given chunk and outputs the rendered CSS. The 'chunk' defaults to `page.url`, the URL of the current page being rendered.

```html
<!-- Inline all styles on current page -->
<style>
  {% styles page.url %}
</style>

<!-- Capture styles on current page -->
{% capture css %}
  {% styles page.url %}
{% endcapture %}
<!-- And output asset using `asset` shortcode -->
<link rel="stylesheet" href="{% asset css, 'css' %}">
```

Note that the `styles` shortcode must be placed below any `stylesheet` paired shortcodes in the template; see the `append` paired shortcode and transform for more information.

</details>

<details>
<summary><code>javascript</code> paired shortcode</summary>
<br>

Include your JavaScript using the `javascript` paired shortcode. Twelvety uses [Browserify](http://browserify.org) so that you can `require('modules')` and [Babel](https://babeljs.io) so you can use the latest JavaScript. Your JavaScript will then be minified using [Uglify](https://github.com/mishoo/UglifyJS) in production or beautified by [JS Beautifier](https://github.com/beautify-web/js-beautify) in development.

```html
{% javascript %}
  const axios = require('axios')

  axios.get('/api/endpoint')
    .then((response) => {
      console.log('Yay, it worked!')
    })
    .catch((error) => {
      console.log('Uh oh, something went wrong')
    })
{% endjavascript %}
```

The `javascript` paired shortcode has a second parameter, which by default is set to `page.url`, the URL of the current page being rendered. This means that only the required JavaScript is included in each page. You can make your own 'chunk' of JavaScript using this parameter, for example, a JavaScript file for all vendor code.

The output of each `javascript` paired shortcode will be wrapped in an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) so that your variables do not pollute global scope. If you want to define something on `window`, use `window.something =`.

</details>

<details>
<summary><code>script</code> shortcode</summary>
<br>

The `script` shortcode collects together all the JavaScript for the given chunk and outputs the JavaScript (after transpilation and minification). The 'chunk' defaults to `page.url`, the URL of the current page being rendered.

```html
<!-- Inline all JavaScript on current page -->
<script>
  {% script page.url %}
</script>

<!-- Capture JavaScript on current page -->
{% capture js -%}
  {% script page.url %}
{%- endcapture -%}
<!-- And output asset using `asset` shortcode -->
<script src="{% asset js, 'js' %}" defer></script>
```

Note that the `script` shortcode must be placed below any `javascript` paired shortcodes in the template; usually this is not a problem as JavaScript is often included immediately preceding `</body>`. If you want the JavaScript somewhere else, see the `append` paired shortcode and transform.

</details>

<details>
<summary><code>asset</code> shortcode</summary>
<br>

The `asset` shortcode outputs a content-hashed asset with the given content and extension. The content may be either a `String` or `Buffer`. Assets will be saved to the `assets` directory inside the `output` directory (both defined within `.twelvety.js`).

```html
<!-- Capture some content -->
{% capture css %}
h1 {
  color: red;
}
{% endcapture %}

<!-- Save content to content-hashed file with .css extension -->
<link rel="stylesheet" href="{% asset css, 'css' %}">

<!-- Output of shortcode -->
<link rel="stylesheet" href="/_assets/58f4b924.css">
```

You can import the `asset` shortcode function in JavaScript: this is how the `picture` shortcode saves your responsive images into the `assets` directory.

</details>

## Visual Studio Code

If you're using Visual Studio Code I recommend this [Liquid extension](https://github.com/panoply/vscode-liquid) so that your Sass and JavaScript will be highlighted correctly.
