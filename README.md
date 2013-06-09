# jquery-selective

<a href="http://apostrophenow.org/"><img src="https://raw.github.com/punkave/jquery-selective/master/logos/logo-box-madefor.png" align="right" /></a>

`jquery-selective` provides multiple selection with autocomplete: a list to which items may be added by typing part of the label. A server-side or client-side source may be specified for autocomplete (see jQuery UI autocomplete). Items may be reordered if the sortable option is present (this feature requires jQuery UI sortable). Items may also be removed.

This plugin is especially useful when there are too many options to be presented in an ordinary dropdown list. This plugin is basically a mashup of jQuery autocomplete and jQuery sortable.

<img src="https://raw.github.com/punkave/jquery-selective/master/test/screenshot1.png" align="center" />

## Requirements

You need jQuery, of course. `jquery-selective` is actively supported with jQuery 1.9 and 2.0 but should work fine with reasonably recent versions. `jquery-selective` is supported in IE7 or better and in recent versions of Firefox, Chrome, Opera and Safari. (We cannot officially support IE6, because Microsoft doesn't, but that probably works too.)

You also need jquery.ui.autocomplete. jquery.ui.sortable is optional. See [jqueryui.com](http://jqueryui.com/) for more information on these widely used plugins, typically downloaded as part of a single build.

## How to Use

### Your Markup

The element on which you call $.selective must have the following
structure (the div is the element itself):

    <div class="my-list">
      <!-- Text entry for autocompleting the next item -->
      <input data-autocomplete />
      <!-- The list of existing items added so far -->
      <ul data-list>
        <li data-item>
          <span data-label>Example label</span>
          <a href="#" data-remove>x</a>
        </li>
      </ul>
    </div>

Your markup can be different as long as nested elements with the data
attributes shown are present.

**The element with the data-item attribute
will be duplicated and given the label and value of each actual selection in the list. The original template element will be removed**, so it does not matter what the example label is.

We suggest hiding the entire element until after you call
`$('.my-element').selective({...})` to populate it with the
real data. Of course, if you are building a dialog before displaying it, you won't need to worry about that.

### Invoking selective

Just write:

    $('.my-list').selective({ source: 'http://mysite.com/autocomplete' });

The `source` URL will receive a GET request with a `term` query string parameter containing what the user has typed so far. This URL should respond with a JSON-encoded array in which each object has `value` and `label` properties. The `label` is what is shown in the menu of possible completions, while the `value` is what is actually returned by `get`, described below. The `value` property must be a string or number.

**source is passed to jquery.ui.autocomplete**, so it is possible to use any source property that is acceptable to that function: an array of possible choices, a function, or a URL.

### Passing In Existing Selections

Often the user has already made several choices previously and you need to redisplay these existing selections. **Pass in any existing choices as the `data` property of `options`.**

#### Passing `data` Complete With Labels

`data` may be an array in which each element has `label` and `value` properties, as described above. Here is an example:

    $('.my-list').selective({
      source: 'http://mysite.com/autocomplete',
      data: [ { label: 'One', value: 1 }, { label: 'Two', value: 2 } ]
    });

#### Passing `data` With Values Only

Often you only have the values (typically database IDs) for previous selections. You could fetch the labels yourself, but this is a pain, so we've provided a convenient alternative.

If you pass an array containing only the values, like this:

    $('.my-list').selective({
      source: 'http://mysite.com/autocomplete',
      data: [ 1, 2 ]
    });

Then `jquery-selective` will make a POST request to `source` with a `values` parameter. The `values` parameter will be an array containing what was passed as the `data` option. This is an extension to the standard behavior of autocomplete's `source` option.

A POST request is used because if there are many selections one can easily exceed the maximum size of a GET request.

The use of a function as `source` is also supported. In this case the `request` object passed to `source` will have a single `values` property containing what was passed as the `data` option.

For more information about the normal behavior of the `source` option, see the jQuery UI autocomplete documentation.

### Preventing Duplicates

If you set the `preventDuplicates` option to `true`, `jquery-selective` will automatically prevent the user from seeing suggestions that duplicate choices already made. This check is made on the `value` property of each suggestion, not the `label` property. Duplicate labels may occur, depending on your data (for instance, users with the same name).

### Reordering Selections With Drag and Drop

If you set the `sortable` option to `true` and jQuery UI sortable is available, `jquery-selective` will allow the user to drag and drop the choices they have made into a different order, and the `get` command will respect that order. If you set the `sortable` option to an object, that object is used to configure jQuery UI sortable.

### Retrieving the Result

Call `$('.my-element').selective('get')` to retrieve an array of the current values. This will be a flat array containing the `value` properties of each selection, for example:

    [ 1, 2 ]

Most often the `value` properties are database identifiers. `jquery-selective` is great for managing one-to-many and many-to-many relationships.

### Removing Choices With Strikethrough

By default, when the user removes one of their choices made so far, it disappears from that list. If you set the `strikethrough` option to `true`, any options removed are ~~struck through~~ instead. The user can click the `remove` link again to change their mind. Deleted options still are not returned by the `get` command, unless the `removed` option is passed to the `get` command as described below.

### Including Removed Choices In Results

When the `removed` option is passed to the `get` command, the result consists of an array like this:

    $('.my-element').selective('get', { removed: true });

    [
      { value: 5 },
      { value: 10, removed: true },
      { value: 20 }
    ]

This option is only available if the `strikethrough` option is `true` when configuring `jquery-selective`.

### Propagating Changes to Children

<img src="https://raw.github.com/punkave/jquery-selective/master/test/screenshot2.png" align="center" />

Users making changes to one object, such as a webpage, may sometimes wish to propagate those changes to other objects, such as "child pages" of that webpage. Permissions are a good example: if I am adding or removing permission for the "staff" group to edit a page, I may wish to indicate that this should also apply to all children of that page.

To support this common pattern, `jquery-selective` offers the `propagate` option. If the `propagate` option is true, `jquery-selective` will expect to find a checkbox with the `data-propagate` attribute in the `data-item` template element. The user may check this box to indicate that they wish to propagate that particular choice to other objects. The status of that checkbox will be included in the results as described below. Note that this checkbox appears even for preexisting choices, as a convenient way to propagate that choice at a later time.

** When the `propagate` option is present, the `strikethrough` option is assumed, and the `removed` option to the `get` command is also assumed. ** This allows the user to propagate the removal of a choice as well as the addition of a choice.

** When the `propagate` option is present, a `propagate` property is added to each result returned by `get`. **

If we configure `selective` like this:

    $('.my-element').selective({
      source: 'http://example.com/complete',
      preventDuplicates: true,
      propagate: true
    });

The results of a call to `get` will look like this:

    $('.my-element').selective('get');

    [
      { value: 5, propagate: 1 },
      { value: 10, removed: 1, propagate: 0 },
      { value: 20, propagate: 0 },
      { value: 7, removed: 1, propagate: 1 }
    ]

Ones and zeroes are used as booleans for convenience when POSTing these values over a REST API.

Implementing propagation on the server side is, of course, up to you.

## About P'unk Avenue and Apostrophe

`jquery-selective` was created at [P'unk Avenue](http://punkave.com) for use in Apostrophe, an open-source content management system built on node.js. If you like `jquery-selective` you should definitely [check out apostrophenow.org](http://apostrophenow.org). Also be sure to visit us on [github](http://github.com/punkave).

## Support

Feel free to open issues on [github](http://github.com/punkave/jquery-selective).

<a href="http://punkave.com/"><img src="https://raw.github.com/punkave/jquery-selective/master/logos/logo-box-builtby.png" /></a>

