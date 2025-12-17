---
layout: default
title: Custom Elements
description: DOM loaded content who?
parent: /developers.html
---

## Custom Elements

Custom Elements are a web standard that allows you to create your own HTML elements with custom behavior. They're part of the Web Components specification and provide a way to build reusable, encapsulated components for Shopify themes.

### What are Custom Elements?

Custom Elements let you define new HTML tags that have their own JavaScript behavior. They're perfect for creating reusable components in Shopify themes, such as product cards, cart drawers, quantity selectors, and more.

### Basic Syntax

Custom Elements must:
- Have a name with at least one hyphen (e.g., `product-card`, `cart-drawer`)
- Extend the `HTMLElement` class
- Be registered with `customElements.define()`

### Basic Example

```javascript
class ProductCard extends HTMLElement {
  constructor() {
    super();
    // Initialize your element
  }

  connectedCallback() {
    // Called when element is added to the DOM
    this.render();
  }

  disconnectedCallback() {
    // Called when element is removed from the DOM
    // Clean up event listeners, timers, etc.
  }

  render() {
    this.innerHTML = `
      <div class="product-card">
        <img src="${this.getAttribute('image')}" alt="${this.getAttribute('title')}">
        <h3>${this.getAttribute('title')}</h3>
        <p class="price">${this.getAttribute('price')}</p>
      </div>
    `;
  }
}

// Register the custom element
customElements.define('product-card', ProductCard);
```

```html
<product-card
  image="/products/t-shirt.jpg"
  title="Cool T-Shirt"
  price="$29.99">
</product-card>
```

### Lifecycle Methods

Custom Elements have several lifecycle callbacks:

1. **`constructor()`**: Called when the element is created
2. **`connectedCallback()`**: Called when element is inserted into the DOM
3. **`disconnectedCallback()`**: Called when element is removed from the DOM
4. **`attributeChangedCallback(name, oldValue, newValue)`**: Called when observed attributes change
5. **`adoptedCallback()`**: Called when element is moved to a new document

### Observing Attributes

To react to attribute changes, you need to:
1. Define which attributes to observe
2. Implement `attributeChangedCallback`

```javascript
class ProductCard extends HTMLElement {
  static get observedAttributes() {
    return ['price', 'sale-price', 'on-sale'];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue !== newValue) {
      this.updatePrice();
    }
  }

  updatePrice() {
    const priceEl = this.querySelector('.price');
    if (this.hasAttribute('on-sale')) {
      priceEl.innerHTML = `
        <span class="price--sale">${this.getAttribute('sale-price')}</span>
        <span class="price--original">${this.getAttribute('price')}</span>
      `;
    } else {
      priceEl.textContent = this.getAttribute('price');
    }
  }
}
```

### Shopify-Specific Examples

#### Product Card Component

```javascript
class ShopifyProductCard extends HTMLElement {
  constructor() {
    super();
    this.productHandle = this.getAttribute('handle');
  }

  connectedCallback() {
    this.fetchProduct();
  }

  async fetchProduct() {
    try {
      const response = await fetch(`/products/${this.productHandle}.js`);
      const product = await response.json();
      this.render(product);
    } catch (error) {
      console.error('Error fetching product:', error);
    }
  }

  render(product) {
    const variant = product.variants[0];
    this.innerHTML = `
      <a href="/products/${product.handle}" class="product-card">
        <div class="product-card__image">
          <img src="${product.featured_image}" alt="${product.title}" loading="lazy">
          ${product.compare_at_price > product.price ? '<span class="badge badge--sale">Sale</span>' : ''}
        </div>
        <div class="product-card__content">
          <h3 class="product-card__title">${product.title}</h3>
          <div class="product-card__price">
            ${this.formatPrice(variant.price)}
            ${variant.compare_at_price ? `<span class="price--compare">${this.formatPrice(variant.compare_at_price)}</span>` : ''}
          </div>
        </div>
      </a>
    `;
  }

  formatPrice(price) {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD'
    }).format(price / 100);
  }
}

customElements.define('shopify-product-card', ShopifyProductCard);
```

```html
<shopify-product-card handle="cool-t-shirt"></shopify-product-card>
```

#### Quantity Selector

