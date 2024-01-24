# Organizing JavaScript Code
In our previous lessons, we've seen how [JavaScript](https://learn.firstdraft.com/lessons/203-minimal-js) can enhance the interactivity of our Rails applications using [Ajax with Rails Unobtrusive JavaScript (UJS)](https://learn.firstdraft.com/lessons/204-rails-unobtrusive-ajax). Building on this foundation, we're now set to delve deeper into the organization of JavaScript code and three distinct approaches to managing JavaScript in a Rails environment. To bring these concepts to life, we'll implement a practical example, implementing interactive features in a Ruby on Rails application using both Vanilla JavaScript, Stimulus.js, and React.

## Evolution of JavaScript in Rails
JavaScript's integration in Rails has evolved significantly, adapting to the changing landscape of web development. Let's explore this evolution and its implications on how we manage JavaScript in a Rails application today.

### Early Days: Inline JavaScript
Initially, JavaScript in Rails was often embedded directly within HTML using `<script>` tags. This approach is straightforward but quickly becomes unmanageable as applications grow in complexity.

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
The [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) combines multiple JavaScript files into a single file, reducing HTTP requests and improving load times.

#### Caching and Cache-Busting
The [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) also implementes caching strategies. A unique fingerprint is added to file names ensuring users always receive the most updated version. The rendered HTML of the deployed site translates to something like this:

```html
<script src="/assets/application-abcdef1234567890.js"></script>
```
The rendered html version includes a fingerprint "abcdef1234567890". This fingerprint is a hash generated based on the file's content for cache-busting purposes.

#### Minification
The [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) implements a technique called **minification** to remove unnecessary characters from JavaScript files (spaces, line breaks, etc.), reducing file size and speed up page loads.

<aside>
  There is a convention to add a `min` suffix to minified files. (eg `filename.min.js`)
</aside>

#### Directory Structure and Sprockets
The [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) standardizes the organization of these assets in `app/assets/`. 

```
app/
  assets/
    config/
      manifest.js
    images/
    stylesheets/
```

