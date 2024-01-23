# JavaScript level up with Stimulus.js
In our previous lessons, we've seen how [JavaScript](https://learn.firstdraft.com/lessons/203-minimal-js) can enhance the interactivity of our Rails applications using [Ajax with Rails Unobtrusive JavaScript (UJS)](https://learn.firstdraft.com/lessons/204-rails-unobtrusive-ajax). Building on this foundation, we're now set to delve deeper into the organization of JavaScript code and three distinct approaches to managing JavaScript in a Rails environment. To bring these concepts to life, we'll implement a practical example, implementing interactive features in a Ruby on Rails application using both Vanilla JavaScript and Stimulus.js.

## Evolution of JavaScript in Rails
JavaScript's integration in Rails has evolved significantly, adapting to the changing landscape of web development. Let's explore this evolution and its implications on how we manage JavaScript in a Rails application today.

### Early Days: Inline JavaScript
Initially, JavaScript in Rails was often embedded directly within HTML using `<script>` tags. This approach was straightforward but quickly became unmanageable as applications grew in complexity.

```html
<!DOCTYPE html>
<html>
  <body>
    <h2>JavaScript Alert</h2>
    <button onclick="myAlertFunction()">Click me</button>
    <script>
      function myAlertFunction() {
        alert("Hello, world!");
      }
    </script>
  </body>
</html>
```

### The Asset Pipeline Era
With Rails 3.1, the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) was introduced, addressing the challenges of managing JavaScript. It provides a structured approach to bundling, concatenating, and minifying JavaScript files.

#### Bundling and Concatenation
The Asset Pipeline combines multiple JavaScript files into a single file, reducing HTTP requests and improving load times.

#### Caching and Cache-Busting
The Asset Pipeline also implementes caching strategies. A unique fingerprint (hash) is added to file names (eg `/assets/application-abcdef1234567890.js`), ensuring users always receive the most updated version.

The rendered HTML of the deployed site translates to something like this:

```html
<script src="/assets/application-abcdef1234567890.js"></script>
```
This line in the deployed HTML is responsible for loading your application's JavaScript, ensuring users get the latest version of the script as per the current deployment. The rendered html version includes a fingerprint "abcdef1234567890". This fingerprint is a hash generated based on the file's content for cache-busting purposes.

#### Minification
Minification removes unnecessary characters from JavaScript files, reducing file size and speeding up page loads.

<aside>
  There is a convention to add a `min` suffix to minified files. (eg `filename.min.js`)
</aside>

#### Directory Structure and Sprockets
The Asset Pipeline standardizes the organization of these assets in `app/assets/`. 

```
app/
  assets/
    config/
      manifest.js
    images/
    stylesheets/
```

