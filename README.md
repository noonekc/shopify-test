# Build Process

This build process aims to be an agnostic way to ensure modern features in our
Shopify projects. It is entirely opt-in. This enables it to be dropped into an
existing theme but only affect new files created in the scripts/styles folders.
Any old assets wouldn't be converted to build process unless explicitly done so.
This also allows one to 'opt-out' of the process during development if needed.
so at any point, a file can be moved back to `assets`, `theme.liquid` file
reference updated, and theme deployed to circumvent the build process.

## Overview

### Scripts and Styles folders

The root directory for these folders will be iterated through on build. Each file
located in root will have an equitable `.min` file created for it in the assets
folder. This provides the opportunity to only load the scripts and styles
needed for a specific page instead of loading everything for the entire site at
one time. For the components that need to be on every page, you can do this by
creating a subdirectory in the `scripts` or `styles` folder and importing it to
a `main.scss`/`theme.scss`/`index.js`/`theme.js` file in the root of the parent
directory (see Note).

The build process is possible because these are javascript, css, or scss files, 
not `.liquid` files. Liquid in the build process files will keep it from
working. If `.js.liquid`, `.css.liquid` or `.scss.liquid` files need to be moved
over, please follow [this process](./setup-docs/liquid-settings-for-js-css.md) for sanitizing the files of liquid and
referencing them in a way the build process can handle.

_Note: There is an issue currently where changes to files in subdirectories will
not force a rebuild for files in the parent directory which might include
references to the child directory files. For now, the workarounds are to rebuild
each time or place the file in assets while in development and reference it
directly in theme, then move it back to build process as you prepare for
deployment._

### File Structure

Potential file structure for a project:

``` html
|-- YourProject
    |-- assets
        |-- index.min.js
        |-- my-page.min.js
        |-- product.min.js
        |-- main.min.css
        |-- my-page.min.css
        |-- product.min.css
    |-- config
    |-- layout
        | theme.liquid (references .min files in assets)
    |-- locales
    |-- scripts
        |-- components
            |-- header.js
        |-- index.js (includes header.js)
        |-- my-page.js
        |-- product.js
    |-- snippets
    |-- styles
        |-- components
            |-- header.scss
        |-- main.scss (includes header.scss)
        |-- my-page.scss
        |-- product.scss
    |-- templates

```

## Essential Setup

### Setup in New Project

1. Setup project (recommend [Skeleton Theme](https://github.com/Shopify/skeleton-theme/tree/master/src/styles))
2. Clone this repo
3. (If needed) Install bundler 2 - `gem install bundler` 
4. Install gems - `bundle install`
5. Run `ruby build_process_app.rb migrate [PATH_TO_PROJECT_DIR]` to copy files
from `build-process-files` to the directory of your project
6. Add `main.scss` to `styles` folder

(Optional)
7. Add `index.js` to `scripts` folder

### Setup in Existing Project

1. Clone this repo
2. (If needed) Install bundler 2 - `gem install bundler`  
3. Install gems - `bundle install`
4. Run `ruby build_process_app.rb migrate [PATH_TO_PROJECT_DIR]` to copy files
from `build-process-files` to the directory of your project
5. Move to the folder of the theme you downloaded
6. Add `main.scss` to `styles` folder

(Optional)
7. Add `index.js` to `scripts` folder

### Next Steps

1. Add snippet `css-variables.liquid` to snippets folder
    - This file allow us to sanitize our css and javascript files from any liquid 

Code for file: 
```
{% comment %}

:root {
--color-body-text: {{ settings.color_body_text }};
--color-background: {{ settings.color_background_color }}
}
{% endcomment %}
<style>
  :root {
    --color-black: #000000;
    --color-white: #ffffff;
  }
</style>
```

2. Add snippet `js-variables.liquid` to snippets folder

Code for file: 
```
{% capture js_variables %}

<script>
Shopify = window.Shopify || {};
{% comment %} /* # Theme settings
================================================== */ {% endcomment %}
Shopify.theme_settings = {};
{% comment %} Example: {% endcomment %}
{% comment %} Cart 
Shopify.theme_settings.display_tos_checkbox = {{ settings.display_tos_checkbox | json }};
Shopify.theme_settings.go_to_checkout = {{ settings.go_to_checkout | json }};
Shopify.theme_settings.cart_action = {{ settings.cart_action | json }};
{% endcomment %}
</script>

{% endcapture %}
{%- assign js_variables = js_variables | split: 'Shopify.' -%}
{%- for variable in js_variables -%}
  {%- assign variableblock = variable | strip -%}
  {% if forloop.first %}
    {{ variableblock }}
  {% else %}
    {{ variableblock | prepend: 'Shopify.' }}
  {% endif %}
{%- endfor -%}
```

2. Add code for css and js variables to `theme.liquid` in the `head`.
```
 {% include 'css-variables' %}
```

```
 {% include 'js-variables' %}
```

3. Run npm install (or yarn install)
4. Make sure Shopify tooling is installed
- `brew tap shopify/shopify`
- `brew install themekit`
5. Install Gulp CLI
- `npm install gulp-cli -g`
6. Build minified files
- `npm run build`
7. Include minified files in `theme.liquid`
- `{{ 'main.min.css' | asset_url | stylesheet_tag }}`
- `{{ 'index.min.js' | asset_url | script_tag }}`

## Commands

* `npm run start`        - Starts Gulp watcher on `scripts` and `styles` directories.
* `npm run build`        - Gulp builds the .min files from `scripts` and `styles`, but doesn't watch.
* `npm run watch`        - Runs theme deploy and theme watch on development config.
* `npm run test`         - Runs Cypress open and will start any tests.
* `npm run deploy-dev`   - `theme deploy --env=development`
* `npm run deploy-stage` - `theme deploy --env=staging`

## Project Setup

To build this project:

1. Clone repo locally

2. Install Shopify tooling:
   **Using Homebrew**

   - `brew tap shopify/shopify`
   - `brew install themekit`

3. Install [Themekit](https://shopify.github.io/themekit/)

4. Run `npm install`

5. Set up config.yml

``` yaml
# password, theme_id, and store variables are required.
#
# For more information on this config file:
# https://shopify.github.io/themekit/commands/#configure

development:
  password: [your-api-password]
  theme_id: "[your-theme-id]"
  store: [your-store].myshopify.com
  ignores:
    -themekit.ignores
  ignore_files:
      - config/settings_data.json
      - config/settings_schema.json  
  
staging: 
  password: [your-api-password]
  theme_id: "[your-theme-id]"
  store: [your-store].myshopify.com
  ignores:
    -themekit.ignores
  ignore_files:
      - config/settings_data.json
      - config/settings_schema.json  
  
production: 
  password: [your-api-password]
  theme_id: "[your-theme-id]"
  store: [your-store].myshopify.com
  timeout: 100s
  readonly: true
```

6. Get password from private app (All developers at The Taproom use the same app for each client)

- **New Client**
  - Shopify admin => Apps => Private Apps => Manage Private Apps => Create New
    Private App
    - Enter App Name (Taproom Development) & Contact Email (kelly@thetaproom.com)
    - _Theme templates and theme assets_ set to **Read Write** access.
    - Save
    - Copy **Password**

    See gif below for walkthrough

- **Previous Client**
  - Shopify admin => Apps => Private Apps => Manage Private Apps => Taproom App
    => Password

    Gif for walkthrough:
    ![Custom App Walkthrough](../setup-docs/shopify-local-theme-development-generate-api.gif)

7. Run `theme deploy