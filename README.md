
# Designers Guide

You're a designer and want to publish and sell your design on branchbob's theme store?

* Get a free sandbox store: [Signup here](https://www.branchbob.com/accounts/register/en/designer/)
* Build your design
* Publish on the branchbob theme store
* Earn money

## Contents of this guide (work in progress)


<!-- toc -->

* [Contents of this guide (work in progress)](#contents-of-this-guide-work-in-progress)
* [Required files](#required-files)
* [Design configuration](#design-configuration)
* [Template reference](#template-reference)
  * [Liquid engine](#liquid-engine)
    * [General store attributes](#general-store-attributes)
    * [Objects and collections on the store](#objects-and-collections-on-the-store)
    * [Tags](#tags)
* [wundery.js](#wunderyjs)
  * [Basic integration](#basic-integration)
  * [Advanced Integration](#advanced-integration)
    * [Example: Customize the cart design and show a popup when an item was added to the cart.](#example-customize-the-cart-design-and-show-a-popup-when-an-item-was-added-to-the-cart)
    * [Example: Manually add a product to the cart and directly redirect to the checkout page](#example-manually-add-a-product-to-the-cart-and-directly-redirect-to-the-checkout-page)
  * [Page markup](#page-markup)

<!-- toc stop -->





## Required files

Only one file is required: the design master. You may choose an arbitray name, e.g. `master.html` or `index.html` but make sure to set it as master file in the branchbob designer.

## Design configuration

To make your design easily customizable for the store owner, you may create a so called manifest file in the JSON format. Name it as you like, e.g. `design.json`, but make sure to set it as manifest file within the designer. The following example shows the available options:

```json

{
  "name": "Theme Name",
  "engine": "liquid",
  "master": "master.html",
  "sections": [
    {
      "name": "general_settings",
      "title": "General Settings",
      "display": true,
      "options": [
        {
          "name": "eg. font",
          "type": "text",
          "default_value": "Verdana",
          "title": "Primary Font / Google Font..."
        },
        {
          "name": "color_1",
          "type": "color",
          "default_value": "#eb8f73",
          "title": "Color 1"
        },
        
    {
      "name": "categories_enable",
      "title": "Show Categories",
      "display": true,
      "options": [
      ]
    }
      ]
    }
  ]
}

```

**IMPORTANT** - Your manifest file must contain valid JSON. If it does not, no option at all will be shown to the store owner. We recommend using a validator, e.g. http://jsonlint.com.

The top-level structure of the manifest file is defined as follows:

| Key | Description          |
| :------------- | :----------- |
| options | Array of available customization options |

Every option has the following structure:

| Key | Description          |
| :------------- | :----------- |
| name | A unique name for the option. It's used for fetching the options value in the designs source, e.g. `{% setting 'background_color' %}` |
| type | The options type. Supported: **color** (shows a color picker) |
| default_value | The options default value. This will be used if the user did not define another value. |
| title | A short explanation of the option that will be displayed in the user interface. If it is defined as a string, it is assumed to be the english title. Otherwise it is assumed to be a hash containing locales as keys and translated titles as values. The following values are both valid: `"title": "..."` or `"title": { "de": "...", "en": "...", ... }` |
| description | Same as title but used for a longer explanation |

## Template reference

### Liquid engine

You can use the following tags:

#### General store attributes

* `{{ store.id }}`
* `{{ store.title }}`
* `{{ store.description }}`
* `{{ store.description_sanitized }}`
* `{{ store.slogan }}`
* `{{ store.facebook_url }}`
* `{{ store.pinterest_url }}`
* `{{ store.twitter_url }}`
* `{{ store.google_analytics_id }}`
* `{{ store.display_net }}`
* `{{ store.url }}`
* `{{ store.description }}`
* `{{ store.locale }}`

#### Objects and collections on the store

* `{{ store.homepage_category }}` - Returns the stores homepage category.
* `{{ store.homepage_products }}` - Shortcut to the stores homepage category products.
* `{{ store.categories }}`
* `{{ store.pages }}`
* `{{ store.payment_methods }}`
* `{{ store.shipping_methods }}`
* `{{ store.logo }}`

#### Tags

* `{{ setting "background_color", "#ffffff" }}` - Returns the value of a setting `background_color` of the current design. You should also set a default value in case the setting is not defined or available.

## wundery.js

IMPORTANT: This documentation is work in progress. It should give you an idea about what's possible with `wundery.js`. Contact us with feedback or questions via support@branchbob.com.

`wundery.js` is a Javascript library providing several methods to communicate with the branchbob API. One of it's main purposes is the handling of the little cart boxes. Because of the branchbob architecture where all store pages are delivered from a high performance cache, the cart has to be injected dynamically.

### Basic integration

**1) Make wundery.js available in your page**

Load the `wundery.js` source in the head section of your page:

```html
<script type="text/javascript" src="https://js.wundery.com/v2/wundery.js"></script>
```

**2) Initialize the cart object**

Put the following script directly below the previous script tag within the head section.

```html
<script type="text/javascript">
var cart = new Wundery.Cart({
  storeId: "{{ store.id }}"
});
cart.setup();
</script>
```

That's it. This will inject the little cart box in your store and handle all cart interactions.

### Advanced Integration

The following script shows several possible methods you may use. Also check the specific use cases below.

```html
<script type="text/javascript">
var cart = new Wundery.Cart({
  storeId: "{{ store.id }}",

  // This will trigger debug messages to the browser console.
  // Defaults to false, optional.
  debug: true
});

// Inject only the box, do nothing else
cart.inject();

// Request a checkout form the branchbob API. If none is present
// in the current session, it will create a new one. If a checkout
// was created before, it will be fetched.
// If a checkout is present, but finished (checkout completed by
// the customer), a new one will be created.
cart
  .getCheckout()
  .then(function (checkout) {
    console.log("Checkout returned from branchbob API", checkout);
  });

// Discover all interaction elements, e.g. "Add to cart" buttons etc.
// IMPORTANT: You should do this once the DOM is ready, not earlier.
cart.discover({
  // shows all discovered elements.
  // Defaults to true, optional.
  visualize: true
});

// Add an item to the current checkout.
cart.add({
  variant_id: "...",
  quantity: 2
});

// refreshes the cart box.
cart.refresh();

// Callbacks

// After an item was added to the cart
cart.added(function (checkout_item) {
  console.log("Item was added to the cart", checkout_item);
});

// After an item was added to the cart
cart.addFailed(function () {
  console.log("Failed to add item, API not available or server error");
});

// Register callbacks that are executed after injection
cart.injected(function () {
  console.log("Cart was injected");
});

</script>
```

#### Example: Customize the cart design and show a popup when an item was added to the cart.

**1) Insert your custom cart markup within your page**

You may apply whatever styling (CSS) you like. Just make sure that you give a class `.wundery-cart` to the outer container.

```html
<div class="wundery-cart">
  <!-- The link to the cart will be injected automatically -->
  <a class="wundery-cart-link">
    My Custom Cart
    (<span class="wundery-cart-numitems"></span> Items)
    Total: <span class="wundery-cart-total"></span>
  </a>
</div>
```
**2) Make wundery.js available in your page**

Load the `wundery.js` source in the head section of your page:

```html
<script type="text/javascript" src="https://js.wundery.com/v2/wundery.js"></script>
```

**3) Initialize the cart object**

Put the following script directly below the previous script tag within the head section.

```html
<script type="text/javascript">
var cart = new Wundery.Cart({
  storeId: "{{ store.id }}"
});

// After an item was added to the cart
cart.added(function (checkout_item) {
  // your popup logic here (e.g. open a modal)
});

jQuery(document).ready(function () {
  // Discover all interaction elements, e.g. "Add to cart" buttons etc.
  cart.discover({
    // shows all discovered elements.
    // Defaults to true, optional.
    visualize: true
  });

  // Refresh the cart box.
  // This will fetch a new or existing checkout and
  // insert total, item count and link into the cart box.
  cart.refresh();
});

</script>
```

#### Example: Manually add a product to the cart and directly redirect to the checkout page

This could be interesting for you if you want to design a one product page.

**1) Insert your button markup into your page**

```html
...
<button id="add" type="button">
BUY
</button>
...
```

**2) Make wundery.js available in your page**

Load the `wundery.js` source in the head section of your page:

```html
<script type="text/javascript" src="https://js.wundery.com/v2/wundery.js"></script>
```

**3) Initialize the cart object**

Put the following script directly below the previous script tag within the head section on the product page only.

```html
<script type="text/javascript">
var cart = new Wundery.Cart({
  storeId: "{{ store.id }}"
});

jQuery(document).ready(function () {
  var button = jQuery("#add");
  button.click(function () {
    cart
      .add({
        variant_id: "{{ current_product.default_variant.id }}"
      })
      .then(function (checkout_item) {
        // redirect to the checkout page
        window.location.href = checkout_item.checkout.cart_url;
      });
  });
});
</script>
```

### Page markup

Within your page you can declare products so that they can be discovered automatically by  `wundery.js`. This completely takes away the logical part of adding products to the cart from you.

```html
<div
  data-wundery-product="{{ current_product.id }}"
  data-wundery-variants="{{ current_product.serialized_variants }}">

  ...

  {% for option in current_product.options %}
    <select data-wundery-option="{{ option.id }}">
      {% for characteristic in option.characteristics %}
        <option value="{{ characteristic.id }}">{{ characteristic.title }}</option>
      {% endfor %}
    </select>
  {% endfor %}

  ...

  <div data-wundery-variant-value="inherited_price_gross_formatted"></div>
  <div data-wundery-variant-value="inherited_price_gross_net"></div>
  ...
  <!--
    You may use everything as a value which is available within
    {{ product.serialized_variants }}
  -->

  <div data-wundery-noselection>
    Please select your options
  </div>

  <div data-wundery-notavailable>
    Your selection is not available
  </div>

  <button type="button" data-wundery-add>
    <span data-wundery-not-adding>Add to cart</span>
    <span data-wundery-adding>Adding ...</span>
  </button>
</div>
```