```javascript
class QuantitySelector extends HTMLElement {
  constructor() {
    super();
    this.quantity = parseInt(this.getAttribute('value') || '1');
    this.min = parseInt(this.getAttribute('min') || '1');
    this.max = parseInt(this.getAttribute('max') || '999');
  }

  connectedCallback() {
    this.render();
    this.attachEventListeners();
  }

  render() {
    this.innerHTML = `
      <button class="quantity-selector__button" data-action="decrease" aria-label="Decrease quantity">−</button>
      <input
        type="number"
        class="quantity-selector__input"
        value="${this.quantity}"
        min="${this.min}"
        max="${this.max}"
        aria-label="Quantity"
      >
      <button class="quantity-selector__button" data-action="increase" aria-label="Increase quantity">+</button>
    `;
  }

  attachEventListeners() {
    const input = this.querySelector('.quantity-selector__input');
    const buttons = this.querySelectorAll('[data-action]');

    buttons.forEach(button => {
      button.addEventListener('click', (e) => {
        const action = e.currentTarget.dataset.action;
        if (action === 'increase' && this.quantity < this.max) {
          this.quantity++;
        } else if (action === 'decrease' && this.quantity > this.min) {
          this.quantity--;
        }
        input.value = this.quantity;
        this.dispatchChangeEvent();
      });
    });

    input.addEventListener('change', (e) => {
      const value = parseInt(e.target.value) || this.min;
      this.quantity = Math.max(this.min, Math.min(this.max, value));
      input.value = this.quantity;
      this.dispatchChangeEvent();
    });
  }

  dispatchChangeEvent() {
    this.dispatchEvent(new CustomEvent('quantity-change', {
      detail: { quantity: this.quantity },
      bubbles: true
    }));
  }

  static get observedAttributes() {
    return ['value', 'min', 'max'];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue !== newValue && this.querySelector('.quantity-selector__input')) {
      this.quantity = parseInt(newValue) || this.min;
      this.querySelector('.quantity-selector__input').value = this.quantity;
    }
  }
}

customElements.define('quantity-selector', QuantitySelector);
```

```html
<quantity-selector value="1" min="1" max="10"></quantity-selector>

<script>
  document.querySelector('quantity-selector').addEventListener('quantity-change', (e) => {
    console.log('New quantity:', e.detail.quantity);
  });
</script>
```

#### Add to Cart Button

```javascript
class AddToCartButton extends HTMLElement {
  constructor() {
    super();
    this.variantId = this.getAttribute('variant-id');
    this.loading = false;
  }

  connectedCallback() {
    this.render();
    this.attachEventListeners();
  }

  render() {
    this.innerHTML = `
      <button class="add-to-cart-button" ${this.loading ? 'disabled' : ''}>
        <span class="add-to-cart-button__text">${this.loading ? 'Adding...' : 'Add to Cart'}</span>
      </button>
    `;
  }

  async attachEventListeners() {
    const button = this.querySelector('.add-to-cart-button');

    button.addEventListener('click', async () => {
      if (this.loading) return;

      this.loading = true;
      this.render();

      try {
        const response = await fetch('/cart/add.js', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            items: [{
              id: this.variantId,
              quantity: 1
            }]
          })
        });

        if (response.ok) {
          const data = await response.json();
          this.dispatchEvent(new CustomEvent('cart-added', {
            detail: data,
            bubbles: true
          }));

          // Update cart count if cart-count element exists
          const cartCount = document.querySelector('cart-count');
          if (cartCount) {
            cartCount.updateCount();
          }
        } else {
          throw new Error('Failed to add to cart');
        }
      } catch (error) {
        console.error('Error adding to cart:', error);
        this.showError();
      } finally {
        this.loading = false;
        this.render();
      }
    });
  }

  showError() {
    const button = this.querySelector('.add-to-cart-button');
    button.textContent = 'Error - Try Again';
    button.classList.add('add-to-cart-button--error');

    setTimeout(() => {
      button.textContent = 'Add to Cart';
      button.classList.remove('add-to-cart-button--error');
    }, 3000);
  }

  static get observedAttributes() {
    return ['variant-id'];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (name === 'variant-id' && oldValue !== newValue) {
      this.variantId = newValue;
    }
  }
}

customElements.define('add-to-cart-button', AddToCartButton);
```

```html
<add-to-cart-button variant-id="123456789"></add-to-cart-button>

<script>
  document.querySelector('add-to-cart-button').addEventListener('cart-added', (e) => {
    // Show success message, open cart drawer, etc.
    console.log('Item added:', e.detail);
  });
</script>
```

#### Cart Count Badge

```javascript
class CartCount extends HTMLElement {
  constructor() {
    super();
    this.count = 0;
  }

  connectedCallback() {
    this.updateCount();
    // Listen for cart updates from other components
    document.addEventListener('cart-added', () => this.updateCount());
    document.addEventListener('cart-updated', () => this.updateCount());
  }

  async updateCount() {
    try {
      const response = await fetch('/cart.js');
      const cart = await response.json();
      this.count = cart.item_count;
      this.render();
    } catch (error) {
      console.error('Error fetching cart:', error);
    }
  }

  render() {
    this.innerHTML = `
      <span class="cart-count" aria-label="${this.count} items in cart">
        ${this.count}
      </span>
    `;
  }
}

customElements.define('cart-count', CartCount);
```

```html
<a href="/cart" class="cart-link">
  Cart
  <cart-count></cart-count>
</a>
```

### Best Practices

1. **Use descriptive names with hyphens**
   ```javascript
   // ✅ Good
   customElements.define('product-card', ProductCard);
   customElements.define('cart-drawer', CartDrawer);

   // ❌ Bad (no hyphen)
   customElements.define('productcard', ProductCard);
   ```

2. **Clean up in disconnectedCallback**
   ```javascript
   connectedCallback() {
     this.handleClick = this.handleClick.bind(this);
     document.addEventListener('click', this.handleClick);
   }

   disconnectedCallback() {
     document.removeEventListener('click', this.handleClick);
   }
   ```

