---
subtitle: Tailor website content to your exact requirements.
---
# Introduction

<VideoBlockLink src="https://www.youtube.com/watch?v=_WMH4mlMdjk" title="Tailor Tutorial" description="This video describes how to quickly create a complete blog solution with Tailor." prompt="Watch the tutorial" />

Tailor is a feature that defines file-based content structures used by your website, such as a company blog or team page. Tailor automatically generates a backend user interface for managing records and provides CMS Components for displaying and linking records on the frontend.

When using Tailor you can skip the traditional plugin development workflow and go straight to defining content. Fields are defined simply as blueprint templates and content is stored in special database tables. The blueprint template can also specify navigation and other modifiers.

## Directory Structure

Blueprints are YAML files that reside in the **app/blueprints** directory by default, in addition to the **themes/.../blueprints** directory for a [CMS theme](../themes/themes.md). Where to place your blueprints depends on the context: **App Blueprints** are globally available, whereas **Theme Blueprints** are only available when that theme is selected.

### App Blueprints

Below you can see an example blueprint directory structure. Each blueprint can reside in any directory and any file name can be used. Blueprints can be organised in subdirectories of any nesting depth.

::: dir
├── app
|   └── `blueprints`  _← Blueprints Start Here_
|       ├── blog
|       │   └── blog.yaml
|       │   └── author.yaml
|       ├── about
|       │   └── about.yaml
|       ├── wiki
|       │   └── article.yaml
:::

### Theme Blueprints

Next is an example of a blueprint structure inside the **demo** theme. Just like in the app directory, blueprints can reside in any directory with any filename, with any nesting depth. The admin panel navigation for these blueprints will only appear when the theme is selected, often controlled by the [multisite settings](../resources/multisite.md).

::: dir
├── themes
|   └── demo
|       ├── `blueprints`  _← Blueprints Start Here_
|       │   └── settings.yaml
|       │   └── color-scheme.yaml
:::

## Blueprint Types

The blueprint **type** property determines how the blueprint should be implemented. There are several types available and most blueprints will specify [form field definitions](../../element/form-fields.md).

Type | Description
------------- | -------------
Entry | the standard content structure that supports drafts.
Global | a single record in the database and is often used for settings and configuration.
Mixin | defines reusable field definitions that can be imported and mixed in with other field definitions.

Each blueprint type is described in more detail in the [Blueprints article](./blueprints.md).

## Blueprint Structure

::: aside
Blueprints are 100% portable. They use internal identifiers and can reside in any directory with any file name.
:::

Blueprints are defined using the YAML syntax and will always contain three identifiers, a unique UUID, a user-friendly handle and the blueprint type. The filename and folder of a blueprint is used to organise blueprints and is not used as an identifier. All other properties are defined in the blueprint's relevant documentation article.

```yaml
uuid: edcd102e-0525-4e4d-b07e-633ae6c18db6
handle: Blog\Post
type: entry
name: Post

fields:
    # [...]
```

The blueprint **handle** is a human-readable approach to referencing a blueprint object. Using the above blueprint as a reference, we can reference the entries using the handle.

::: tip
Handles follow a similar naming convention to PHP namespaces and can be organised with the backslash `\` separator.
:::

```ini
[section blog]
handle = "Blog\Post"
```

The blueprint **uuid** is a unique identifier used when blueprints reference other blueprints. For example, when a field references a mixin. When first creating a blueprint, you can choose to not include a UUID and one will be magically created for you on the first migration.

```yaml
_blog_content:
    source: edcd102e-0525-4e4d-b07e-633ae6c18db6
    type: mixin
```

## Integration with Multisite

Blueprints do not use [multisite capabilities](../resources/multisite.md) by default. You may use the `multisite` property to enable this. When enabled, records can be unique to each configured site.

```yaml
handle: Blog\Post
type: entry
# ...
multisite: true
```

When multisite is enabled, all fields in the blueprint become translatable. To keep the same value for a field, set the `translatable` property to false. In this example, when saving the record the **name** field will be copied to every site when it is saved.

```yaml
# ...
multisite: true

fields:
    name:
        label: Full Name
        type: text
        translatable: false
```

You may also set the value to **sync** to keep the records synchronized across sites, which is helpful for categories and tags. When using sync, each record will always exist on every site, although the content can be different.

```yaml
multisite: sync
```

When using [Site Groups](../resources/multisite.md), the records will be propagated to all sites within that group. This can be changed by setting the `multisite` property to **all** to sync within all sites.

```yaml
multisite: all
```

Setting to **locale** will sync the records to all sites that share the same locale.

```yaml
multisite: locale
```

## Migrating Blueprints

Blueprints and their structure are migrated in the database during the normal database migration process. When a change is made manually to a blueprint file, you should run the `tailor:migrate` command to update the database tables.

```bash
php artisan tailor:migrate
```

::: tip
Blueprints are cached when debug mode is turned off. The migration command can also be used to clear the blueprint cache.
:::

### Refreshing Content

You may delete all the content managed by Tailor using the `tailor:refresh` command.

```bash
php artisan tailor:refresh
```

To refresh a single blueprint use the `--blueprint` option and specify its handle.

```bash
php artisan tailor:refresh --blueprint="Blog\Post"
```

### Propagating Content

When using the **sync** option for multisite, you may retroactively propagate records using the `tailor:propagate` command.

```bash
php artisan tailor:propagate
```

To propagate a single blueprint use the `--blueprint` option and specify its handle.

```bash
php artisan tailor:propagate --blueprint="Blog\Category"
```

### Pruning Content

As a general rule Tailor will never drop table columns and delete content. If a field is removed, the column will be renamed instead of dropped. For example, an old field named `content` may appear as `x_content_fb418fac` in the database table. Tables for old blueprints are also kept in case they are ever restored.

You may prune unused database columns with the `tailor:prune` command. This command will also drop tables that no longer have an associated blueprint.

```bash
php artisan tailor:prune
```

You may only prune fields using the `--fields` modifier. To only prune tables, use the `--tables` modifier.

```bash
php artisan tailor:prune --fields
php artisan tailor:prune --tables
```

To prune a single blueprint use the `--blueprint` option and specify its handle.

```bash
php artisan tailor:prune --blueprint="Blog\Post"
```