[Sprockets](https://github.com/rails/sprockets), a key component of the Pipeline, handles the concatenation and compression of assets. Projects using the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) will have the `sprockets-rails` gem in the `Gemfile`. There is a manifest file at `app/assets/config/manifest.js` that indexes all the assets you want to concatenate and minify. `//=` directives are used in `app/assets/config/manifest.js` to include images, stylesheets, and javascript.

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
    //= require jquery
    //= require bootstrap
  ```
</aside>


### Webpacker
This led to the introduction of [Webpacker](https://github.com/rails/webpacker) in Rails 5, allowing the integration of modern JavaScript frameworks (like [React](https://react.dev/) or [Vue](https://vuejs.org/)), tools like [Babel](https://babeljs.io/) for transpiling and [npm](https://www.npmjs.com/) for managing third party libraries.

<aside>
  [Babel](https://babeljs.io/) is a JavaScript transpiler that is used to convert ES6 code into backwards-compatible JavaScript code that can be run by older JavaScript engines. It allows web developers to take advantage of the newest features of JavaScript while still supporting older browsers like Internet Explorer.
</aside>

<aside>
  **ES6** (also known as ECMAScript 6) is a recent update to the JavaScript programming language. ES6 modules allow you to break your JavaScript into smaller, reusable components. One approach to organize JavaScript in your project is to create separate modules in the `javascript/` directory. Then, export functions, objects, or classes from these modules and import them in `app/javascript/application.js` or any other module where you need them.

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

### Import Maps
[Import Maps](https://github.com/rails/importmap-rails) in Rails 7 addresses some of the Asset Pipeline's limitations by simplifying the inclusion of JavaScript dependencies. It leverages modern browser capabilities to load JavaScript modules directly from the browser at runtime (loading them from a CDN), without the need for compilation or bundling. 

<aside>
  **CDNs** (or Content Delivery Networks) store copies of web content on multiple servers across different geographical locations. When a user accesses a web page, the CDN delivers content from the server closest to them, reducing load times.
</aside>

## Practical Example: Toggling Text Visibility
We'll implement a feature where users can toggle the visibility of a paragraph of text on a webpage. We'll implement this same feature using 5 distinct approaches.

1. "Vanilla" JavaScript
2. The Asset Pipeline with Import Maps
3. Stimulus.js
4. Alternative Bundling with jsbundling-rails, webpack, and React
5. API-only

Let's start by creating a new repository using the [Rails 7 template](https://github.com/new?template_name=rails-7-template&template_owner=appdev-projects) and name it something like "toggle-text-example" and then open it up in a codespace.

### Example 1: "Vanilla" JavaScript
<aside>
  "Vanilla" JavaScript means using plain JavaScript without any additional libraries like jQuery.
</aside>

Let's create a `PagesController` with a `home` action and view then set that action to the root route.

```ruby
# app/controllers/pages_controller.rb
class PagesController < ApplicationController

  def home; end

end
```

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root "pages#home"
end
```

Create a `home` view and include an `onclick` handler in the `<button>` and a `.hidden` css class:

```html
<!-- app/views/pages/home.html.erb -->

<style>
  .hidden {
    display: none;
  }
</style>

<p id="my-hidden-text" class="hidden">
  This is hidden text.
</p>

<button>Toggle Visibility</button>
```

If we run `rails server` in the terminal and visit our root route we should see our button.

![](assets/vanilla-js-example-1.png)


However, when we click the button, nothing happens. Now we need to add javascript functionality to this button. Define the `toggleTextVisibility` function in a `<script>` tag within the HTML file. Using the `onclick` attribute, we can directly attach a click event handler to the button in our HTML. 

```html
<!-- app/views/pages/home.html.erb -->

<style>
  .hidden {
    display: none;
  }
</style>

<p id="my-hidden-text" class="hidden">
  This is hidden text.
</p>

<!-- toggleTextVisibility() will be called when we click the button -->
<button onclick="toggleTextVisibility()">Toggle Visibility</button>

<script>
  function toggleTextVisibility() {
    const text = document.getElementById('my-hidden-text');
    text.classList.toggle('hidden');
  }
</script>
```

In this code, the `toggleTextVisibility` function will be called when the button is clicked.

![](assets/vanilla-js-example-2.gif)

Now that this is working, let's refactor our code a bit for better organization. We can start by moving the CSS for `.hidden` to `app/assets/stylesheets/application.css`:

```css
/* app/assets/stylesheets/application.css */

.hidden {
  display: none;
}
```

and remove the `<style>` tag from your `app/views/pages/home.html.erb` view.

```html
<!-- app/views/pages/home.html.erb -->

<p id="my-hidden-text" class="hidden">
  This is hidden text.
</p>

<!-- toggleTextVisibility() will be called when we click the button -->
<button onclick="toggleTextVisibility()">Toggle Visibility</button>

<script>
  function toggleTextVisibility() {
    const text = document.getElementById('my-hidden-text');
    text.classList.toggle('hidden');
  }
</script>
```

### Example 2: The Asset Pipeline with Import Maps
Now let's take advantage of the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) with [Import Maps](https://github.com/rails/importmap-rails) to include a simple JavaScript framework called [Stimulus.js](https://stimulus.hotwired.dev/). Stimulus.js enhances HTML by connecting elements to JavaScript objects via data attributes. JavaScript functionalities are encapsulated in controllers and targets, making the code more organized and maintainable.

Ensure the `importmap-rails` gem is in your `Gemfile` (it's included/installed by default in Rails 7). This will allow us to fetch stimulus at runtime using ES6 import statements.

```ruby
# Gemfile
gem 'importmap-rails'
```

Then run the install script.

```bash
$ bundle install
$ bin/rails importmap:install
```

This will create a `config/importmap.rb` file that will "pin" your `app/javascript/application.js` to the import map.

```ruby
# Pin npm packages by running ./bin/importmap

pin "application", preload: true
```

Ensure your layout file includes the `javascript_importmap_tags` in the `<head>`. It should look something like this.

```html
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title>Rails Template</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```

Now if you visit the root route of your application it will import your `application.js` in the head using a`<script type="importmap">` tag in the rendered html.

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- other head elements -->
    <script type="importmap" data-turbo-track="reload">{
      "imports": {
        "application": "/assets/application-3897b39d0f7fe7e947af9b84a1e1304bb30eb1dadb983104797d0a5e26a08736.js"
      }
    }</script>
    <link rel="modulepreload" href="/assets/application-3897b39d0f7fe7e947af9b84a1e1304bb30eb1dadb983104797d0a5e26a08736.js">
    <script src="/assets/es-module-shims.min-d89e73202ec09dede55fb74115af9c5f9f2bb965433de1c2446e1faa6dac2470.js" async="async" data-turbo-track="reload"></script>
    <script type="module">import "application"</script>
  </head>

  <body>
    <!-- your html body -->
  </body>
</html>
```

This will allow us to use ES6 style imports in our application. Let's refactor that `toggleTextVisibility` function we wrote into it's own file and import it in the `app/javascript/application.js` file.

<!-- TODO: show how we can import the function we wrote using ES6 and how it's somewhat limited -->


<!--
### Example 3: Stimulus.js
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

`app/views/pages/home.html.erb`
```erb

<div data-controller="toggle-hidden">
  <p data-toggle-hidden-target="text" class="hidden">
    This is hidden text.
  </p>

  <button data-action="click->toggle#toggle">Toggle Visibility</button>
</div>
```

-->


2. Configure
Define your dependencies in `config/importmap.rb`. For example, to add a library like lodash use the importmap CLI tool:

```bash
$ bin/importmap pin lodash
```
Which will add lodash to `config/importmap.rb`:

```ruby
# config/importmap.rb
pin "lodash", to: "https://cdn.skypack.dev/lodash"
```

3. Using the Dependency
In your JavaScript files (e.g., `app/javascript/application.js`), you can now import the module:

```javascript
import _ from 'lodash';

console.log(_.shuffle([1, 2, 3, 4]));
```

4. Execute

Run your Rails server and the JavaScript code using lodash will work as expected.

### Example 3: Alternative Bundling with jsbundling-rails, webpack, and React
For more complex JavaScript setups like integrating [React](https://react.dev/), Rails 7 offers tools like `jsbundling-rails`.

1. Setting Up jsbundling-rails with React
Add the `jsbundling-rails` gem to your Gemfile:

```ruby
gem 'jsbundling-rails'
```

Run `bundle install` in your terminal to install the gem.

Ensure your layout file includes the JavaScript pack tag.
```erb
<%= javascript_pack_tag 'application', 'data-turbo-track': 'reload' %>
```

2. Install JavaScript Bundler
For this example, we'll use Webpack. 

Run:

```sh
$ bin/rails javascript:install:webpack
```

<aside>
  [Webpacker](https://github.com/rails/webpacker) is a wrapper gem for integrating [Webpack](https://webpack.js.org/) with Rails applications. There are also alternative bundling libraries like [esbuild](https://esbuild.github.io/) that offer similar functionalities.
</aside>

3. Add React
You can add [React](https://react.dev/) and other dependencies via npm or Yarn. For instance:

```sh
yarn add react react-dom
```

4. Set Up React
In your `app/javascript` directory, create a React component `app/javascript/components/HelloReact.jsx`

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

function HelloReact() {
  return <h1>Hello from React</h1>;
}

document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    <HelloReact />,
    document.body.appendChild(document.createElement('div')),
  )
});
```

5. Include in Application Pack
In your `app/javascript/application.js`, import the React component:

```javascript
import 'components/HelloReact'
```

6. Execution
Start your Rails server and visit the page where the React component should appear.

### Example 4: API-only approach
<!-- TODO -->

### Choosing the Right Approach
<!-- Traditional approach for vanilla js -->
- Choose the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) with [Import Maps](https://github.com/rails/importmap-rails) if your project aligns with the Rails asset management approach, particularly for less complex JavaScript integrations.
- Choose Alternative Bundling libraries like [jsbundling-rails](https://github.com/rails/jsbundling-rails) if your project requires integration with Single Page Application (SPA) frameworks like [React](https://react.dev/) or [Vue](https://vuejs.org/) or extensive JavaScript tooling like [NPM](https://www.npmjs.com/).
<!-- Choose API-only approach if... -->

<!-- TODO: add a gif/image of the app -->

## Resources

- [The Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html)
- [Working with JavaScript in Rails](https://guides.rubyonrails.org/working_with_javascript_in_rails.html)
- [Stimulus](https://stimulus.hotwired.dev/)

## Conclusion
In this lesson, you've learned 4 different ways to add JavaScript to your Rails application. Understanding these different approaches allows you to choose the right tool for your project's needs.
