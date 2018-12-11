# Jobeet Day 14: Translations

Yesterday, we finished the mailing feature by connecting external service.
Now, we will talk about Jobeet internationalization (or i18n) and localization (or l10n).

From [Wikipedia][3]:

> **Internationalization** is the process of designing a software application so that it can be adapted to various languages and regions without engineering changes.
>
> **Localization** is the process of adapting software for a specific region or language by adding locale-specific components and translating text.

## Configuration

Symfony comes with [translations][4] package out of the box. We don’t have to install it.  
Also we have initial configuration in `config/packages/translation.yaml` file:

```yaml
framework:
    default_locale: '%locale%'
    translator:
        # ...
        fallbacks:
            - '%locale%'
```

Variable `%locale%` is defined in file `config/services.yaml` and value is `'en'`.
We can start with that configuration without any changes.

## Locale in the URL

The Jobeet site will be available in English and Russian.
As an URL can only represent a single resource, the culture must be embedded in the URL.
In order to do that, open the `config/routes/annotations.yaml` file, and add `prefix` for all routes (except API).

```yaml
controllers:
    resource: ../../src/Controller/
    type: annotation
    prefix:
        en: ''
        ru: '/ru'

api_controllers:
    # ...
```

Do the same change in `config/routes.yaml` for routes defined by FOSUserBundle:

```yaml
# ...

fos_user:
    resource: "@FOSUserBundle/Resources/config/routing/security.xml"
    prefix:
        en: ''
        ru: '/ru'
```

