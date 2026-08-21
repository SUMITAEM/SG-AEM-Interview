# AEM Interview Preparation — Part 5: Components & HTL (Sightly)

---

## AEM Components

A component is a reusable module for rendering content. It consists of:
- **HTL template** (`.html`) — Rendering logic
- **Dialog** (`_cq_dialog/.content.xml`) — Author editing UI
- **ClientLib** — CSS/JS for the component
- **Sling Model** (optional) — Java business logic
- **Edit config** (optional) — Author experience configuration

### Component Structure
```
/apps/myapp/components/hero/
├── .content.xml              → Component definition
├── hero.html                 → Main HTL template
├── _cq_dialog/
│   └── .content.xml          → Touch UI dialog
├── _cq_editConfig/
│   └── .content.xml          → Edit configuration
├── _cq_design_dialog/
│   └── .content.xml          → Design dialog (style system)
└── clientlibs/
    ├── .content.xml
    ├── css/
    │   └── hero.css
    └── js/
        └── hero.js
```

### Component Definition (`.content.xml`)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0"
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="Hero Component"
    jcr:description="A hero banner with image and CTA"
    componentGroup="MyApp - Content"
    sling:resourceSuperType="core/wcm/components/commons/editor/dialog"/>
```

---

## HTL (HTML Template Language)

HTL (formerly Sightly) replaced JSP as AEM's templating language. It's:
- **Secure by default** (automatic XSS protection)
- **Separation of concerns** (logic in Java, rendering in HTML)
- **Valid HTML** (uses data attributes)

### HTL Block Statements

| Statement | Purpose | Example |
|-----------|---------|---------|
| `data-sly-use` | Load Java/JS logic | `data-sly-use.model="com.app.Model"` |
| `data-sly-test` | Conditional | `data-sly-test="${model.show}"` |
| `data-sly-list` | Loop over collection | `data-sly-list="${model.items}"` |
| `data-sly-repeat` | Loop keeping element | `data-sly-repeat="${model.items}"` |
| `data-sly-resource` | Include sub-resource | `data-sly-resource="${'child'}"` |
| `data-sly-include` | Include HTL template | `data-sly-include="partial.html"` |
| `data-sly-template` | Define reusable block | `data-sly-template.tmpl="${@ param}"` |
| `data-sly-call` | Call template | `data-sly-call="${tmpl @ param=val}"` |
| `data-sly-text` | Set text content | `data-sly-text="${model.title}"` |
| `data-sly-attribute` | Set attribute | `data-sly-attribute.class="${model.cls}"` |
| `data-sly-element` | Change element tag | `data-sly-element="${model.tag}"` |
| `data-sly-unwrap` | Remove wrapper element | `data-sly-unwrap` |

---

### HTL Expression Examples

```html
<!-- Basic property output (XSS-safe by default) -->
<h1>${properties.jcr:title}</h1>

<!-- With display context -->
<a href="${properties.link @ context='uri'}">${properties.linkText}</a>
<div>${properties.richText @ context='html'}</div>

<!-- Using a Sling Model -->
<sly data-sly-use.hero="com.myapp.core.models.HeroModel">
    <section class="hero">
        <h1>${hero.title}</h1>
        <p>${hero.description}</p>
        <a href="${hero.ctaLink}" class="btn">${hero.ctaText}</a>
    </section>
</sly>

<!-- Conditionals -->
<div data-sly-test="${hero.hasImage}">
    <img src="${hero.imagePath}" alt="${hero.imageAlt}"/>
</div>
<div data-sly-test="${!hero.hasImage}">
    <div class="placeholder">No image configured</div>
</div>

<!-- Ternary-like -->
<div class="${hero.isLarge ? 'hero--large' : 'hero--small'}">

<!-- Lists -->
<ul data-sly-list="${hero.navItems}">
    <li class="${itemList.first ? 'first' : ''}">
        <a href="${item.url}">${item.title}</a>
    </li>
</ul>

<!-- List variables available -->
<!-- item     = current item -->
<!-- itemList.index  = 0-based index -->
<!-- itemList.count  = 1-based count -->
<!-- itemList.first  = is first? -->
<!-- itemList.last   = is last? -->

<!-- data-sly-repeat (keeps host element) -->
<div data-sly-repeat="${hero.cards}" class="card">
    <h3>${item.title}</h3>
</div>
<!-- Produces: <div class="card">...</div> for each card -->

<!-- Include another resource -->
<div data-sly-resource="${'header' @ resourceType='myapp/components/header'}"></div>

<!-- Template definition and call -->
<sly data-sly-template.badge="${@ text, color}">
    <span class="badge badge--${color}">${text}</span>
</sly>