[Sprockets](https://github.com/rails/sprockets), a key component of the Pipeline, handled the concatenation and compression of assets. Projects using the Asset Pipeline will have the `sprockets-rails` gem in the `Gemfile`. There is a manifest file at `app/assets/config/manifest.js` that indexes all the assets you want to concatenate and minify. `//=` directives are used in `app/assets/config/manifest.js` to include images, stylesheets, and javascript.

```javascript
// app/assets/config/manifest.js

//= link_tree ../images
//= link_directory ../stylesheets .css
//= link_tree ../../javascript .js
```

Typically, JavaScript files in Rails are placed under `app/javascript`. As your application grows, keeping your javascript code in this directory with a clear structure keeps your codebase manageable. Clear organization also helps team members (and instructors 🥹) to understand and contribute to the codebase effectively. Here’s a typical structure:

```
app/
  javascript/
    channels/
    controllers/
    custom/
      your_custom_script.js
    application.js
```

- **Channels**: Used for [ActionCable](https://guides.rubyonrails.org/action_cable_overview.html) related files.
- **Controllers**: [Stimulus](https://github.com/hotwired/stimulus-rails) controllers are typically placed here. (more on this later)
- **Custom**: Create custom directories like `custom/` or `util/` for your specific scripts.
- **application.js**: The JavaScript manifest file. You can think of it as a central directory or index of your JavaScript files.

#### Limitations 
Despite its benefits, the Asset Pipeline has limitations, especially in managing JavaScript dependencies and modern JavaScript tooling.

<aside>
  Adding external JavaScript dependencies in a Rails application using the Asset Pipeline (before Rails 7) involved a few more manual steps. The process typically included:

  1. The `vendor/assets/javascripts` directory was used to store external JavaScript libraries or frameworks.

  2. Developers had to manually download the JavaScript file (e.g., a jQuery plugin) or copy it from a source and place it into the `vendor/assets/javascripts` directory. This process was not automated and required manual updating for each new version of the library.

  3. `app/assets/config/manifest.js` was used to include the external library using [Sprockets](https://github.com/rails/sprockets) directives.

  ```
    // Example of including an external library in app/assets/config/manifest.js
    //= require jquery
    //= require bootstrap
  ```
</aside>


#### Webpacker
This led to the introduction of [Webpacker](https://github.com/rails/webpacker) in Rails 5, allowing the integration of modern JavaScript frameworks (like [React](https://react.dev/) or [Vue](https://vuejs.org/)), tools like [Babel](https://babeljs.io/) for transpiling and [npm](https://www.npmjs.com/) for managing third party libraries.

<aside>
  Babel is a JavaScript transpiler that is used to convert ES6 code into backwards-compatible JavaScript code that can be run by older JavaScript engines. It allows web developers to take advantage of the newest features of the language while continuing to support older browsers like Internet Explorer.
</aside>

<aside>
  ES6 (also known as ECMAScript 6) is a recent update to the JavaScript programming language. ES6 modules allow you to break your JavaScript into smaller, reusable components. One approach to organize JavaScript in your project is to create separate modules in the `javascript/` directory. Then, export functions, objects, or classes from these modules and import them in `app/javascript/application.js` or any other module where you need them.

  Example:

  ```javascript
    // app/custom/hello.js
    export function sayHello(name) {
      return `Hello, ${name}!`;
    }

    // app/javascript/application.js
    import { sayHello } from './custom/hello';

    console.log(sayHello('Alice'));
  ```
</aside>

## Rails 7: Import Maps and Beyond

### The Role of Import Maps
[Import Maps](https://github.com/rails/importmap-rails) in Rails 7 addresses some of the Asset Pipeline's limitations by simplifying the inclusion of JavaScript dependencies. It leverages modern browser capabilities to load JavaScript modules directly from the browser at runtime (loading them from a CDN), without the need for compilation or bundling (like how we include [Bootstrap](https://getbootstrap.com/) or [Font Awesome](https://fontawesome.com/) in our `<head>`). 

<!-- TODO: stronger step by step example of how import maps works -->
```javascript
// Example of how Import Maps improved dependency management
import { myFunction } from 'my-dependency';
```

### Alternative Bundling Approaches
For more complex JavaScript requirements, Rails 7 offers alternative bundling options, catering to applications that use Single Page Application (SPA) frameworks (like [React](https://react.dev/) or [Vue](https://vuejs.org/)) or require extensive JavaScript tooling.

<!-- TODO: stronger example of how jsbundling works with React -->
```javascript
// Example of a complex JavaScript setup using alternative bundling
import React from 'react';
```

### API-only approach
<!-- TODO -->

### Choosing the Right Approach
- Choose the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) with [Import Maps](https://github.com/rails/importmap-rails) if your project aligns with the Rails asset management approach, particularly for less complex JavaScript integrations.
- Choose Alternative Bundling libraries like [jsbundling-rails](https://github.com/rails/jsbundling-rails) if your project requires integration with Single Page Application (SPA) frameworks like [React](https://react.dev/) or [Vue](https://vuejs.org/) or extensive JavaScript tooling like [NPM](https://www.npmjs.com/).
<!-- Choose API-only approach if... -->

<!-- TODO: more step by step setup for each practical example -->
## Practical Example: Toggling Text Visibility
We'll implement a feature where users can toggle the visibility of a paragraph of text on a webpage.

### Part 1: "Vanilla" JavaScript Approach (with Asset Pipeline)
Using the `onclick` attribute, we can directly attach a click event handler to the button in our HTML.

#### Step 1: Creating the View
Create a view `app/views/pages/home.html.erb` and include an `onclick` handler in the `<button>`:

```erb
<!-- app/views/pages/home.html.erb -->
<p id="my-hidden-text" class="hidden">
  This is hidden text.
</p>

<button onclick="toggleTextVisibility()">Toggle Visibility</button>
```
In this code, the `toggleTextVisibility` function will be called when the button is clicked.

#### Step 2: Adding JavaScript
Define the `toggleTextVisibility` function in a `<script>` tag within the HTML file or in a separate JavaScript file:

```html
<!-- Directly in the HTML file -->
<script>
  function toggleTextVisibility() {
    const text = document.getElementById('my-hidden-text');
    text.classList.toggle('hidden');
  }
</script>
```
Even better, keep the JavaScript separate:

```javascript
// app/javascript/custom/toggle_visibility.js
function toggleTextVisibility() {
  const text = document.getElementById('my-hidden-text');
  text.classList.toggle('hidden');
}
```
And then include it in your `application.js`:

```javascript
// app/javascript/application.js
import "../custom/toggle_visibility"
```

#### Step 3: CSS
Add the CSS for `.hidden`:

```css
/* app/assets/stylesheets/application.css */

.hidden {
  display: none;
}
```

<!-- TODO: add conclusion and a gif of the app -->

### Part 2: Stimulus.js Approach (with Import Maps)
Stimulus.js is a JavaScript framework designed for Rails applications. It enhances HTML by connecting elements to JavaScript objects via data attributes. JavaScript functionalities are encapsulated in controllers and targets, making the code more organized and maintainable.

#### Step 1: Setting Up Stimulus
Run the command to install Stimulus, if not already done:

1. Add the `stimulus-rails` gem to your `Gemfile`: `gem 'stimulus-rails'`
2. Run `./bin/bundle install`.
3. Run `./bin/rails stimulus:install`

#### Step 2: Creating a Stimulus Controller
Generate a Stimulus controller:

```bash
./bin/rails generate stimulus toggle
```

Edit the controller:

```javascript
// app/javascript/controllers/toggle_hidden_controller.js

import { Controller } from "stimulus";

export default class extends Controller {
  static targets = ["text"]

  toggle() {
    this.textTarget.classList.toggle('hidden');
  }
}
```

#### Step 3: Adjusting the View
Update the HTML to use Stimulus:

```erb
<!-- app/views/pages/home.html.erb -->

<div data-controller="toggle-hidden">
  <p data-toggle-hidden-target="text" class="hidden">
    This is hidden text.
  </p>

  <button data-action="click->toggle#toggle">Toggle Visibility</button>
</div>
```

<!-- 
TODO:
- add a React.js example?
- add a API-only example?
-->

## Resources

- [Working with JavaScript in Rails](https://guides.rubyonrails.org/working_with_javascript_in_rails.html)
- [Stimulus](https://stimulus.hotwired.dev/)

## Conclusion
In this lesson, you've learned two different ways to add interactive features to your Rails application. The vanilla JavaScript approach offers direct control over the DOM, while [Stimulus](https://stimulus.hotwired.dev/) provides a more structured and Rails-integrated method. Understanding both allows you to choose the right tool for your project's needs.
