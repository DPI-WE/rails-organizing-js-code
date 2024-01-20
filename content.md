# JavaScript level up with Stimulus.js
In our previous lessons, we've seen how [JavaScript](https://learn.firstdraft.com/lessons/203-minimal-js) can enhance the interactivity of our Rails applications using [Ajax with Rails Unobtrusive JavaScript (UJS)](https://learn.firstdraft.com/lessons/204-rails-unobtrusive-ajax). Building on this foundation, we're now set to delve deeper into the organization of JavaScript code, the role of `javascript_include_tag`, various lifecycle methods, and three distinct approaches to managing JavaScript in a Rails environment. To bring these concepts to life, we'll implement a practical example, implementing interactive features in a Ruby on Rails application using both Vanilla JavaScript and Stimulus.js.

## Understanding JavaScript Organization in Rails
Typically, JavaScript files in Rails are placed under `app/javascript`. As your application grows, keeping your javascript code in this directory with a clear structure keeps your codebase manageable. Clear organization also helps team members (and instructors 🥹) to understand and contribute to the codebase effectively.

### Directory Structure
Rails 7 provides a standard way to organize your JavaScript files. Here’s a typical structure:

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
- **Controllers**: [Stimulus](https://github.com/hotwired/stimulus-rails) controllers are typically placed here.
- **Custom**: Create custom directories like `custom/` or `util/` for your specific scripts.
- **application.js**: The JavaScript manifest file. You can think of it as a central directory or index of your JavaScript files.

### Importing ES6 Modules
ES6 modules allow you to break your JavaScript into smaller, reusable components.
- Create separate modules in the `javascript/` directory.
- Export functions, objects, or classes from these modules.
- Import them in `app/javascript/application.js` or any other module where you need them.

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

## Understanding `javascript_include_tag` in Rails
The `javascript_include_tag` is a built-in Rails helper used to include JavaScript files in your HTML templates. It generates a `<script>` tag to link JavaScript files to your HTML pages. This is an essential part of integrating JavaScript into your Rails views.

### Basic Usage
In your application layout file `app/views/layouts/application.html.erb`, you would typically use `javascript_include_tag` to include the main JavaScript file:

```erb
<%= javascript_include_tag 'application' %>
```
This line automatically includes the `app/javascript/application.js` file, which is usually the main JavaScript file in a Rails application. Rails will handle the processing of this file, which might involve bundling (more on this later).

<aside>

### `data-turbo-track` Attribute
The `data-turbo-track` attribute is specifically related to [Turbo Drive](https://turbo.hotwired.dev/handbook/drive), a part of the [Hotwire toolkit](https://hotwired.dev/).

```erb
<%= javascript_include_tag 'application', 'data-turbo-track': 'reload' %>
```

### What Turbo Drive Does:
- **Speeds Up Navigation**: Turbo Drive is designed to make navigation in a Rails application faster by partially updating the page's HTML without a full reload.
- **Ajax-like Behavior**: It achieves this by intercepting clicks on links and form submissions, and instead of doing a full page load, it updates only the body and the head of the page.

### The Role of `data-turbo-track`
- **Cache Management**: The `data-turbo-track: 'reload'` attribute is used to tell Turbo Drive how to manage caching for scripts.
- **Reload Behavior**: When set to reload, it ensures that if the JavaScript file changes, Turbo Drive will fully reload the page rather than using its caching mechanism. This is crucial for ensuring that users always receive the most up-to-date version of your JavaScript.

### When Is This Important?
This feature becomes particularly important in applications where JavaScript files change often, and you need to ensure that these changes are reflected immediately to the user without them having to clear their cache or force-reload the page.

</aside>

In the rendered HTML of the deployed site, it translates to something like:

```html
<script src="/assets/application-abcdef1234567890.js"></script>
```
This line in the deployed HTML is responsible for loading your application's JavaScript, ensuring users get the latest version of the script as per the current deployment. You probably noticed the rendered html version includes a fingerprint "abcdef1234567890". This fingerprint is a hash generated based on the file's content for cache-busting purposes.

## JavaScript Management in Rails: Bundling, Importmaps, and the Asset Pipeline
Rails 7 offers several approaches for handling JavaScript, each catering to different needs: [Bundling](https://github.com/rails/jsbundling-rails), [Importmaps](https://github.com/rails/importmap-rails), and the traditional [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html). Understanding the differences and use cases of each helps in choosing the right tool for your project.

### [Bundling](https://github.com/rails/jsbundling-rails)
[Bundling](https://github.com/rails/jsbundling-rails) refers to the process of compiling and packaging multiple JavaScript files into one or a few files. This approach is efficient and reduces the number of requests a browser makes to load a page.
<!-- TODO: Talk about tradeoffs? slower development because of compile step, more complexity, etc. -->
- **Use Case**: Ideal for complex applications that utilize Single Page Application (SPA) JavaScript frameworks (like [React](https://react.dev/) or [Vue](https://vuejs.org/)) or need to handle a variety of NPM packages.
- **How It Works**: Bundling tools like Webpack compile and optimize your JavaScript files during the deployment process, ensuring efficient loading and potentially smaller file sizes.
- **Identifying Bundling**: Projects using bundling will have the `jsbundling-rails` gem in the `Gemfile` and specific configuration or build scripts for the chosen JavaScript bundler (eg [Webpack](https://github.com/webpack/webpack)).

### Importmaps
[Importmaps](https://github.com/rails/importmap-rails) leverage modern browser capabilities to load JavaScript modules directly from the browser at runtime, without the need for compilation or bundling. (Just like how we include [Bootstrap](https://getbootstrap.com/) or [Font Awesome](https://fontawesome.com/) in our `<head>`.)

- **Use Case**: Suited for simpler applications that require basic JavaScript functionality or want to follow Rails conventions more closely.
- **How It Works**: Importmaps allow the browser to manage JavaScript modules at runtime,  loading them from a CDN (Content Delivery Network). This approach reduces server load and simplifies JavaScript management.
- **Identifying Importmaps**: Look for the `importmap-rails` gem in the `Gemfile` and the presence of `config/importmap.rb` in the project.

### JavaScript and CSS Asset Pipeline
The [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) is a Rails framework that concatenates and minifies JavaScript and CSS assets, optimizing them for production.

- **Use Case**: Good for applications with simpler JavaScript needs or for those who prefer the traditional Rails way of handling assets.
- **How It Works**: It concatenates all JavaScript files into a single file and compresses them to reduce file size. This happens server-side, simplifying deployment.

<aside>
Differences from Node.js `require`: Unlike Node.js `require`, which is part of CommonJS module syntax and used for server-side JavaScript, the Asset Pipeline's `//= require` is specific to Rails.
</aside>

- **Identifying the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html)**: JavaScript files are placed in `app/assets/javascripts/`, and `//= require` statements are used in `app/assets/javascripts/application.js` to include them.

### Choosing the Right Approach
- Choose [Bundling](https://github.com/rails/jsbundling-rails) if your project requires integration with Single Page Application (SPA) frameworks like  [React](https://react.dev/) or [Vue](https://vuejs.org/), or handling numerous [NPM packages](https://www.npmjs.com/).
- Choose [Importmaps](https://github.com/rails/importmap-rails) for simpler projects that benefit from less configuration and direct browser handling of JavaScript modules.
- Choose the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) if your project aligns with the traditional Rails asset management approach, particularly for less complex JavaScript integrations.

Each method - [Bundling](https://github.com/rails/jsbundling-rails), [Importmaps](https://github.com/rails/importmap-rails), and the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) - offers distinct advantages and fits different scenarios in a Rails application. Your choice depends on the specifics of your JavaScript requirements, your preferred development workflow, and the specific needs of your project.

## Practical Example: Toggling Text Visibility
We'll implement a feature where users can toggle the visibility of a paragraph of text on a webpage.

### Part 1: "Vanilla" JavaScript Approach
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

<!-- TODO: add conclusion and a gif -->

### Part 2: Stimulus.js Approach
Stimulus.js is a modest JavaScript framework designed for Rails applications. It enhances HTML by connecting elements to JavaScript objects in a structured and organized manner. It's beneficial for its ease of use, integration with Rails conventions, and clarity in connecting HTML to JavaScript.

<!--
- Declarative Interaction: Stimulus allows for a more declarative approach, connecting HTML and JavaScript via data attributes.
- Controllers and Targets: JavaScript functionalities are encapsulated in controllers and targets, making the code more organized and maintainable.
- Automatic Initialization: Stimulus automatically initializes controllers and their actions without needing to manually set up event listeners or wait for DOMContentLoaded.
- Stronger Coupling: There's a stronger coupling between the HTML structure and JavaScript functionality, making it clearer how elements are intended to behave.
-->

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

## Resources

- [Working with JavaScript in Rails](https://guides.rubyonrails.org/working_with_javascript_in_rails.html)
- [Stimulus](https://stimulus.hotwired.dev/)

## Conclusion
In this lesson, you've learned two different ways to add interactive features to your Rails application. The vanilla JavaScript approach offers direct control over the DOM, while [Stimulus](https://stimulus.hotwired.dev/) provides a more structured and Rails-integrated method. Understanding both allows you to choose the right tool for your project's needs.