<sly data-sly-call="${badge @ text='New', color='green'}"/>
```

---

### HTL Display Contexts

| Context | Use For | Protection |
|---------|---------|-----------|
| `text` (default) | Plain text content | HTML entity encoding |
| `html` | Rich text (trusted HTML) | Minimal sanitization |
| `attribute` | HTML attributes | Attribute encoding |
| `uri` | href/src URLs | URL validation |
| `number` | Numeric values | Ensures numeric |
| `scriptString` | Inside JS strings | JS string encoding |
| `styleString` | Inside CSS values | CSS encoding |
| `unsafe` | No protection | **AVOID** — XSS risk |

---

## Component Dialogs (Touch UI)

Dialogs use Granite UI / Coral UI components:

### Basic Dialog Structure
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    xmlns:granite="http://www.adobe.com/jcr/granite/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="Hero"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <tabs jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/tabs">
                <items jcr:primaryType="nt:unstructured">
                    <general jcr:primaryType="nt:unstructured"
                        jcr:title="General"
                        sling:resourceType="granite/ui/components/coral/foundation/container"
                        margin="{Boolean}true">
                        <items jcr:primaryType="nt:unstructured">
                            <title jcr:primaryType="nt:unstructured"
                                sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                                fieldLabel="Title"
                                name="./title"
                                required="{Boolean}true"/>
                            <description jcr:primaryType="nt:unstructured"
                                sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
                                fieldLabel="Description"
                                name="./description"/>
                            <image jcr:primaryType="nt:unstructured"
                                sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                                fieldLabel="Image"
                                name="./imagePath"
                                rootPath="/content/dam"/>
                        </items>
                    </general>
                </items>
            </tabs>
        </items>
    </content>
</jcr:root>
```

### Common Dialog Field Types

| Resource Type | Purpose |
|---------------|---------|
| `.../form/textfield` | Single line text |
| `.../form/textarea` | Multi-line text |
| `.../form/richtext` | Rich text editor |
| `.../form/pathfield` | Path browser |
| `.../form/checkbox` | Boolean toggle |
| `.../form/select` | Dropdown |
| `.../form/radiogroup` | Radio buttons |
| `.../form/numberfield` | Numeric input |
| `.../form/datepicker` | Date selection |
| `.../form/colorfield` | Color picker |
| `.../form/multifield` | Repeatable fields |
| `.../form/hidden` | Hidden value |
| `.../include` | Include another dialog |
| `granite/ui/components/coral/foundation/form/fileupload` | File upload |

---

## Component Inheritance

### Resource Super Type
```xml
<!-- Child component inherits from parent -->
<jcr:root
    jcr:primaryType="cq:Component"
    jcr:title="Custom Text"
    sling:resourceSuperType="core/wcm/components/text/v2/text"/>
```

This means:
- If the child doesn't have a script, Sling uses parent's script
- Dialog fields can be extended
- Core Components are commonly used as super types

---

## AEM Core Components

Pre-built, production-ready components by Adobe:
- **Text, Title, Image, Teaser, List**
- **Carousel, Tabs, Accordion, Container**
- **Navigation, Breadcrumb, Language Navigation**
- **Form Container, Form Text, Form Options**
- **Download, Embed, PDF Viewer**

Always extend Core Components rather than building from scratch.

---

## MCQs — Components & HTL

### Q1: Which HTL statement is used to conditionally render content?
- A) data-sly-if
- B) data-sly-test ✅ **CORRECT**
- C) data-sly-show
- D) data-sly-when

> **Explanation:** `data-sly-test` conditionally renders the element and its children.

### Q2: What is the default display context in HTL?
- A) html
- B) unsafe
- C) text ✅ **CORRECT**
- D) attribute

> **Explanation:** Default is `text`, which HTML-encodes output to prevent XSS.

### Q3: How do you iterate AND keep the host element in HTL?
- A) data-sly-list
- B) data-sly-repeat ✅ **CORRECT**
- C) data-sly-each
- D) data-sly-for

> **Explanation:** `data-sly-repeat` repeats the host element. `data-sly-list` removes it and only keeps children.

### Q4: Where is a component's Touch UI dialog defined?
- A) dialog.xml
- B) _cq_dialog/.content.xml ✅ **CORRECT**
- C) cq:dialog/definition.xml
- D) touch-dialog.xml

> **Explanation:** Touch UI dialogs go in the `_cq_dialog` folder with a `.content.xml` file.

### Q5: What does `data-sly-use` do?
- A) Includes a CSS file
- B) Initializes a Java/JavaScript Use-object ✅ **CORRECT**
- C) Creates a new DOM element
- D) Defines a template

> **Explanation:** `data-sly-use` loads and initializes a Use-object (Sling Model, WCMUsePojo, or JS file).

### Q6: Which display context should you use for rich text content?
- A) text
- B) html ✅ **CORRECT**
- C) unsafe
- D) richtext

> **Explanation:** `html` context allows safe HTML tags while filtering dangerous ones.

### Q7: What property in .content.xml controls which group a component appears in?
- A) cq:group
- B) componentGroup ✅ **CORRECT**
- C) jcr:group
- D) sling:group

> **Explanation:** `componentGroup` determines the group in the component browser (side panel).

### Q8: What does `sling:resourceSuperType` enable?
- A) Multiple inheritance
- B) Component inheritance (proxy pattern) ✅ **CORRECT**
- C) Template binding
- D) Dialog merging

> **Explanation:** It enables inheritance — child component uses parent's scripts/dialogs when it doesn't have its own.
