---
name: shopify-commerce-engineer
description: Dawn theme architecture, metafields, Liquid templates, and e-commerce logic for Shopify stores. Use for metafield schemas, product variants, Liquid snippets, FileFlare downloads, Locksmith gating, DigitalOcean Spaces presigned URLs, subscription entitlements, and card component design. Triggers include Shopify, Liquid, metafields, Dawn theme, product split, gating, FileFlare, presigned URL, subscription logic, CTA buttons, color tokens, card components. Outputs JSON schemas, minimal-working Liquid with inline comments, and CSV-ready tables.
---

# Shopify Commerce Engineer

Structured workflows for Shopify Dawn theme architecture, metafields, and e-commerce logic.

## Core Workflow

1. Identify requirement (metafields, Liquid template, gating logic, download flow)
2. Define schema or logic specification
3. Provide minimal-working code with inline comments
4. Include testing steps and edge cases
5. Note performance and mobile considerations

## Metafield Schema Pattern

ALWAYS structure metafield definitions as JSON tables:

### Schema Definition

```json
{
  "namespace": "custom",
  "key": "download_entitlement",
  "type": "list.product_reference",
  "name": "Download Entitlements",
  "description": "Products that grant access to this download",
  "validations": [],
  "owner_type": "product"
}
```

### Implementation Table

| Field | Value | Notes |
|-------|-------|-------|
| Namespace | `custom` | Use `custom` for app-managed fields |
| Key | `download_entitlement` | Snake_case, descriptive |
| Type | `list.product_reference` | References other products |
| Owner | `product` | Attached to product object |
| Access | Storefront API | Exposed to Liquid templates |

### Supported Metafield Types

| Type | Use Case | Example Value |
|------|----------|---------------|
| `single_line_text_field` | Short text | "FAB360 Piping Edition" |
| `multi_line_text_field` | Descriptions | Product details with line breaks |
| `number_integer` | Counts | Download limit: 5 |
| `number_decimal` | Prices | Custom price: 149.99 |
| `date` | Release dates | 2025-11-15 |
| `date_time` | Timestamps | 2025-11-15T10:30:00Z |
| `boolean` | Flags | is_premium: true |
| `json` | Complex data | {"version": "2.0", "features": [...]} |
| `color` | Brand colors | #FF6B35 |
| `file_reference` | Asset links | PDF, images via Files API |
| `product_reference` | Related products | Link to parent/child product |
| `list.product_reference` | Multiple products | Array of product references |

## Liquid Template Pattern

### Basic Structure

```liquid
{% comment %}
  Component: Product Card with Gating
  Purpose: Display product with conditional CTA based on ownership
  Dependencies: Locksmith app for gating logic
{% endcomment %}

{% assign is_owned = false %}
{% if customer %}
  {% for order in customer.orders %}
    {% if order.line_items contains product %}
      {% assign is_owned = true %}
      {% break %}
    {% endif %}
  {% endfor %}
{% endif %}

<div class="product-card" data-product-id="{{ product.id }}">
  <h3>{{ product.title }}</h3>
  <p>{{ product.price | money }}</p>
  
  {% if is_owned %}
    <a href="{{ product.metafields.custom.download_url }}" class="btn btn-primary">
      Download Now
    </a>
  {% else %}
    <a href="{{ product.url }}" class="btn btn-secondary">
      Purchase to Download
    </a>
  {% endif %}
</div>
```

### Code Requirements

- Inline comments explaining logic
- Use `{% comment %}` blocks for section headers
- Assign variables for readability (`{% assign is_owned = false %}`)
- Handle null/undefined with `{% if variable %}` checks
- Use Shopify filters (`| money`, `| img_url`, `| date`)

## Product Split Strategy

For decisions like FAB360 Piping vs HVAC editions:

### Decision Matrix

| Criterion | Variants Approach | Separate Products | Recommendation |
|-----------|-------------------|-------------------|----------------|
| Inventory tracking | Single SKU, complex | Distinct SKUs | Separate |
| Customer clarity | Dropdown selection | Clear product cards | Separate |
| Pricing flexibility | Variant pricing | Independent pricing | Separate |
| Marketing | Single message | Tailored messaging | Separate |
| SEO | One product page | Multiple pages | Separate |
| Admin UX | Single edit | Multiple edits | Variants |

**Recommendation:** Separate products for FAB360 Piping and HVAC based on customer clarity, marketing flexibility, and SEO benefits.

### Implementation for Separate Products

1. **Shared Metafields Schema**
   - Define common attributes (version, release_date, category)
   - Use metafield definitions API to create

2. **Card Component System**
   - Design consistent card layout
   - Use color tokens to differentiate (Piping: blue, HVAC: orange)
   - Icon system for visual distinction

3. **Collapsible Row Content**
   - Feature comparison per SKU
   - Technical specs per edition
   - Use Dawn's accordion component

4. **CTA Logic**
   - Check customer.tags for entitlements
   - Display appropriate action (Buy, Download, Upgrade)

## Card Component Design Pattern

### Visual System

**Color Tokens:**

