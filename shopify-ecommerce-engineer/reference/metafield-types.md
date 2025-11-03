---
title: 'Liquid objects: metafield'
description: 'A [metafield](/apps/metafields) attached to a parent object.'
api_name: liquid
source_url:
  html: 'https://shopify.dev/docs/api/liquid/objects/metafield'
  md: 'https://shopify.dev/docs/api/liquid/objects/metafield.md'
---

# metafield

A [metafield](https://shopify.dev/apps/metafields) attached to a parent object.

To learn about how to access a metafield on a specific object, refer to [Access metafields](https://shopify.dev/docs/api/liquid/objects/metafield#metafield-access-metafields).

Metafields support [multiple data types](https://shopify.dev/apps/metafields/types), which determine the kind of information that's stored in the metafield. You can also output the metafield content in a type-specific format using [metafield filters](https://shopify.dev/docs/api/liquid/filters/metafield-filters).

***

Note

You can't create metafields in Liquid. Metafields can be created in only the following ways:

* [In the Shopify admin](https://help.shopify.com/manual/metafields)
* [Through an app](https://shopify.dev/apps/metafields)

***

***

Note

Metafields of type `integer`, `json_string`, and `string` are older implementations that don't have the properties noted on this page, and aren't compatible with metafield filters. To learn more, refer to [Deprecated metafields](https://shopify.dev/docs/api/liquid/objects/metafield#metafield-deprecated-metafields).

***

## Properties

* * list?

    [boolean](https://shopify.dev/docs/api/liquid/basics#boolean)

  * Returns `true` if the metafield is a list type. Returns `false` if not.

    Tip

    To learn about metafield types, refer to [Metafield types](https://shopify.dev/apps/metafields/types).

  * type

    [string](https://shopify.dev/docs/api/liquid/basics#string) from a set of values

  * The [type](https://shopify.dev/apps/metafields/types) of the metafield.

    | Possible values |
    | - |
    | single\_line\_text\_field |
    | multi\_line\_text\_field |
    | rich\_text\_field |
    | product\_reference |
    | collection\_reference |
    | variant\_reference |
    | page\_reference |
    | file\_reference |
    | number\_integer |
    | number\_decimal |
    | date |
    | date\_time |
    | url\_reference |
    | json |
    | boolean |
    | color |
    | weight |
    | volume |
    | dimension |
    | rating |
    | money |

  * value

  * The value of the metafield.

    The following table outlines the value format for each metafield type:

    | Type | Returned format |
    | - | - |
    | `single_line_text_field` `multi_line_text_field` | [A string](https://shopify.dev/docs/api/liquid/basics#string) |
    | `rich_text_field` | A field that supports headings, lists, links, bold, and italics |
    | `product_reference` | [A product object](https://shopify.dev/docs/api/liquid/objects/product) |
    | `collection_reference` | [A collection object](https://shopify.dev/docs/api/liquid/objects/collection) |
    | `variant_reference` | [A variant object](https://shopify.dev/docs/api/liquid/objects/variant) |
    | `page_reference` | [A page object](https://shopify.dev/docs/api/liquid/objects/page) |
    | `file_reference` | [A generic\_file object](https://shopify.dev/docs/api/liquid/objects/generic-file) [A media object (images and videos only)](https://shopify.dev/docs/api/liquid/objects/media) |
    | `number_integer` `number_decimal` | [A number](https://shopify.dev/docs/api/liquid/basics#number) |
    | `date` `date_time` | A date string. To format the string, use the [date](https://shopify.dev/docs/api/liquid/filters/date) filter. |
    | `url_reference` | [A url string](https://shopify.dev/docs/api/liquid/basics#string) |
    | `json` | [A JSON object](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON) |
    | `boolean` | [A boolean](https://shopify.dev/docs/api/liquid/basics#boolean) |
    | `color` | [A color object](https://shopify.dev/docs/api/liquid/objects/color) |
    | `weight` `volume` `dimension` | [A measurement object](https://shopify.dev/docs/api/liquid/objects/measurement) |
    | `rating` | [A rating object](https://shopify.dev/docs/api/liquid/objects/rating) |
    | `money` | [A money object, displayed in the customer's local (presentment) currency.](https://shopify.dev/docs/api/liquid/objects/money) |

```json
{
  "list?": false,
  "type": "single_line_text_field",
  "value": "Take with a meal."
}
```

##### Example

```
{
  "list?": false,
  "type": "single_line_text_field",
  "value": "Take with a meal."
}
```

### Access metafields

The access path for metafields consists of two layers:

* namespace - A grouping of metafields to prevent conflicts.
* key - The metafield name.

Given this, you can access the metafield object with the following syntax:

```liquid
{{ resource.metafields.namespace.key }}
```

```liquid
Type: {{ product.metafields.information.directions.type }}
Value: {{ product.metafields.information.directions.value }}
```

##### Code

```
Type: {{ product.metafields.information.directions.type }}
Value: {{ product.metafields.information.directions.value }}
```

##### Data

```
{
  "product": {
    "metafields": {}
  }
}
```

## Output

```html
Type: single_line_text_field
Value: Take with a meal.
```

### Accessing metafields of type `json`

The `value` property of metafields of type `json` returns a [JSON object](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON). You can access the properties of this object directly in Liquid, either by name or 0-based index. You can also iterate through the properties.

```liquid
Temperature: {{ product.metafields.information.burn_temperature.value.temperature }}
Unit: {{ product.metafields.information.burn_temperature.value['unit'] }}


{% for property in product.metafields.information.burn_temperature.value -%}
  {{ property.first | capitalize }}: {{ property.last }}
{%- endfor %}
```

##### Code

```
Temperature: {{ product.metafields.information.burn_temperature.value.temperature }}
Unit: {{ product.metafields.information.burn_temperature.value['unit'] }}

{% for property in product.metafields.information.burn_temperature.value -%}
  {{ property.first | capitalize }}: {{ property.last }}
{%- endfor %}
```

##### Data

```
{
  "product": {
    "metafields": {}
  }
}
```

## Output

```html
Temperature: 700
Unit: degrees


Temperature: 700
Unit: degrees
Scale: Fahrenheit
```

### Accessing metafields of type `list`

The `value` property of metafields of type `list` returns an array. You can iterate through the array to access the values.

```liquid
{% for item in product.metafields.information.combine_with.value -%}
  {{ item.product.title }}
{%- endfor %}
```

##### Code

```
{% for item in product.metafields.information.combine_with.value -%}
  {{ item.product.title }}
{%- endfor %}
```

##### Data

```
{
  "product": {
    "metafields": {}
  }
}
```

## Output

```html
Blue Mountain Flower
Charcoal
```

If the list is of type `single_line_text_field`, then you can access the items in the array directly in Liquid using a 0-based index.

```liquid
First item in list: {{ product.metafields.information.pickup_locations.value[0] }}
Last item in list: {{ product.metafields.information.pickup_locations.value.last }}
```

##### Code

```
First item in list: {{ product.metafields.information.pickup_locations.value[0] }}
Last item in list: {{ product.metafields.information.pickup_locations.value.last }}
```

##### Data

```
{
  "product": {
    "metafields": {}
  }
}
```

## Output

```html
First item in list: Ottawa
Last item in list: Vancouver
```

### Determining the length of a list metafield

The way that you determine the length of a list metafield depends on its type:

* **[Reference types](https://shopify.dev/docs/apps/custom-data/metafields/types#reference-types)**: Use the `count` property to determine the list length.
* **Non-reference types**: These lists are rendered as arrays. Use the [`size`](https://shopify.dev/docs/api/liquid/filters/size) filter to determine the number of items in the array.

```liquid
# list.product_reference
Number of similar products: {{ product.metafields.information.similar_products.value.count }}


# list.single_line_text_field
Number of pickup locations: {{ product.metafields.information.pickup_locations.value.size }}
```

##### Code

```
# list.product_reference
Number of similar products: {{ product.metafields.information.similar_products.value.count }}

# list.single_line_text_field
Number of pickup locations: {{ product.metafields.information.pickup_locations.value.size }}
```

##### Data

```
{
  "product": {
    "metafields": {}
  }
}
```

## Output

```html
# list.product_reference
Number of similar products: 2


# list.single_line_text_field
Number of pickup locations: 4
```

### Deprecated metafields

Deprecated metafields are older metafield types with limited functionality. The following metafield types are deprecated:

* `integer`
* `json_string`
* `string`

These metafield types don't have the same metafield object properties mentioned in the previous sections. Instead, they return the metafield value directly.

The following table outlines the value type for each deprecated metafield type:

| Metafield type | Value type |
| - | - |
| `integer` | [An integer](https://shopify.dev/docs/api/liquid/basics#number) |
| `json_string` | [A JSON object](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON) |
| `string` | [A string](https://shopify.dev/docs/api/liquid/basics#string) |