When the `prefix` with locale variable is used in a route, Symfony will automatically use its value.  
If you want to test that, then just add somewhere in `templates/base.html.twig` next code `{{ dump(app.request.locale) }}` and open homepage.
You will see `en` value, because it’s configured as default locale.
Then open URL [http://127.0.0.1/ru/][5] and `ru` will be displayed.
That means symfony understands the locale from URL.

Also it’s possible to check current locale through Profiler:
click on Profiler Bar in the bottom of the page, find `Request / Response` tab in the left menu and in section `Request Attributes` you can see `_locale` parameter.

## Language Switching

For the user to change the language, a [dropdown][6] must be added in the layout.

First of all define the list of languages supported by the system.
Add it in `config/services.yaml` file:

```diff
  parameters:
      locale: 'en'
+     locales: ['en', 'ru']
      max_jobs_on_homepage: 10
      # ...
```

And define as global twig variable in `config/packages/twig.yaml`:

```diff
  twig:
      paths: ['%kernel.project_dir%/templates']
      debug: '%kernel.debug%'
      strict_variables: '%kernel.debug%'
      globals:
          max_jobs_on_homepage: '%max_jobs_on_homepage%'
          jobs_web_directory: '%jobs_web_directory%'
+         locales: '%locales%'
      form_themes:
          - 'bootstrap_3_horizontal_layout.html.twig'
```

Later we will use this variable to render selector.

Till now we connected only CSS file from bootstrap library using CDN, but now we need JS file too which requires [jQuery][8].
Let’s see how to connect CSS/JS libraries in Symfony way!

Add [Node.js][7] container in `docker-compose.yml` file:

```yaml
version: "3.1"
services:
    # ...

    node:
        image: node:9.11.1
        container_name: jobeet-node
        working_dir: /application
        volumes:
        - .:/application
```

This container has **Node.JS** and **npm** inside.
Npm is a package manager for JavaScript (like Composer for PHP).

Build and run Node.js container:

```bash
docker-compose up -d
```

Enter in Node.js container:

```bash
docker-compose run node bash
```

Install packages from package.json file:

```bash
npm install
```

Install jQuery and Bootstrap:

```bash
npm install --save-dev jquery@^3.3.1
npm install --save-dev bootstrap@^3.3.7
```

Create main js file (`assets/js/app.js`), which will be included on all pages.
Trigger `dropdown` from bootstrap library on all elements with `dropdown-toggle` class:

```js
// require jQuery normally
var $ = require('jquery');

// create global $ and jQuery variables
global.$ = global.jQuery = $;

require('bootstrap');

$(document).ready(function() {
  $(".dropdown-toggle").dropdown();
});
```

Symfony provides us a file with initial webpack configuration - `webpack.config.js`.
Register out new `app.js` file as entry point and activate jQuery:

```diff
  var Encore = require('@symfony/webpack-encore');
  
  Encore
      // the project directory where compiled assets will be stored
      .setOutputPath('public/build/')
      // the public path used by the web server to access the previous directory
      .setPublicPath('/build')
      .cleanupOutputBeforeBuild()
      .enableSourceMaps(!Encore.isProduction())
      // uncomment to create hashed filenames (e.g. app.abc123.css)
      // .enableVersioning(Encore.isProduction())
  
      // uncomment to define the assets of the project
-     // .addEntry('js/app', './assets/js/app.js')
+     .addEntry('js/app', './assets/js/app.js')
      // .addStyleEntry('css/app', './assets/css/app.scss')
  
      // uncomment if you use Sass/SCSS files
      // .enableSassLoader()
  
      // uncomment for legacy applications that require $/jQuery as a global variable
-     // .autoProvidejQuery()
+     .autoProvidejQuery()
+     .autoProvideVariables({
+         $: 'jquery',
+         jQuery: 'jquery',
+         'window.jQuery': 'jquery'
+     })
  ;
  
  module.exports = Encore.getWebpackConfig();
```

Build assets with next command in Node.js container:

```bash
npm run dev
```

You can notice that new `app.js` file appeared in folder `public/build/js`.
This file is build using webpack and contains all JavaScript code required for out application.
Include this file in base layout (`templates/base.html.twig`) and add dropdown to select the language:

```diff
  <!DOCTYPE html>
  <html>
  <head>
      <title>{% block title %}{{ 'job.base.title'|trans }}{% endblock %}</title>
  
      <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  
      {% block stylesheets %}{% endblock %}
  
+     <script src="{{ asset('build/js/app.js') }}"></script>
      {% block javascripts %}{% endblock %}
  
      <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
  <nav class="navbar navbar-default">
      <div class="container-fluid">
          <div class="navbar-header">
              <a class="navbar-brand" href="{{ path('job.list') }}">{{ 'job.base.list'|trans }}</a>
          </div>
  
          <div class="collapse navbar-collapse">
              <ul class="nav navbar-nav navbar-right">
                  {% if is_granted('ROLE_ADMIN') %}
                      <li>
                          <div>
                              <a href="{{ path('admin.category.list') }}" class="btn btn-default navbar-btn">{{ 'admin_panel'|trans }}</a>
                          </div>
                      </li>
                  {% endif %}
  
                  <li>
                      <div>
                          <a href="{{ path('affiliate.create') }}" class="btn btn-default navbar-btn">{{ 'affiliates'|trans }}</a>
                      </div>
                  </li>
  
                  <li>
                      <div>
                          <a href="{{ path('job.create') }}" class="btn btn-default navbar-btn">{{ 'job.base.create'|trans }}</a>
                      </div>
                  </li>
  
+                 <li class="dropdown">
+                     <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">{{ app.request.locale|upper }}<span class="caret"></span></a>
+                     <ul class="dropdown-menu">
+                         {% for locale in locales %}
+                             <li>
+                                 <a href="{{ path(app.request.get('_route'), app.request.attributes.get('_route_params')|merge({'_locale': locale})) }}">{{ locale|upper }}</a>
+                             </li>
+                         {% endfor %}
+                     </ul>
+                 </li>
  
                  {% if app.user %}
                      <li><a href="{{ path('fos_user_security_logout') }}">{{ 'logout'|trans }}</a></li>
                  {% endif %}
              </ul>
          </div>
      </div>
  </nav>
  
  <div class="container">
      {% block body %}{% endblock %}
  </div>
  </body>
  </html>
```

Now selector works and language is mentioned in URL.

## Internationalization

### Templates

An internationalized site means that the user interface is translated into several languages.

In a template, all strings that are language dependent must be passed through `trans` filter.

Here is how to use it for the Jobeet layout:

```diff
  <!DOCTYPE html>
  <html>
  <head>
-     <title>{% block title %}Jobeet - Your best job board{% endblock %}</title>
+     <title>{% block title %}{{ 'title'|trans }}{% endblock %}</title>
  
      <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  
      {% block stylesheets %}{% endblock %}
  
      <script src="{{ asset('build/js/app.js') }}"></script>
      {% block javascripts %}{% endblock %}
  
      <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
  <nav class="navbar navbar-default">
      <div class="container-fluid">
          <div class="navbar-header">
-             <a class="navbar-brand" href="{{ path('job.list') }}">Jobeet</a>
+             <a class="navbar-brand" href="{{ path('job.list') }}">{{ 'navbar_label'|trans }}</a>
          </div>
  
          <div class="collapse navbar-collapse">
              <ul class="nav navbar-nav navbar-right">
                  {% if is_granted('ROLE_ADMIN') %}
                      <li>
                          <div>
-                             <a href="{{ path('admin.category.list') }}" class="btn btn-default navbar-btn">Admin Panel</a>
+                             <a href="{{ path('admin.category.list') }}" class="btn btn-default navbar-btn">{{ 'admin_panel'|trans }}</a>
                          </div>
                      </li>
                  {% endif %}
  
                  <li>
                      <div>
-                         <a href="{{ path('affiliate.create') }}" class="btn btn-default navbar-btn">Affiliates</a>
+                         <a href="{{ path('affiliate.create') }}" class="btn btn-default navbar-btn">{{ 'affiliates'|trans }}</a>
                      </div>
                  </li>
  
                  <li>
                      <div>
-                         <a href="{{ path('job.create') }}" class="btn btn-default navbar-btn">Post a Job</a>
+                         <a href="{{ path('job.create') }}" class="btn btn-default navbar-btn">{{ 'job.create'|trans }}</a>
                      </div>
                  </li>
  
                  <li class="dropdown">
                      <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">{{ app.request.locale|upper }}<span class="caret"></span></a>
                      <ul class="dropdown-menu">
                          {% for locale in locales %}
                              <li>
                                  <a href="{{ path(app.request.get('_route'), app.request.attributes.get('_route_params')|merge({'_locale': locale})) }}">{{ locale|upper }}</a>
                              </li>
                          {% endfor %}
                      </ul>
                  </li>
  
                  {% if app.user %}
-                     <li><a href="{{ path('fos_user_security_logout') }}">Logout</a></li>
+                     <li><a href="{{ path('fos_user_security_logout') }}">{{ 'logout'|trans }}</a></li>
                  {% endif %}
              </ul>
          </div>
      </div>
  </nav>
  
  <div class="container">
      {% block body %}{% endblock %}
  </div>
  </body>
  </html>
```

When Symfony renders a template, each time the `trans` filter is called, Symfony looks for a translation for the current request locale.
If a translation is found, it is used, if not, the initial value is returned as a fallback value.

All translations are stored in `translations` folder.
The translations package provides a lot of different strategies to store the translations.
We will use the "XLIFF" format, which is a standard and the most flexible one.

> Other strategies are gettext, yaml, json, etc.

### translation:update

Instead of creating the translation file by hand, use the built-in `translation:update` command:

```bash
bin/console translation:update --output-format xlf --force en
```

The `translation:update` command finds all strings that need to be translated in en in templates and creates or updates the corresponding translation files.
The `--force` option saves the new strings in the translation files.
You can also use the `--clean` option to automatically remove strings that do not exist anymore.

In our case, it populates the file we have created:

```xml
<?xml version="1.0" encoding="utf-8"?>
<xliff xmlns="urn:oasis:names:tc:xliff:document:1.2" version="1.2">
  <file source-language="en" target-language="en" datatype="plaintext" original="file.ext">
    <header>
      <tool tool-id="symfony" tool-name="Symfony"/>
    </header>
    <body>
      <trans-unit id="YM.OFL6" resname="navbar_label">
        <source>navbar_label</source>
        <target>__navbar_label</target>
      </trans-unit>
      <trans-unit id="4iIkq4Y" resname="admin_panel">
        <source>admin_panel</source>
        <target>__admin_panel</target>
      </trans-unit>
      <trans-unit id="Frdv8uy" resname="affiliates">
        <source>affiliates</source>
        <target>__affiliates</target>
      </trans-unit>
      <trans-unit id="Fipexar" resname="job.create">
        <source>job.create</source>
        <target>__job.create</target>
      </trans-unit>
      <trans-unit id="ACcM9j_" resname="logout">
        <source>logout</source>
        <target>__logout</target>
      </trans-unit>
      <trans-unit id="qvIyBkY" resname="title">
        <source>title</source>
        <target>__title</target>
      </trans-unit>
    </body>
  </file>
</xliff>
```

Each translation is managed by a `trans-unit` tag which has a unique `id` attribute.
You can now edit this file and add translations for the English language:

```xml
<?xml version="1.0" encoding="utf-8"?>
<xliff xmlns="urn:oasis:names:tc:xliff:document:1.2" version="1.2">
  <file source-language="en" target-language="en" datatype="plaintext" original="file.ext">
    <header>
      <tool tool-id="symfony" tool-name="Symfony"/>
    </header>
    <body>
      <trans-unit id="YM.OFL6" resname="navbar_label">
        <source>navbar_label</source>
        <target>Jobeet</target>
      </trans-unit>
      <trans-unit id="4iIkq4Y" resname="admin_panel">
        <source>admin_panel</source>
        <target>Admin panel</target>
      </trans-unit>
      <trans-unit id="Frdv8uy" resname="affiliates">
        <source>affiliates</source>
        <target>Affiliates</target>
      </trans-unit>
      <trans-unit id="Fipexar" resname="job.create">
        <source>job.create</source>
        <target>Post a Job</target>
      </trans-unit>
      <trans-unit id="ACcM9j_" resname="logout">
        <source>logout</source>
        <target>Logout</target>
      </trans-unit>
      <trans-unit id="SPM6JVE" resname="title">
        <source>title</source>
        <target>Jobeet - Your best job board</target>
      </trans-unit>
    </body>
  </file>
</xliff>
```

> As XLIFF is a standard format, a lot of tools exist to ease the translation process.
> You can use this [Free Online XLIFF Editor][9] to manage translations.

> As XLIFF is a file-based format, the same precedence and merging rules that exist for other Symfony configuration files are also applicable.
> Translations files can exist in a project or a bundle, and the most specific file overrides translations found in the more global ones.

### Translations with Arguments

...

## Final Thoughts

Internationalization and localization are first-class citizens in Symfony.
Providing a localized site to your users is very easy as symfony provides all the basic tools and even gives you command-line tasks to make it fast.

That’s all for today, you can find the code here: [https://github.com/gregurco/jobeet/tree/day14][1]

## Additional information

- [Translations][2]

## Next Steps

Continue this tutorial here: Jobeet Day 15: The Unit Tests

Previous post is available here: [Jobeet Day 13: The Mailer](day-13.md)

Main page is available here: [Symfony 4.1 Jobeet Tutorial](../index.md)

[1]: https://github.com/gregurco/jobeet/tree/day14
[2]: https://symfony.com/doc/4.1/translation.html
[3]: https://en.wikipedia.org/wiki/Internationalization
[4]: https://packagist.org/packages/symfony/translation
[5]: http://127.0.0.1/ru/
[6]: https://getbootstrap.com/docs/3.3/javascript/#dropdowns
[7]: https://nodejs.org
[8]: https://jquery.com/
[9]: http://xliff.brightec.co.uk/
