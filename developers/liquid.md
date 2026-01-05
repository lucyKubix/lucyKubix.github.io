---
layout: default
title: Liquid
description: Not Water
parent: /developers.html
---

## Liquid

Liquid is an open-source templating language created by Shopify. It's the backbone of Shopify themes and is used to load dynamic content on storefronts. This guide covers best practices for writing maintainable, performant, and secure Liquid code in Shopify themes.

### What is Liquid?

Liquid uses a combination of:
- **Objects**: Variables that have attributes (e.g., `product.title`, `collection.handle`)
- **Tags**: Logic that controls flow (e.g., <code>{% raw %}{% if %}{% endraw %}</code>, <code>{% raw %}{% for %}{% endraw %}</code>, <code>{% raw %}{% render %}{% endraw %}</code>)
- **Filters**: Methods that modify output (e.g., <code>{% raw %}{{ price | money }}{% endraw %}</code>, <code>{% raw %}{{ title | upcase }}{% endraw %}</code>)

Liquid code is processed server-side before the HTML is sent to the browser.

### Basic Syntax

#### Objects and Output
{% raw %}
```liquid
{{ product.title }}
{{ collection.description }}
{{ cart.item_count }}
```
{% endraw %}

#### Tags
{% raw %}
```liquid
{% if product.available %}
  <p>In stock</p>
{% endif %}

{% for product in collection.products %}
  <div>{{ product.title }}</div>
{% endfor %}
```
{% endraw %}

#### Filters
{% raw %}
```liquid
{{ product.title | upcase }}
{{ product.price | money }}
{{ collection.handle | handleize }}
```
{% endraw %}

### Performance Best Practices

1. **Limit loops and nested loops**
   {% raw %}
   ```liquid
   {% comment %} ❌ Bad - Nested loops can be slow {% endcomment %}
   {% for collection in collections %}
     {% for product in collection.products %}
       {% for variant in product.variants %}
         {{ variant.title }}
       {% endfor %}
     {% endfor %}
   {% endfor %}

   {% comment %} ✅ Good - Use limit and break when possible {% endcomment %}
   {% for collection in collections limit: 5 %}
     {% for product in collection.products limit: 10 %}
       {% if product.variants.size > 0 %}
         {{ product.variants.first.title }}
       {% endif %}
     {% endfor %}
   {% endfor %}
   ```
   {% endraw %}

2. **Use `forloop.first` and `forloop.last` instead of checking indexes**
   {% raw %}
   ```liquid
   {% comment %} ❌ Less efficient {% endcomment %}
   {% for product in collection.products %}
     {% if forloop.index0 == 0 %}
       <div class="featured">{{ product.title }}</div>
     {% endif %}
   {% endfor %}

   {% comment %} ✅ Better {% endcomment %}
   {% for product in collection.products %}
     {% if forloop.first %}
       <div class="featured">{{ product.title }}</div>
     {% endif %}
   {% endfor %}
   ```
   {% endraw %}

3. **Avoid unnecessary filters**
   {% raw %}
   ```liquid
   {% comment %} ❌ Chaining unnecessary filters {% endcomment %}
   {{ product.title | strip | strip_html | escape | upcase }}

   {% comment %} ✅ Only use what you need {% endcomment %}
   {{ product.title | escape | upcase }}
   ```
   {% endraw %}

4. **Use `assign` to avoid repeated lookups**
   {% raw %}
   ```liquid
   {% comment %} ❌ Bad - Multiple lookups {% endcomment %}
   <h1>{{ product.title }}</h1>
   <p>{{ product.title }} is available!</p>
   <span class="badge">{{ product.title }}</span>

   {% comment %} ✅ Good - Cache the value {% endcomment %}
   {% assign product_title = product.title %}
   <h1>{{ product_title }}</h1>
   <p>{{ product_title }} is available!</p>
   <span class="badge">{{ product_title }}</span>
   ```
   {% endraw %}

5. **Prefer `render` over `include` (deprecated)**
   {% raw %}
   ```liquid
   {% comment %} ❌ Deprecated {% endcomment %}
   {% include 'product-card' %}

   {% comment %} ✅ Recommended {% endcomment %}
   {% render 'product-card', product: product %}
   ```
   {% endraw %}

### Maintainability Best Practices

1. **Use meaningful variable names**
   {% raw %}
   ```liquid
   {% comment %} ❌ Bad {% endcomment %}
   {% assign x = product.price %}
   {% assign y = product.compare_at_price %}

   {% comment %} ✅ Good {% endcomment %}
   {% assign current_price = product.price %}
   {% assign original_price = product.compare_at_price %}
   ```
   {% endraw %}