3. **Use data attributes for configuration**
   ```html
   <!-- ✅ Good -->
   <product-card data-handle="cool-t-shirt"></product-card>

   <!-- ❌ Less flexible -->
   <product-card handle="cool-t-shirt"></product-card>
   ```

4. **Handle loading states**
   ```javascript
   render() {
     if (this.loading) {
       this.innerHTML = '<div class="loading">Loading...</div>';
       return;
     }
     // Render actual content
   }
   ```

5. **Dispatch custom events for communication**
   ```javascript
   this.dispatchEvent(new CustomEvent('product-loaded', {
     detail: { product: this.product },
     bubbles: true,
     cancelable: true
   }));
   ```

6. **Validate required attributes**
   ```javascript
   connectedCallback() {
     if (!this.hasAttribute('variant-id')) {
      console.error('variant-id is required');
      return;
     }
     // Continue initialization
   }
   ```

7. **Use private methods for internal logic**
   ```javascript
   class ProductCard extends HTMLElement {
     #formatPrice(price) {
       // Private method
       return new Intl.NumberFormat('en-US', {
         style: 'currency',
         currency: 'USD'
       }).format(price / 100);
     }
   }
   ```

### Shopify-Specific Considerations

1. **Theme Editor Compatibility**
   - Custom elements work well with Shopify's theme editor
   - Use data attributes that can be set via section settings
   ```liquid
   <product-card data-handle="{{ section.settings.featured_product }}"></product-card>
   ```

2. **Lazy Loading**
   - Use `loading="lazy"` for images
   - Consider lazy-loading entire components with Intersection Observer
   ```javascript
   connectedCallback() {
     if ('IntersectionObserver' in window) {
       const observer = new IntersectionObserver((entries) => {
         entries.forEach(entry => {
           if (entry.isIntersecting) {
             this.loadContent();
             observer.unobserve(this);
           }
         });
       });
       observer.observe(this);
     } else {
       this.loadContent();
     }
   }
   ```

3. **Cart API Integration**
   - Always handle errors when using `/cart/add.js` and `/cart.js`
   - Update cart UI components when cart changes
   - Consider rate limiting for rapid cart updates

4. **Performance**
   - Register custom elements early in your theme
   - Consider code splitting for large components
   - Use `requestIdleCallback` for non-critical initialization

5. **Accessibility**
   - Always include proper ARIA labels
   - Ensure keyboard navigation works
   - Test with screen readers

### Common Patterns

#### Component with Slots

```javascript
class ProductCard extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <div class="product-card">
        <slot name="image"></slot>
        <div class="product-card__content">
          <slot name="title"></slot>
          <slot name="price"></slot>
        </div>
      </div>
    `;
  }
}

customElements.define('product-card', ProductCard);
```

```html
<product-card>
  <img slot="image" src="/product.jpg" alt="Product">
  <h3 slot="title">Product Name</h3>
  <span slot="price">$29.99</span>
</product-card>
```

#### Component with Template

```javascript
class ProductCard extends HTMLElement {
  connectedCallback() {
    const template = document.getElementById('product-card-template');
    const content = template.content.cloneNode(true);

    // Populate template with data
    content.querySelector('.product-title').textContent = this.getAttribute('title');
    content.querySelector('.product-price').textContent = this.getAttribute('price');
    content.querySelector('.product-image').src = this.getAttribute('image');

    this.appendChild(content);
  }
}
```

```html
<template id="product-card-template">
  <div class="product-card">
    <img class="product-image" src="" alt="">
    <h3 class="product-title"></h3>
    <p class="product-price"></p>
  </div>
</template>
```

### Browser Support

Custom Elements are supported in all modern browsers:
- Chrome 67+
- Firefox 63+
- Safari 10.1+
- Edge 79+

For older browsers, use a polyfill:
```html
<script src="https://unpkg.com/@webcomponents/custom-elements@1.5.0/custom-elements.min.js"></script>
```

### When to Use Custom Elements

✅ **Use Custom Elements when:**
- Building reusable components across multiple pages
- Creating interactive elements (cart, product selectors, etc.)
- You want encapsulated, self-contained functionality
- Working with dynamic content from Shopify APIs

❌ **Consider alternatives when:**
- Simple static content (use Liquid templates)
- Very simple interactions (vanilla JS might be simpler)
- You need to support very old browsers without polyfills

### Tips

- **Start simple**: Begin with basic functionality and add features incrementally
- **Test thoroughly**: Custom elements can have timing issues with DOM manipulation
- **Document your components**: Create a component library or style guide
- **Keep components focused**: One component, one responsibility

### Resources

- <a href="https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements" target="_blank" rel="noopener noreferrer">MDN: Using Custom Elements</a>
- <a href="https://web.dev/custom-elements-v1/" target="_blank" rel="noopener noreferrer">Web.dev: Custom Elements v1</a>
- <a href="https://shopify.dev/docs/api/ajax" target="_blank" rel="noopener noreferrer">Shopify Ajax API</a>
- <a href="https://shopify.dev/docs/themes/architecture/sections" target="_blank" rel="noopener noreferrer">Shopify Theme Sections</a>
