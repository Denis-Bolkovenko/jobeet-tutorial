# Jobeet Day 14: Translations

Yesterday, we finished the mailing feature by connecting external service.
Now, we will talk about Jobeet internationalization (or i18n) and localization (or l10n).

From [Wikipedia][3]:

> **Internationalization** is the process of designing a software application so that it can be adapted to various languages and regions without engineering changes.
>
> **Localization** is the process of adapting software for a specific region or language by adding locale-specific components and translating text.

## Configuration

Symfony comes with [translations][4] package out of the box. We don't have to install it.  
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

The Jobeet website will be available in English and Russian.
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
You will see `en` value, because it's configured as default locale.
Then open URL [http://127.0.0.1/ru/][5] and `ru` will be displayed.
That means symfony understands the locale from URL.

## Language Switching

...

## Internationalization

...

```yaml
bin/console translation:update --dump-messages --output-format xlf --force en
```

### Templates

...

### Translations with Arguments

...

## Final Thoughts

Internationalization and localization are first-class citizens in Symfony.
Providing a localized website to your users is very easy as symfony provides all the basic tools and even gives you command line tasks to make it fast.

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
