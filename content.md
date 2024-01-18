# JavaScript level up with Stimulus.js
In our previous lessons, we've seen how [JavaScript](https://learn.firstdraft.com/lessons/203-minimal-js) can enhance the interactivity of our Rails applications using [Ajax with Rails Unobtrusive JavaScript (UJS)](https://learn.firstdraft.com/lessons/204-rails-unobtrusive-ajax). Now, let's dive deeper into organizing our JavaScript code, understanding the lifecycle methods, adding interactive features to a Ruby on Rails application using two approaches: Vanilla JavaScript and Stimulus.js. You'll learn about the benefits of each approach and implement a practical example in both ways.

## Understanding JavaScript Organization in Rails
Typically, JavaScript files in Rails are placed under `app/javascript`. As your application grows, keeping your javascript code in this directory with a clear structure keeps your codebase manageable. Clear organization also helps team members (and instructors 🥹) to understand and contribute to the codebase effectively.

### Directory Structure
Rails 7 provides a standard way to organize your JavaScript files. Here’s a typical structure:

```yml
app/
    - javascript:
        - channels:
        - controllers:
        - custom:
            - your_custom_script.js
        - application.js
```
- **Channels**: Used for ActionCable related files.
- **Controllers**: Stimulus controllers are typically placed here.
- Custom: Create custom directories like `custom/` for your specific scripts.

## Using ES6 Modules
ES6 modules allow you to break your JavaScript into smaller, reusable components. Rails 7 supports this out of the box with Webpacker.

- Create separate modules in the `javascript/` directory.
- Export functions, objects, or classes from these modules.
- Import them in `app/javascript/application.js` or any other module where you need them.

Example:

```javascript
// In hello.js
export function sayHello(name) {
  return `Hello, ${name}!`;
}

// In application.js
import { sayHello } from './hello';

console.log(sayHello('Alice'));
```

## `javascript_include_tag`
The `javascript_include_tag` is a Rails helper that includes JavaScript files into your HTML templates. Typically used in your layout file:

```erb
<%= javascript_include_tag 'application', 'data-turbo-track': 'reload' %>
```

This line includes `application.js`, which is your main JavaScript file. Rails handles the compilation and packaging of all the linked JavaScript files.

## Understanding Lifecycle Methods in JavaScript
Lifecycle methods are crucial in understanding how your JavaScript interacts with the DOM.

### The `DOMContentLoaded` Event
Triggers when the initial HTML document has been completely loaded and parsed.
Does not wait for stylesheets, images, and subframes to finish loading.

```javascript
document.addEventListener('DOMContentLoaded', (event) => {
  // Your code goes here
});
```

### The `load` Event
Fires when the entire page, including all dependent resources (like images), is fully loaded.

```javascript
window.addEventListener('load', (event) => {
  // Your code goes here
});
```

### The `beforeunload` and `unload` Events
- `beforeunload` can be used to alert the user before leaving a page.
- `unload` triggers when the document or a child resource is being unloaded.

### Additional Concepts

#### JavaScript and CSS Asset Pipeline
Rails 7 supports the asset pipeline, which is a framework to concatenate and minify or compress JavaScript and CSS assets.

Place your JavaScript files in `app/assets/javascripts/`.
Use `//= require` statements in application.js to include them.

#### Turbolinks and JavaScript
Turbolinks can make your application faster but requires a different approach in handling JavaScript:

Use the `turbolinks:load` event instead of DOMContentLoaded.
Remember to clean up or re-initialize JavaScript on page changes.

## Practical Example: Toggling Text Visibility
We'll implement a feature where users can toggle the visibility of a paragraph of text on a webpage.

### Part 1: Vanilla JavaScript Approach
The vanilla approach involves directly manipulating the DOM using JavaScript. It's straightforward but requires managing event listeners and ensuring the DOM is fully loaded before script execution.

<!--
- Direct DOM Manipulation: The script directly manipulates DOM elements using getElementById and classList.
- Event Listeners: Manually adding event listeners in the JavaScript file.
- Script Initialization: Ensuring that scripts are initialized on DOMContentLoaded.
- Loose Coupling with HTML: The JavaScript is somewhat decoupled from the HTML structure, relying on id and class selectors. 
-->


#### Step 1: Creating the View
Include a paragraph and a button:

```erb
<!-- app/views/pages/home.html.erb -->
<p id="hidden-text" class="hidden-text">
  This is hidden text.
</p>

<button id="toggle-button">Toggle Visibility</button>
```

#### Step 2: Adding JavaScript
Create a function to toggle the visibility and add an event listener to the button:

```javascript
// app/javascript/custom/toggle_visibility.js

function toggleTextVisibility() {
  const text = document.getElementById('hidden-text');
  text.classList.toggle('hidden-text');
}

document.addEventListener('DOMContentLoaded', () => {
  const button = document.getElementById('toggle-button');
  button.addEventListener('click', toggleTextVisibility);
});
```

#### Step 3: Including JavaScript in Application
Import the script:

```javascript
// app/javascript/packs/application.js
import "../custom/toggle_visibility"
```

#### Step 4: CSS
Add the .hidden-text class:

```css
/* app/assets/stylesheets/application.css */

.hidden-text {
  display: none;
}
```

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

```bash
./bin/rails webpacker:install:stimulus
```

#### Step 2: Creating a Stimulus Controller
Generate a Stimulus controller:

```bash
./bin/rails generate stimulus toggle
```

Edit the controller:

```javascript
// app/javascript/controllers/toggle_controller.js

import { Controller } from "stimulus";

export default class extends Controller {
  static targets = ["text"]

  toggle() {
    this.textTarget.classList.toggle('hidden-text');
  }
}
```

#### Step 3: Adjusting the View
Update the HTML to use Stimulus:

```erb
<!-- app/views/pages/home.html.erb -->

<div data-controller="toggle">
  <p data-toggle-target="text" class="hidden-text">
    This is hidden text.
  </p>

  <button data-action="click->toggle#toggle">Toggle Visibility</button>
</div>
```

## Resources

- [Working with JavaScript in Rails](https://guides.rubyonrails.org/working_with_javascript_in_rails.html)
- [Stimulus](https://stimulus.hotwired.dev/)

## Conclusion
In this lesson, you've learned two different ways to add interactive features to your Rails application. The traditional JavaScript approach offers direct control over the DOM, while Stimulus.js provides a more structured and Rails-integrated method. Understanding both allows you to choose the right tool for your project's needs.
