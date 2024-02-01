---
subtitle: Design your API and update the page dynamically.
---
# Event Handlers

AJAX event handlers are API endpoints for the AJAX framework to communicate with the server. They can respond with raw data, redirect the browser or dynamically update partials on the page.

## AJAX Handlers

To create an AJAX handler, define it as a PHP function the page, partial or layout PHP section, or [inside CMS components](../themes/components.md). Handler names should use the `onSomething` pattern, for example, `onName`. All handlers support the use of [updating partials](./update-partials.md) as part of the AJAX request.

```php
function onSubmitContactForm()
{
    // ...
}
```

::: tip
If two handlers with the same name are defined in a page and layout together, the page handler will take priority. The handlers defined in components have the lowest priority.
:::

### Calling a Handler

Every AJAX request should specify a handler name, either using the [data attributes API](../ajax/attributes-api.md) or the [JavaScript API](../ajax/javascript-api.md). When the request is made, the server will search all the registered handlers and locate the first one it finds.

```html
<!-- Attributes API -->
<button data-request="onSubmitContactForm">Go</button>

<!-- JavaScript API -->
<script> oc.ajax('onSubmitContactForm') </script>
```

Handlers defined by pages, layouts and components are all registered automatically. If you are calling a handler from inside a partial, use the [`{% ajaxPartial %}` Twig tag](../../markup/tag/ajax-partial.md), which adjusts the page cycle to register its handlers.

### Form Serialization

When an AJAX request occurs inside a HTML form tag, all the input values of the form are available to the handler. In the example below, the `first_name` value will be sent with the request.

```html
<form id="myForm">
    <input name="first_name" />
    <button data-request="onSubmitContactForm">Go</button>
</form>
```

The JavaScript API support this logic with the `oc.request` function.

```html
<script> oc.request('#myForm', 'onSubmitContactForm') </script>
```

You may use the `input()` PHP function to access the variable.

```php
function onSubmitContactForm()
{
    $firstName = input('first_name');
}
```

### Generic Handler

Sometimes you may need to make an AJAX request for the sole purpose of updating page contents, not needing to execute any code. You may use the `onAjax` handler for this purpose. This handler is available everywhere without needing to write any code.

```html
<button data-request="onAjax">Do nothing</button>
```

### Component Handlers

If two components register the same handler name, it is advised to prefix the handler with the [component short name or alias](../../cms/themes/components.md). If a component uses an alias of **mycomponent** the handler can be targeted with `mycomponent::onName`.

```html
<button data-request="mycomponent::onSubmitContactForm">Go</button>
```

See the [Component Development article](../../extend/cms-components.md) to learn more.

## Redirects in AJAX Handlers

If you need to redirect the browser to another location, return the `Redirect` response object from the AJAX handler. The framework will redirect the browser as soon as the response is returned from the server. Example AJAX handler with a redirect.

```php
function onRedirectMe()
{
    return Redirect::to('http://google.com');
}
```

## Returning Data from AJAX Handlers

The response from an AJAX handler can serve as consumable API by returning structured data. If an AJAX handler returns an array, you can access its elements in the `success` event handler. Example AJAX handler that returns a data object.

```php
function onFetchDataFromServer()
{
    // Some server-side code

    return [
        'totalUsers' => 1000,
        'totalProjects' => 937
    ];
}
```

The data can be fetched with the data attributes API.

```html
<form data-request="onHandleForm" data-request-success="console.log(data)">
```

The same with the JavaScript API.

```html
<form onsubmit="oc.request(this, 'onHandleForm', {
        success: function(data) {
            console.log(data);
        }
    }); return false"
>
```

## Running Code Before Handlers

Sometimes you may want code to execute before a handler executes. Defining an `onInit` function as part of the [Layout Execution Life Cycle](../../cms/themes/layouts.md) allows code to run before every AJAX handler.

```php
function onInit()
{
    // From a page or layout PHP code section
}
```

You may define an `init` method inside a [CMS component class](../../extend/cms-components.md).

```php
function init()
{
    // From a component or widget class
}
```

## Throwing an AJAX Exception

You may throw an [AJAX exception](../../extend/system/exceptions.md) using the `AjaxException` class to treat the response as an error while retaining the ability to send response contents as normal. Simply pass the response contents as the first argument of the exception.

```php
throw new AjaxException([
    'error' => 'Not enough questions',
    'questionsNeeded' => 2
]);
```

These errors are handled by the AJAX framework.

```html
<form data-request="onHandleForm" data-request-error="console.log(data)">
```

The same with the JavaScript API.

```html
<form onsubmit="oc.request(this, 'onHandleForm', {
        error: function(data) {
            console.log(data);
        }
    }); return false"
>
```

::: tip
When throwing this exception type [partials will be updated](./update-partials.md) as normal.
:::

## Dispatching Browser Events

::: aside
Dispatched events are triggered in an AJAX response after the request completes and before partials are updated.
:::

You may dispatch JavaScript events from AJAX handlers using the `dispatchBrowserEvent` method. This method takes any event name (first argument) and detail variables to pass to the event (second argument), the variables must be compatible with JSON serialization.

```php
function onPerformAction()
{
    $this->dispatchBrowserEvent('app:update-profile');

    $this->dispatchBrowserEvent('app:update-profile', ['name' => 'Jeff']);
}
```

In the browser, use the `addEventListener` to listen for the dispatched event when the AJAX request completes. The event variables are available via the `event.detail` object.

```js
addEventListener('app:update-profile', function (event) {
    alert('Profile updated with name: ' + event.detail.name);
});
```

For example, if you want to show an alert that a document has already been updated by another user, you could dispatch an event to the browser and throw an `AjaxException` to halt the process.

::: tip
`AjaxException` and `ValidationException` are halting exceptions that support dispatched events.
```php
public function onUpdate()
{
    $this->dispatchBrowserEvent('app:stale-document');

    throw new AjaxException;
}
```
:::

You can listen to this event in the browser using a generic listener. This example prompts the user before resubmitting the request with a `force` flag set in the data.

```js
addEventListener('app:stale-document', function (event) {
    if (confirm('Another user has updated this document, proceed?')) {
        oc.request(event.target, 'onUpdate', { data: {
            force: true
        }});
    }
});
```

To prevent the partials from updating as part of the response, call `preventDefault()` on the event object.

```js
addEventListener('app:stale-document', function (event) {
    event.preventDefault();
});
```