```css
/* Piping Edition */
--piping-primary: #0066CC;
--piping-secondary: #E6F0FF;
--piping-accent: #004C99;

/* HVAC Edition */
--hvac-primary: #FF6B35;
--hvac-secondary: #FFE6DB;
--hvac-accent: #CC5529;
```

**Icon Set:**
- Piping: Pipe icon (SVG in assets/)
- HVAC: Duct icon (SVG in assets/)

### Card Template

```liquid
{% comment %}
  Product Card - FAB360 Edition
  Variables: product, edition_type (piping|hvac)
{% endcomment %}

<div class="product-card product-card--{{ edition_type }}">
  <div class="product-card__icon">
    {% if edition_type == 'piping' %}
      {% render 'icon-pipe' %}
    {% elsif edition_type == 'hvac' %}
      {% render 'icon-duct' %}
    {% endif %}
  </div>
  
  <h3 class="product-card__title">{{ product.title }}</h3>
  <p class="product-card__description">{{ product.metafields.custom.short_description }}</p>
  
  <div class="product-card__features">
    {% assign features = product.metafields.custom.key_features | split: '|' %}
    <ul>
      {% for feature in features %}
        <li>{{ feature }}</li>
      {% endfor %}
    </ul>
  </div>
  
  <div class="product-card__cta">
    {{ cta_button }}
  </div>
</div>

<style>
  .product-card--piping {
    border-color: var(--piping-primary);
  }
  .product-card--hvac {
    border-color: var(--hvac-primary);
  }
</style>
```

## FileFlare Download Flow

### Architecture

```
Customer purchases product
  → Shopify creates order
  → Webhook to backend
  → Backend checks entitlements
  → Generate presigned URL (DigitalOcean Spaces)
  → Send to customer via email
  → Customer clicks link
  → Download initiates (URL valid 24h)
```

### DigitalOcean Spaces Presigned URL

**Python Example (Backend):**

```python
import boto3
from datetime import timedelta

def generate_presigned_url(file_key, expiration=86400):
    """
    Generate presigned URL for DO Spaces download
    
    Args:
        file_key: Path to file in Spaces (e.g., 'fab360/piping/v2.0.zip')
        expiration: URL validity in seconds (default 24h)
    
    Returns:
        Presigned URL string
    """
    session = boto3.session.Session()
    client = session.client(
        's3',
        region_name='nyc3',  # DO Spaces region
        endpoint_url='https://nyc3.digitaloceanspaces.com',
        aws_access_key_id='YOUR_SPACES_KEY',
        aws_secret_access_key='YOUR_SPACES_SECRET'
    )
    
    url = client.generate_presigned_url(
        ClientMethod='get_object',
        Params={
            'Bucket': 'your-bucket-name',
            'Key': file_key
        },
        ExpiresIn=expiration
    )
    
    return url

# Usage
download_url = generate_presigned_url('fab360/piping/v2.0.zip')
```

### Entitlement Check

```liquid
{% comment %}
  Check if customer owns product granting download access
{% endcomment %}

{% assign has_entitlement = false %}
{% assign required_product_id = 8675309 %}

{% if customer %}
  {% for order in customer.orders %}
    {% for line_item in order.line_items %}
      {% if line_item.product_id == required_product_id %}
        {% assign has_entitlement = true %}
        {% break %}
      {% endif %}
    {% endfor %}
    {% if has_entitlement %}
      {% break %}
    {% endif %}
  {% endfor %}
{% endif %}

{% if has_entitlement %}
  <a href="{{ product.metafields.custom.download_url }}" class="btn-download">
    Download FAB360 Piping Edition
  </a>
{% else %}
  <a href="{{ product.url }}" class="btn-purchase">
    Purchase to Download
  </a>
{% endif %}
```

## Locksmith Gating Pattern

### Product-Level Gating

```liquid
{% comment %}
  Locksmith lock check
  Lock name: "FAB360 Premium Access"
{% endcomment %}

{% if lock.user_has_key %}
  {% comment %} Customer has access {% endcomment %}
  <div class="premium-content">
    {{ product.metafields.custom.premium_features }}
  </div>
{% else %}
  {% comment %} Show upgrade CTA {% endcomment %}
  <div class="upgrade-prompt">
    <p>Upgrade to Premium to access advanced features.</p>
    <a href="/products/fab360-premium" class="btn-upgrade">
      Upgrade Now
    </a>
  </div>
{% endif %}
```

### Content-Level Gating

For gated sections within a product page:

```liquid
<div class="product-details">
  <h2>Standard Features</h2>
  <ul>
    <li>Feature 1</li>
    <li>Feature 2</li>
  </ul>
  
  {% comment %} Gated advanced features {% endcomment %}
  {% if lock.user_has_key %}
    <h2>Advanced Features (Premium)</h2>
    <ul>
      <li>Advanced Feature 1</li>
      <li>Advanced Feature 2</li>
    </ul>
  {% endif %}
</div>
```

## Subscription Entitlement Logic

### Check Active Subscription

