---
layout: default
title: BEM
description: Block Element Modifier
parent: /developers.html
---

## BEM

BEM (Block Element Modifier) is a CSS naming methodology that helps create reusable components and avoid naming conflicts. It provides a clear structure for organizing your CSS classes.

### What is BEM?

BEM stands for:
- **Block**: A standalone component that is meaningful on its own (e.g., `button`, `card`, `menu`)
- **Element**: A part of a block that has no standalone meaning (e.g., `button__icon`, `card__title`)
- **Modifier**: A flag on a block or element that changes its appearance or behavior (e.g., `button--primary`, `card--featured`)

### Naming Convention

BEM uses a specific naming pattern:
```
block__element--modifier
```

- **Block**: Single word, lowercase (e.g., `button`, `card`)
- **Element**: Separated by double underscore `__` (e.g., `button__icon`)
- **Modifier**: Separated by double hyphen `--` (e.g., `button--primary`)

### Examples

#### Basic Block
```liquid
<div class="card">
  <h2 class="card__title">{{ 'card.title' | t }}</h2>
  <p class="card__content">{{ 'card.content' | t }}</p>
  <button class="card__button">{{ 'card.action' | t }}</button>
</div>
```

```css
.card { }
.card__title { }
.card__content { }
.card__button { }
```

#### Block with Modifiers
```liquid
<button class="button button--primary">{{ 'button.primary' | t }}</button>
<button class="button button--secondary">{{ 'button.secondary' | t }}</button>
<button class="button button--large">{{ 'button.large' | t }}</button>
```

```css
.button { }
.button--primary { }
.button--secondary { }
.button--large { }
```

#### Element with Modifiers
```liquid
<div class="card card--featured">
  <h2 class="card__title card__title--highlighted">{{ 'card.featured_title' | t }}</h2>
  <p class="card__content">{{ 'card.content' | t }}</p>
</div>
```

```css
.card { }
.card--featured { }
.card__title { }
.card__title--highlighted { }
```

### Best Practices

1. **Keep blocks independent**
   - Blocks should be reusable and not depend on parent elements
   - Avoid nesting blocks inside blocks (use composition instead)

2. **Use modifiers, not separate classes**
   ```liquid
   {% comment %} ❌ Bad {% endcomment %}
   <button class="button primary-button">{{ 'button.click' | t }}</button>

   {% comment %} ✅ Good {% endcomment %}
   <button class="button button--primary">{{ 'button.click' | t }}</button>
   ```

3. **Don't create elements of elements**
   ```liquid
   {% comment %} ❌ Bad {% endcomment %}
   <div class="card">
     <div class="card__header">
       <h2 class="card__header__title">{{ 'card.title' | t }}</h2>
     </div>
   </div>

   {% comment %} ✅ Good {% endcomment %}
   <div class="card">
     <div class="card__header">
       <h2 class="card__title">{{ 'card.title' | t }}</h2>
     </div>
   </div>
   ```

4. **Use semantic HTML**
   - BEM is for CSS organization, not HTML structure
   - Always use appropriate HTML elements

5. **Keep class names short but descriptive**
   ```liquid
   {% comment %} ❌ Too verbose {% endcomment %}
   <div class="user-profile-card-container">

   {% comment %} ✅ Better {% endcomment %}
   <div class="profile-card">
   ```

6. **Modifiers should be optional**
   - Blocks and elements should work without modifiers
   - Modifiers enhance or change appearance/behavior

### Common Patterns

#### Navigation Menu
```liquid
<nav class="nav">
  <ul class="nav__list">
    {% for link in linklists.main-menu.links %}
      <li class="nav__item">
        <a href="{{ link.url }}" class="nav__link {% if link.active %}nav__link--active{% endif %}">
          {{ link.title }}
        </a>
      </li>
    {% endfor %}
  </ul>
</nav>
```

#### Form Elements
```liquid
<form class="form" action="{{ routes.contact }}" method="post">
  <div class="form__group">
    <label class="form__label" for="email">{{ 'form.email' | t }}</label>
    <input
      class="form__input {% if form.errors contains 'email' %}form__input--error{% endif %}"
      type="email"
      id="email"
      name="contact[email]"
      value="{{ form.email }}"
    >
    {% if form.errors contains 'email' %}
      <span class="form__error">{{ 'form.email_error' | t }}</span>
    {% endif %}
  </div>
  <button
    class="form__submit {% unless form.posted_successfully? %}{% if form.errors %}form__submit--disabled{% endif %}{% endunless %}"
    type="submit"
    {% if form.errors %}disabled{% endif %}
  >
    {{ 'form.submit' | t }}
  </button>
</form>
```

### When to Use BEM

✅ **Use BEM when:**
- Building component-based interfaces
- Working on large projects with multiple developers
- You need clear, maintainable CSS
- You want to avoid CSS conflicts

❌ **Consider alternatives when:**
- Working on very small projects
- Using CSS-in-JS solutions
- Using utility-first frameworks (Tailwind, etc.)

### Tips

- **Be consistent**: Stick to the naming convention throughout the project
- **Document your blocks**: Keep a style guide or component library
- **Don't over-nest**: Keep your BEM structure flat (max 1 level of elements)
- **Use mixins/utilities**: Combine BEM with utility classes for spacing, typography, etc.

### Resources

- <a href="https://en.bem.info/" target="_blank" rel="noopener noreferrer">Official BEM Website</a>
- <a href="https://css-tricks.com/bem-101/" target="_blank" rel="noopener noreferrer">BEM 101</a>
- <a href="https://9elements.com/bem-cheat-sheet/" target="_blank" rel="noopener noreferrer">BEM Cheat Sheet</a>