2. **Add comments for complex logic**
   {% raw %}
   ```liquid
   {% comment %} Calculate discount percentage {% endcomment %}
   {% if product.compare_at_price > product.price %}
     {% assign discount = product.compare_at_price | minus: product.price %}
     {% assign discount_percentage = discount | times: 100 | divided_by: product.compare_at_price %}
   {% endif %}
   ```
   {% endraw %}

3. **Break complex templates into snippets**
   {% raw %}
   ```liquid
   {% comment %} ✅ Good - Reusable and maintainable {% endcomment %}
   {% render 'product-card', product: product %}
   {% render 'product-card', product: related_product %}
   ```
   {% endraw %}

4. **Use consistent formatting**
   {% raw %}
   ```liquid
   {% comment %} ✅ Good - Consistent indentation {% endcomment %}
   {% if product.available %}
     {% for variant in product.variants %}
       <option value="{{ variant.id }}">
         {{ variant.title }}
       </option>
     {% endfor %}
   {% endif %}
   ```
   {% endraw %}

5. **Use `liquid` tag for code blocks**
   {% raw %}
   ```liquid
   {% comment %} ✅ Good - Prevents conflicts with HTML {% endcomment %}
   {% liquid
     assign price = product.price
     assign compare_price = product.compare_at_price
     if compare_price > price
       assign on_sale = true
     endif
   %}
   ```
   {% endraw %}

### Shopify-Specific Best Practices

1. **Use translation filters for all user-facing text**
   {% raw %}
   ```liquid
   {% comment %} ❌ Hard-coded text {% endcomment %}
   <button>Add to Cart</button>

   {% comment %} ✅ Translatable {% endcomment %}
   <button>{{ 'products.product.add_to_cart' | t }}</button>
   ```
   {% endraw %}

2. **Use theme routes for navigation**
   {% raw %}
   ```liquid
   {% comment %} ❌ Hard-coded URLs {% endcomment %}
   <a href="/cart">Cart</a>
   <a href="/account">Account</a>

   {% comment %} ✅ Theme-aware routes {% endcomment %}
   <a href="{{ routes.cart_url }}">{{ 'cart.general.title' | t }}</a>
   <a href="{{ routes.account_url }}">{{ 'customer.account.title' | t }}</a>
   ```
   {% endraw %}

3. **Check for object existence before accessing**
   {% raw %}
   ```liquid
   {% comment %} ❌ May cause errors {% endcomment %}
   {{ collection.products.first.title }}

   {% comment %} ✅ Safe {% endcomment %}
   {% if collection.products.size > 0 %}
     {{ collection.products.first.title }}
   {% endif %}
   ```
   {% endraw %}

4. **Use section settings for customizable content**
   {% raw %}
   ```liquid
   {% comment %} ✅ Good - Configurable via theme editor {% endcomment %}
   {% if section.settings.show_title %}
     <h1>{{ section.settings.title | default: collection.title }}</h1>
   {% endif %}
   ```
   {% endraw %}

5. **Handle empty states gracefully**
   {% raw %}
   ```liquid
   {% if collection.products.size > 0 %}
     {% for product in collection.products %}
       {% render 'product-card', product: product %}
     {% endfor %}
   {% else %}
     <p>{{ 'collections.general.no_matches' | t }}</p>
   {% endif %}
   ```
   {% endraw %}

6. **Use schema for section settings**
   {% raw %}
   ```liquid
   {% schema %}
   {
     "name": "Featured Collection",
     "settings": [
       {
         "type": "collection",
         "id": "collection",
         "label": "Collection"
       },
       {
         "type": "range",
         "id": "product_count",
         "min": 1,
         "max": 12,
         "step": 1,
         "default": 4,
         "label": "Products to show"
       }
     ]
   }
   {% endschema %}
   ```
   {% endraw %}

### Common Patterns

#### Product Card Component
{% raw %}
```liquid
{% comment %} snippets/product-card.liquid {% endcomment %}
<div class="product-card">
  <a href="{{ product.url }}" class="product-card__link">
    {% if product.featured_image %}
      <img
        src="{{ product.featured_image | image_url: width: 300 }}"
        alt="{{ product.featured_image.alt | escape }}"
        loading="lazy"
        width="300"
        height="{{ 300 | divided_by: product.featured_image.aspect_ratio | round }}"
      >
    {% endif %}
    <h3 class="product-card__title">{{ product.title }}</h3>
    <div class="product-card__price">
      {% if product.compare_at_price > product.price %}
        <span class="price price--sale">{{ product.price | money }}</span>
        <span class="price price--compare">{{ product.compare_at_price | money }}</span>
      {% else %}
        <span class="price">{{ product.price | money }}</span>
      {% endif %}
    </div>
  </a>
</div>
```
{% endraw %}

