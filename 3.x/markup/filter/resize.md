---
subtitle: Twig Filter
---
# |resize

The `|resize` filter attempts to resize the provided image source using the provided resizing configuration and outputs a URL to the resized image.

```twig
{{ 'image.jpg'|resize(100, 100) }}
```

The filter accepts three arguments: `|resize(width, height, options)`.

If the filter can resize the provided image, then a URL to the image resizer (eg: `/resize/filename.png`) is returned. For subsequent requests, a direct URL to the resized image is returned. If the filter is unable to process the provided image, then it will instead return the original URL unmodified.

This will resize a banner.jpg media image to 1920 x 1080 ratio.

```twig
<img src="{{ 'banner.jpg'|media|resize(1920, 1080) }}" />
```

You can also pass a third options argument. This example will crop the image instead.

```twig
<img src="{{ 'banner.jpg'|resize(800, 600, { mode: 'crop' }) }}" />
```

See the [image resizer article](../../extend/services/resizer.md) for more information on the available `options` parameters.

## Custom Filenames

The resizer will assign a random filename to resized images by default. You may use the original filename by passing the `true` to `filename` option to the resizer.

```twig
<img src="{{ 'banner.jpg'|resize(800, 600, { filename: true }) }}" />
```

The `filename` option also supports a custom filename as a string. The filename should not contain an extension since the original one is used.

```twig
<img src="{{ 'banner.jpg'|resize(800, 600, { filename: 'my-seo-friendly-name' }) }}" />
```

You may modify the extension with the `extension` option. This process will attempt to convert from one file type to another, for example, converting a JPG to a PNG file.

```twig
<img src="{{ 'banner.jpg'|resize(800, 600, {
    filename: 'my-seo-friendly-name',
    extension: 'png'
}) }}" />
```

## Custom Folder Names

When using custom filenames, the filename will still contain a trailing hash to ensure uniqueness within the resizer resource directory. In most cases this is enough to satisfy basic SEO requirements. For example, the file will appear in the system like so.

- `.../800_600_0_0_auto/my-seo-friendly-name_001a14981ffe90700046616c5f415467.png`

A `group` can be specified as a folder name, this will place the image in a dedicated resource group.

```twig
<img src="{{ 'banner.jpg'|resize(800, 600, {
    filename: 'my-seo-friendly-name',
    group: '2024-banners'
}) }}" />
```

For example, the above will place the file in the following directory:

- `.../800_600_0_0_auto/2024-banners/my-seo-friendly-name.png`

However, keep in mind, this approach can be prone to naming collisions. If a different file is resized using the same name and resize options, it will output the original file since there is no unique hash added to the path.

## Available Sources

You may reference images from multiple sources, including the following paths:

- `/app`
- `/plugins`
- `/themes`
- `/modules`
- `/storage/app/uploads`
- `/storage/app/media`

For example:

```twig
{{ '/plugins/acme/blog/assets/images/someimage.png'|resize(...) }}
```

## PHP Interface

You may resize images in PHP using the `System\Classes\ResizeImages` and `resize` method. The return value is a URL location to the resized image.

```php
ResizeImages::resize('path/to/asset.jpg');
```

The method accepts a width (second argument), height (third argument) and [resizer options](../../extend/services/resizer.md) (fourth argument).

```php
ResizeImages::resize('path/to/asset.jpg', 800, 600, ['mode' => 'crop']);
```

#### See Also

::: also
* [Resizer Service](../../extend/services/resizer.md)
:::
