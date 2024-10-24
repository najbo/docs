---
subtitle: Adds list management features to any backend page.
---
# List Controller

The `Backend\Behaviors\ListController` class is a controller behavior used for easily adding a record list to a page. The behavior provides the sortable and searchable list with optional links on its records. The behavior provides the controller action `index` however the list can be rendered anywhere and multiple list definitions can be used.

List behavior depends on list [column definitions](../../element/list-columns.md) and a [model class](../database/model.md). In order to use the list behavior you should add it to the `$implement` property of the controller class. Also, the `$listConfig` class property should be defined and its value should refer to the YAML file used for configuring the behavior properties.

```php
namespace Acme\Blog\Controllers;

class Categories extends \Backend\Classes\Controller
{
    public $implement = [
        \Backend\Behaviors\ListController::class
    ];

    public $listConfig = 'config_list.yaml';
}
```

::: tip
Very often the list and [form controller](../forms/form-controller.md) are used together in a same controller.
:::

## Configuring the List Behavior

The configuration file referred in the `$listConfig` property is defined in YAML format. The file should be placed into the controller's [views directory](../system/controllers.md). Below is an example of a typical list behavior configuration file.

```yaml
# config_list.yaml
title: Blog Posts
list: ~/plugins/acme/blog/models/post/columns.yaml
modelClass: Acme\Blog\Models\Post
recordUrl: acme/blog/posts/update/:id
```

The following properties are required in the list configuration file.

Property | Description
------------- | -------------
**title** | a title for this list.
**list** | a configuration array or reference to a list column definition file, see [list columns](../../element/list-columns.md).
**modelClass** | a model class name, the list data is loaded from this model.

The configuration properties listed below are optional.

Property | Description
------------- | -------------
**filter** | filter configuration, see [list filters](./filters.md).
**recordUrl** | link each list record to another page. Eg: **users/update:id**. The `:id` part is replaced with the record identifier. This allows you to link the list behavior and the [form behavior](../forms/form-controller.md).
**recordOnClick** | custom JavaScript code to execute when clicking on a record.
**noRecordsMessage** | a message to display when no records are found, can refer to a [localization string](../system/localization.md).
**deleteMessage** | a message to display when records are bulk deleted, can refer to a [localization string](../system/localization.md).
**noRecordsDeletedMessage** | a message to display when a bulk delete action is triggered, but no records were deleted, can refer to a [localization string](../system/localization.md).
**recordsPerPage** | records to display per page, use 0 for no pages. Default: `0`
**perPageOptions** | options for number of items per page. Default: `[20, 40, 80, 100, 120]`
**showPageNumbers** | displays page numbers with pagination. Disable this to improve list performance when working with large tables. Default: `true`
**toolbar** | reference to a Toolbar Widget configuration file, or an array with configuration (see below).
**showSorting** | displays the sorting link on each column. Default: `true`
**defaultSort** | sets a default sorting column and direction when user preference is not defined. Supports a string or an array with keys `column` and `direction`. The direction can be `asc` for ascending (default) or `desc` for descending order.
**showCheckboxes** | displays checkboxes next to each record. Default: `false`.
**showSetup** | displays the list column set up button. Default: `false`.
**structure** | enables a structured list, see the [sorting records article](./structures.md) for more details.
**customViewPath** | specify a custom view path to override partials used by the list, optional.
**customPageName** | specify a custom variable name to use in the page URL for paginated records. Set to `false` to disable storing the page number in the URL. Default: `page`.

### Adding a Toolbar

To include a toolbar with the list, add the following configuration to the list configuration YAML file:

```yaml
toolbar:
    buttons: list_toolbar
    search:
        prompt: Find records
```

The toolbar configuration allows:

Property | Description
------------- | -------------
**buttons** | a reference to a controller partial file with the toolbar buttons. Eg: **_list_toolbar.htm**
**search** | reference to a Search Widget configuration file, or an array with configuration.

The search configuration supports the following properties:

Property | Description
------------- | -------------
**prompt** | a placeholder to display when there is no active search, can refer to a [localization string](../system/localization.md).
**mode** | defines the search strategy to either contain all words, any word or exact phrase. Supported options: `all`, `any`, `exact`. Default: `all`.
**scope** | specifies a [model query scope](../database/model.md) method defined in the **list model** to apply to the search query. The first argument will contain the query object (as per a regular scope method), the second will contain the search term, and the third will be an array of the columns to be searched.
**searchOnEnter** | setting this to true will make the search widget wait for the Enter key to be pressed before it starts searching (the default behavior is that it starts searching automatically after someone enters something into the search field and then pauses for a short moment).  Default: `false`.