#### Conditional Rendering
{% raw %}
```liquid
{% liquid
  assign has_sale = false
  if product.compare_at_price > product.price
    assign has_sale = true
    assign discount = product.compare_at_price | minus: product.price
    assign discount_percentage = discount | times: 100 | divided_by: product.compare_at_price | round
  endif
%}

{% if has_sale %}
  <span class="badge badge--sale">
    {{ 'products.product.save' | t }} {{ discount_percentage }}%
  </span>
{% endif %}
```
{% endraw %}

#### Navigation Menu
{% raw %}
```liquid
<nav class="nav" role="navigation" aria-label="{{ 'general.accessibility.nav' | t }}">
  <ul class="nav__list">
    {% for link in linklists.main-menu.links %}
      <li class="nav__item">
        <a
          href="{{ link.url }}"
          class="nav__link {% if link.active %}nav__link--active{% endif %}"
        >
          {{ link.title }}
        </a>
        {% if link.links != blank %}
          <ul class="nav__sublist">
            {% for childlink in link.links %}
              <li class="nav__subitem">
                <a href="{{ childlink.url }}" class="nav__sublink">
                  {{ childlink.title }}
                </a>
              </li>
            {% endfor %}
          </ul>
        {% endif %}
      </li>
    {% endfor %}
  </ul>
</nav>
```
{% endraw %}

#### Form Handling
{% raw %}
```liquid
{% form 'contact' %}
  {% if form.posted_successfully? %}
    <p class="form-message form-message--success">
      {{ 'contact.form.post_success' | t }}
    </p>
  {% endif %}

  {% if form.errors %}
    <div class="form-message form-message--error">
      {{ form.errors | default_errors }}
    </div>
  {% endif %}

  <div class="form-group">
    <label for="ContactFormName">{{ 'contact.form.name' | t }}</label>
    <input
      type="text"
      id="ContactFormName"
      name="contact[name]"
      value="{% if form.name %}{{ form.name }}{% elsif customer %}{{ customer.name }}{% endif %}"
      required
    >
  </div>

  <button type="submit">{{ 'contact.form.submit' | t }}</button>
{% endform %}
```
{% endraw %}

### Anti-Patterns to Avoid

1. **Don't use `include` (deprecated)**
   {% raw %}
   ```liquid
   {% comment %} ❌ Deprecated {% endcomment %}
   {% include 'snippet-name' %}

   {% comment %} ✅ Use render instead {% endcomment %}
   {% render 'snippet-name', param: value %}
   ```
   {% endraw %}

2. **Don't nest too deeply**
   {% raw %}
   ```liquid
   {% comment %} ❌ Too many nested conditions {% endcomment %}
   {% if product %}
     {% if product.available %}
       {% if product.variants.size > 0 %}
         {% if product.variants.first.inventory_quantity > 0 %}
           {{ product.title }}
         {% endif %}
       {% endif %}
     {% endif %}
   {% endif %}

   {% comment %} ✅ Combine conditions {% endcomment %}
   {% if product.available and product.variants.size > 0 and product.variants.first.inventory_quantity > 0 %}
     {{ product.title }}
   {% endif %}
   ```
   {% endraw %}

3. **Don't use `capture` for simple assignments**
   {% raw %}
   ```liquid
   {% comment %} ❌ Overkill for simple values {% endcomment %}
   {% capture title %}{{ product.title }}{% endcapture %}

   {% comment %} ✅ Use assign {% endcomment %}
   {% assign title = product.title %}
   ```
   {% endraw %}

4. **Don't forget to escape user input**
   {% raw %}
   ```liquid
   {% comment %} ❌ Potential XSS {% endcomment %}
   <div>{{ collection.description }}</div>

   {% comment %} ✅ Safe {% endcomment %}
   <div>{{ collection.description | escape }}</div>
   ```
   {% endraw %}

5. **Don't access nested properties without checking**
   {% raw %}
   ```liquid
   {% comment %} ❌ May cause errors {% endcomment %}
   {{ product.variants.first.option1 }}

   {% comment %} ✅ Safe {% endcomment %}
   {% if product.variants.size > 0 and product.variants.first.option1 %}
     {{ product.variants.first.option1 }}
   {% endif %}
   ```
   {% endraw %}