```liquid
{% assign has_active_subscription = false %}
{% assign subscription_product_id = 8675310 %}

{% if customer %}
  {% for order in customer.orders %}
    {% for line_item in order.line_items %}
      {% if line_item.product_id == subscription_product_id %}
        {% if line_item.properties.subscription_status == 'active' %}
          {% assign has_active_subscription = true %}
          {% break %}
        {% endif %}
      {% endif %}
    {% endfor %}
  {% endfor %}
{% endif %}

{% if has_active_subscription %}
  <div class="subscriber-content">
    <p>You have an active subscription. Download the latest version:</p>
    <a href="{{ product.metafields.custom.subscriber_download_url }}">
      Download Latest Release
    </a>
  </div>
{% else %}
  <div class="non-subscriber-content">
    <p>Subscribe for ongoing updates and support.</p>
    <a href="/products/fab360-subscription">Subscribe Now</a>
  </div>
{% endif %}
```

## Dawn Theme Section Pattern

### Custom Section Structure

```liquid
{% comment %}
  Section: Product Grid with Filtering
  Settings: Collection, filter tags, columns
{% endcomment %}

{{ 'component-product-grid.css' | asset_url | stylesheet_tag }}

<div class="product-grid-section">
  <div class="container">
    {% if section.settings.show_filters %}
      <div class="filter-bar">
        {% for tag in collection.all_tags %}
          <button class="filter-tag" data-tag="{{ tag }}">
            {{ tag }}
          </button>
        {% endfor %}
      </div>
    {% endif %}
    
    <div class="product-grid" data-columns="{{ section.settings.columns }}">
      {% for product in collection.products %}
        <div class="product-grid__item" data-tags="{{ product.tags | join: ',' }}">
          {% render 'product-card', product: product %}
        </div>
      {% endfor %}
    </div>
  </div>
</div>

{% schema %}
{
  "name": "Product Grid",
  "settings": [
    {
      "type": "collection",
      "id": "collection",
      "label": "Collection"
    },
    {
      "type": "checkbox",
      "id": "show_filters",
      "label": "Show filter tags",
      "default": true
    },
    {
      "type": "range",
      "id": "columns",
      "label": "Columns per row",
      "min": 2,
      "max": 4,
      "step": 1,
      "default": 3
    }
  ],
  "presets": [
    {
      "name": "Product Grid"
    }
  ]
}
{% endschema %}
```

## Performance Considerations

### Liquid Optimization

1. **Minimize loops:**
   - Use `limit` and `break` to exit early
   - Cache results in variables

2. **Asset loading:**
   - Use `preload` for critical CSS/JS
   - Lazy load images with `loading="lazy"`

3. **API calls:**
   - Batch metafield reads
   - Use Storefront API for client-side data

### Mobile Optimization

- Test on iOS Safari (primary user device)
- Ensure CTA buttons min 44px tap target
- Use responsive images (`| img_url: '400x'`)
- Progressive enhancement for JS features

## Testing Checklist

Before deploying Liquid changes:

- [ ] Preview in Dawn theme customizer
- [ ] Test with customer account (logged in)
- [ ] Test without customer account (logged out)
- [ ] Verify metafield values display correctly
- [ ] Test on mobile (iOS Safari)
- [ ] Check console for JS errors
- [ ] Validate HTML markup
- [ ] Test Locksmith locks if applicable

## Common Edge Cases

### Metafield Handling

```liquid
{% comment %} Always check if metafield exists {% endcomment %}
{% if product.metafields.custom.download_url %}
  <a href="{{ product.metafields.custom.download_url }}">Download</a>
{% else %}
  <p>Download not available</p>
{% endif %}

{% comment %} Handle list metafields {% endcomment %}
{% if product.metafields.custom.related_products %}
  {% for related_product in product.metafields.custom.related_products.value %}
    {{ related_product.title }}
  {% endfor %}
{% endif %}
```

### Customer Order History

```liquid
{% comment %} Check if customer exists first {% endcomment %}
{% if customer %}
  {% comment %} Safely iterate orders {% endcomment %}
  {% if customer.orders.size > 0 %}
    {% for order in customer.orders %}
      {% comment %} Process order {% endcomment %}
    {% endfor %}
  {% else %}
    <p>No orders found.</p>
  {% endif %}
{% else %}
  <p>Please log in to view orders.</p>
{% endif %}
```

## Output Style

- **Code:** Minimal-working examples with inline comments
- **Tables:** CSV-ready format (pipe-separated)
- **Structure:** Headings, short code blocks, step-by-step
- **Dates:** America/Chicago time, absolute dates
- **Tone:** Technical and precise

## References to Create

1. `references/metafield-schema-patterns.md`
   - Common schema definitions
   - Type validation rules
   - Namespace conventions

2. `references/liquid-filters-cheatsheet.md`
   - Shopify-specific filters
   - Custom filter examples
   - Performance notes

3. `scripts/validate_liquid.py`
   - Syntax checker for Liquid
   - Metafield reference validator

4. `assets/card-component-template.liquid`
   - Reusable card component
   - Variations for different product types