The toolbar buttons partial referred above should contain the toolbar control definition with some buttons. The partial could also contain a [scoreboard control](https://octobercms.com/docs/ui/scoreboard) with charts. Example of a toolbar partial with the **New Post** button referring to the **create** action provided by the [form behavior](forms.md):

```php
<div data-control="toolbar">
    <a href="<?= Backend::url('acme/blog/posts/create') ?>"
        class="btn btn-primary oc-icon-plus">
        New Post
    </a>
</div>
```

When using list checkboxes, you may toggle a button's enabled state with the `data-list-checked-trigger` attribute.

```php
<button
    type="button"
    class="btn btn-primary"
    data-list-checked-trigger>
    Delete Selected
</button>
```

You may also pass the checked values to an AJAX request using the `data-list-checked-request` attribute.

```php
<button
    type="button"
    class="btn btn-primary"
    data-request="onDelete"
    data-list-checked-request>
    Delete Selected
</button>
```

### Filtering the List

To filter a list by user defined input, add the following list configuration to the YAML file:

```yaml
filter: $/acme/blog/models/post/scopes.yaml
```

The **filter** property should make reference to a [filter configuration file](./filters.md) path or supply an array with the configuration.

## Defining List Columns

::: aside
The available list column properties can be found on the [list column definitions](../../element/list-columns.md) page.
:::

List columns are defined with the YAML file. The column configuration is used by the list behavior for creating the record table and displaying model columns in the table cells. The file is placed to a subdirectory of the **models** directory of a plugin. The subdirectory name matches the model class name written in lowercase. The file name doesn't matter, but the **columns.yaml** and **list_columns.yaml** are common names. Example list columns file location:

::: dir
├── plugins
|   └── acme
|       └── blog
|           └── `models`
|               ├── post  _← Config Directory_
|               |   └── columns.yaml  _← Config File_
|               └── Post.php  _← Model Class_
:::

The next example shows the typical contents of a list column definitions file.

```yaml
# columns.yaml
columns:
    name: Name
    email: Email
```

## Displaying the List

Usually lists are displayed in the [index view](../system/views.md) file. Since lists include the toolbar, the view file will consist solely of the single `listRender` method call.

```php
<?= $this->listRender() ?>
```

## Multiple List Definitions

The list behavior can support multiple lists in the same controller using named definitions. The `$listConfig` property can be defined as an array where the key is a definition name and the value is the configuration file.

```php
public $listConfig = [
    'templates' => 'config_templates_list.yaml',
    'layouts' => 'config_layouts_list.yaml'
];
```

Each definition can then be displayed by passing the definition name as the first argument when calling the `listRender` method.

```php
<?= $this->listRender('templates') ?>
```

## Extending List Behavior

Sometimes you may wish to modify the default list behavior and there are several ways you can do this.

### Extending the List Configuration

You may extend the list configuration dynamically using the `listGetConfig` method.

```php
public function listGetConfig($definition)
{
    $config = $this->asExtension('ListController')->listGetConfig($definition);

    // Implement structure dynamically
    $config->structure = [
        'showTree' => true
    ];

    return $config;
}
```

### Overriding Controller Action

You may use your own logic for the `index` action method in the controller, then optionally call the List behavior `index` parent method.

```php
public function index()
{
    //
    // Do any custom code here
    //

    // Call the ListController behavior index() method
    $this->asExtension('ListController')->index();
}
```

### Overriding Views

The `ListController` behavior has a main container view that you may override by creating a special file named `_list_container.php` in your controller directory. The following example will add a sidebar to the list:

```php
<?php if ($toolbar): ?>
    <?= $toolbar->render() ?>
<?php endif ?>

<?php if ($filter): ?>
    <?= $filter->render() ?>
<?php endif ?>

<div class="row row-flush">
    <div class="col-sm-3">
        [Insert sidebar here]
    </div>
    <div class="col-sm-9 list-with-sidebar">
        <?= $list->render() ?>
    </div>
</div>
```

The behavior will invoke a `Lists` widget that also contains numerous views that you may override. This is possible by specifying a `customViewPath` property as described in the list configuration options. The widget will look in this path for a view first, then fall back to the default location.

```yaml
# Custom view path
customViewPath: $/acme/blog/controllers/reviews/list
```

::: tip
It is a good idea to use a sub-directory, for example called `list`, to avoid conflicts.
:::

For example, to modify the list body row markup, create a file called `list/_list_body_row.php` in your controller directory.

```php
<tr>
    <?php foreach ($columns as $key => $column): ?>
        <td><?= $this->getColumnValue($record, $column) ?></td>
    <?php endforeach ?>
</tr>
```

### Extending Column Definitions

You may extend the columns of another controller from outside by binding to the `backend.list.extendColumns` [global event](../services/event.md). The event function will take a `$list` argument that represents the `Backend\Widgets\Lists` object, where you can use the `getController` and `getModel` methods to check the execution context.

Since this event has the potential to affect all lists, it is essential to check that the controller and model is of the correct type. Here is an example using the `addColumns` method to add new columns to the event log list, and modify an existing column.

```php
Event::listen('backend.list.extendColumns', function($list) {
    if (
        !$list->getController() instanceof \System\Controllers\EventLogs ||
        !$list->getModel() instanceof \System\Models\EventLog
    ) {
        return;
    }

    // Add a new column
    $list->addColumns([
        'my_column' => [
            'label' => 'My Column'
        ]
    ]);

    // Modify an existing column
    $list->getColumn('title')->useConfig([
        'path' => 'column_title'
    ]);
});
```

You may also extend the list columns internally by overriding the `listExtendColumns` method inside the controller class. This will only affect the list used by the `ListController` behavior.

```php
class Categories extends \Backend\Classes\Controller
{
    public $implement = [
        \Backend\Behaviors\ListController::class
    ];

    public function listExtendColumns($list)
    {
        $list->addColumns([...]);

        $list->getColumn(...);
    }
}
```

The following methods are available on the `$list` object.

Method | Description
------------- | -------------
**addColumns** | adds new columns to the list
**removeColumn** | removes a column from the list
**getColumn** | returns an existing column definition

Each method takes an array of columns similar to the [list column configuration](../../element/list-columns.md).

### Inject CSS Row Class

You may inject a custom css row class by adding a `listInjectRowClass` method on the controller class. This method can take two arguments, **$record** will represent a single model record and **$definition** contains the name of the List widget definition. You may return any string value containing your row classes. These classes will be added to the row's HTML markup.

```php
class Lessons extends \Backend\Classes\Controller
{
    // ...

    public function listInjectRowClass($lesson, $definition = null)
    {
        // Strike through past lessons
        if ($lesson->lesson_date->lt(Carbon::today())) {
            return 'strike';
        }
    }
}
```

A special CSS class `nolink` is available to force a row to be unclickable, even if the `recordUrl` or `recordOnClick` properties are defined for the list widget. Returning this class in an event will allow you to make records unclickable - for example, for soft-deleted rows or for informational rows:

```php
public function listInjectRowClass($record, $value)
{
    if ($record->trashed()) {
        return 'nolink';
    }
}
```

### Overriding Column URL

You may specify the click action for a column record by overriding the `listOverrideRecordUrl` method. This method can return a string for a new backend URL or an array with a complex definition.

```php
public function listOverrideRecordUrl($record, $definition = null)
{
    if ($record->is_active) {
        return 'acme/test/services/preview/' . $record->id;
    }
}
```

To override the onclick behavior return an array with the `onclick` key and change the `url` to null.

```php
public function listOverrideRecordUrl($record, $definition = null)
{
    if ($record->is_banned) {
        return ['onclick' => "alert('Unable to click')", 'url' => null];
    }
}
```

To make a column unclickable entirely return an array with the `clickable` key set to false.

```php
public function listOverrideRecordUrl($record, $definition = null)
{
    if ($record->is_disabled) {
        return ['clickable' => false];
    }
}
```

### Extending Filter Scopes

You may extend the filter scopes of another controller by binding to the `backend.filter.extendScopes` [global event](../services/event.md). This method can take the argument `$filter` which will represent the `Backend\Widgets\Filter` object, where you can use the `getController`, `getModel` and `getContext` methods to check the execution context.

Since this event has the potential to affect all filters, it is essential to check that the controller and model is of the correct type. Here is an example using the `addScopes` method to add new fields to the event log list, and adjust the CSS classes.

```php
Event::listen('backend.filter.extendScopes', function($filter) {
    if (
        !$filter->getController() instanceof \System\Controllers\EventLogs ||
        !$filter->getModel() instanceof \System\Models\EventLog
    ) {
        return;
    }

    // Add a new scope
    $filter->addScopes([
        'my_scope' => [
            'label' => 'My Filter Scope'
        ]
    ]);

    // Add custom CSS classes to the filter widget
    $filter->cssClasses = array_merge(
        $filter->cssClasses,
        ['my-array', 'of-classes']
    );
});
```

You may also extend the filter scopes internally to the controller class, simply override the `listFilterExtendScopes` method.

```php
class Categories extends \Backend\Classes\Controller
{
    public $implement = [
        \Backend\Behaviors\ListController::class
    ];

    public function listFilterExtendScopes($filter)
    {
        $filter->addScopes([...]);
    }
}
```

The following methods are available on the `$filter` object. The scopes available are the same as the [list filters configuration](./filters.md).

Method | Description
------------- | -------------
**addScopes** | adds new scopes to filter widget using [list filters configuration](./filters.md)
**removeScope** | remove scope from filter widget
**getScope** | returns an existing scope definition

#### Extending the Filter Response

The `listExtendRefreshResults` method can interact with the AJAX update response when the list updates, and should return an array of additional partial updates. The `listGetFilterWidget` will return the filter widget for access to the scopes.

```php
public function listExtendRefreshResults($filter, $result)
{
    $statusCode = $this->listGetFilterWidget()->getScope('status_code')->value;

    return ['#my-partial-id' => $this->makePartial(...)];
}
```

### Extending the Model Query

The lookup query for the list [database model](../database/model.md) can be extended by overriding the `listExtendQuery` method inside the controller class. This example will ensure that soft deleted records are included in the list data, by applying the **withTrashed** scope to the query:

```php
public function listExtendQuery($query)
{
    $query->withTrashed();
}
```

When dealing with multiple lists definitions in a same controller, you can use the second parameter of `listExtendQuery` which contains the name of the definition :

```php
public $listConfig = [
    'inbox' => 'config_inbox_list.yaml',
    'trashed' => 'config_trashed_list.yaml'
];

public function listExtendQuery($query, $definition)
{
    if ($definition === 'trashed') {
        $query->onlyTrashed();
    }
}
```

You may also join other tables to aid with searching and sorting. The following will join the `post_statuses` table and introduce the `status_sort_order` and `status_name` columns to the query.

```php
public function listExtendQuery($query, $definition = null)
{
    $query->leftJoin('post_statuses', 'posts.status_id', 'post_statuses.id');

    $query->addSelect(
        'post_statuses.sort_order as status_sort_order',
        'post_statuses.name as status_name'
    );
}
```

The [list filter](./filters.md) model query can also be extended by overriding the `listFilterExtendQuery` method.

```php
public function listFilterExtendQuery($query, $scope)
{
    if ($scope->scopeName == 'status') {
        $query->where('status', '<>', 'all');
    }
}
```

### Extending the Records Collection

The collection of records used by the list can be extended by overriding the `listExtendRecords` method inside the controller class. This example uses the `sort` method on the [record collection](../database/collection.md) to change the sort order of the records.

```php
public function listExtendRecords($records)
{
    return $records->sort(function ($a, $b) {
        return $a->computedVal() > $b->computedVal();
    });
}
```

### Custom Column Types

Custom list column types can be registered in the back-end with the `registerListColumnTypes` method of the [plugin registration file](../extending.md). The method should return an array where the key is the type name and the value is a callable function. The callable function receives three arguments, the native `$value`, the `$column` definition object and the model `$record` object.

```php
public function registerListColumnTypes()
{
    return [
        // A local method, i.e $this->evalUppercaseListColumn()
        'uppercase' => [$this, 'evalUppercaseListColumn'],

        // Using an inline closure
        'loveit' => function($value) { return 'I love '. $value; }
    ];
}

public function evalUppercaseListColumn($value, $column, $record)
{
    return strtoupper($value);
}
```

Using the custom list column type is as simple as calling it by name using the `type` property.

```yaml
# columns.yaml
columns:
    secret_code:
        label: Secret code
        type: uppercase
```