### Security Best Practices

1. **Always escape user-generated content**
   {% raw %}
   ```liquid
   {% comment %} ✅ Safe {% endcomment %}
   {{ customer.name | escape }}
   {{ product.description | escape }}
   {{ collection.title | escape }}
   ```
   {% endraw %}

2. **Use `json` filter for JavaScript variables**
   {% raw %}
   ```liquid
   {% comment %} ✅ Safe JSON encoding {% endcomment %}
   <script>
     var product = {{ product | json }};
   </script>
   ```
   {% endraw %}

3. **Sanitize URLs**
   {% raw %}
   ```liquid
   {% comment %} ✅ Safe {% endcomment %}
   <a href="{{ product.url | escape }}">{{ product.title }}</a>
   ```
   {% endraw %}

### Advanced Techniques

1. **Using `liquid` tag for cleaner code**
   {% raw %}
   ```liquid
   {% liquid
     assign on_sale = false
     assign discount_percentage = 0

     if product.compare_at_price > product.price
       assign on_sale = true
       assign discount = product.compare_at_price | minus: product.price
       assign discount_percentage = discount | times: 100 | divided_by: product.compare_at_price | round
     endif
   %}
   ```
   {% endraw %}

2. **Custom filters with `case` statement**
   {% raw %}
   ```liquid
   {% case product.type %}
     {% when 'Clothing' %}
       <span class="badge badge--clothing">{{ product.type }}</span>
     {% when 'Accessories' %}
       <span class="badge badge--accessories">{{ product.type }}</span>
     {% else %}
       <span class="badge">{{ product.type }}</span>
   {% endcase %}
   ```
   {% endraw %}

3. **Using `break` and `continue` in loops**
   {% raw %}
   ```liquid
   {% for product in collection.products %}
     {% if product.available == false %}
       {% continue %}
     {% endif %}
     {% if forloop.index > 10 %}
       {% break %}
     {% endif %}
     {% render 'product-card', product: product %}
   {% endfor %}
   ```
   {% endraw %}

### Debugging Tips

1. **Use `comment` tags to temporarily disable code**
   {% raw %}
   ```liquid
   {% comment %}
     {% render 'old-component' %}
   {% endcomment %}
   {% render 'new-component' %}
   ```
   {% endraw %}

2. **Output variables for debugging**
   {% raw %}
   ```liquid
   {% comment %} Debug output {% endcomment %}
   {{ product | json }}
   {{ collection | json }}
   ```
   {% endraw %}

3. **Check object properties**
   {% raw %}
   ```liquid
   {% if product %}
     <p>Product exists</p>
     {% if product.title %}
       <p>Product has title: {{ product.title }}</p>
     {% endif %}
   {% endif %}
   ```
   {% endraw %}

### When to Use Liquid vs. JavaScript

✅ **Use Liquid for:**
- Server-side rendering
- Initial page load content
- Theme customization settings
- Static or mostly static content
- SEO-critical content

❌ **Consider JavaScript/Custom Elements for:**
- Interactive components (cart drawers, modals)
- Dynamic content updates without page reload
- Client-side filtering/sorting
- Real-time features
- Complex user interactions

### Tips

- **Test with different data**: Test your templates with products that have missing images, empty descriptions, etc.
- **Use theme editor**: Make your sections configurable through the theme customizer
- **Keep snippets small**: Break large templates into reusable snippets
- **Document complex logic**: Add comments explaining why you're doing something, not just what
- **Follow Shopify conventions**: Use Shopify's recommended structure for sections, snippets, and templates
- **Validate schema**: Always include proper schema definitions for sections
- **Handle edge cases**: Always consider what happens when data is missing or null

### Resources

- <a href="https://shopify.dev/docs/api/liquid" target="_blank" rel="noopener noreferrer">Shopify Liquid Documentation</a>
- <a href="https://shopify.dev/docs/themes/liquid/reference" target="_blank" rel="noopener noreferrer">Liquid Reference</a>
- <a href="https://shopify.dev/docs/themes/architecture" target="_blank" rel="noopener noreferrer">Shopify Theme Architecture</a>
- <a href="https://shopify.dev/docs/themes/theme-development" target="_blank" rel="noopener noreferrer">Theme Development Guide</a>
- <a href="https://github.com/Shopify/liquid" target="_blank" rel="noopener noreferrer">Liquid on GitHub</a>
