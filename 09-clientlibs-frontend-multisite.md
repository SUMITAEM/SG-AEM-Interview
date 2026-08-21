# AEM Interview Preparation — Part 9: ClientLibs, Frontend & Multi-Site Manager

---

## AEM Client Libraries (ClientLibs)

ClientLibs manage CSS and JavaScript delivery in AEM.

### ClientLib Structure
```
/apps/myapp/components/hero/clientlibs/
├── .content.xml
├── css/
│   └── hero.css
├── js/
│   └── hero.js
├── css.txt          → Lists CSS files to include (order matters)
└── js.txt           → Lists JS files to include (order matters)
```

### ClientLib Definition (.content.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0"
    jcr:primaryType="cq:ClientLibraryFolder"
    categories="[myapp.hero]"
    allowProxy="{Boolean}true"/>
```

### Key Properties

| Property | Purpose | Example |
|----------|---------|---------|
| `categories` | Unique identifier(s) to include | `[myapp.base, myapp.hero]` |
| `dependencies` | Load BEFORE this lib | `[myapp.base]` |
| `embed` | Merge another lib INTO this | `[myapp.utils]` |
| `allowProxy` | Serve via `/etc.clientlibs/` (cacheable) | `true` |
| `channels` | Target channels | `[mobile, desktop]` |

### Including ClientLibs in HTL
```html
<!-- CSS only -->
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html">
    <sly data-sly-call="${clientlib.css @ categories='myapp.base'}"/>
</sly>

<!-- JS only -->
<sly data-sly-call="${clientlib.js @ categories='myapp.base'}"/>

<!-- Both CSS and JS -->
<sly data-sly-call="${clientlib.all @ categories='myapp.base'}"/>

<!-- Multiple categories -->
<sly data-sly-call="${clientlib.css @ categories=['myapp.base', 'myapp.hero']}"/>
```

### Dependencies vs Embed

| Feature | Dependencies | Embed |
|---------|-------------|-------|
| HTTP requests | Separate request per lib | Merged into one file |
| Loading | Loaded before dependent | Inline merged |
| Caching | Cached separately | Cached as one bundle |
| Use case | Shared libs (jQuery) | Component-specific bundles |

### ClientLib Debugging
```
# View all registered categories
http://localhost:4502/libs/granite/ui/content/dumplibs.html

# Debug mode (unminified, separate files)
?debugClientLibs=true

# Rebuild clientlibs
http://localhost:4502/libs/granite/ui/content/dumplibs.rebuild.html
```

---

## Frontend Module (ui.frontend)

Modern AEM projects use a Webpack/npm-based frontend build:

### Build Pipeline
```
ui.frontend/src/
├── main/
│   ├── webpack/          → Webpack configs
│   └── site/
│       ├── main.ts       → JS entry point
│       └── main.scss     → CSS entry point
└── components/
    ├── hero/
    │   ├── hero.ts
    │   └── hero.scss
    └── navigation/
        ├── navigation.ts
        └── navigation.scss
```

### Build Flow
```
npm run build (in ui.frontend)
     │
     ▼
Webpack bundles + minifies
     │
     ▼
Output → ui.apps/src/main/content/.../clientlib-site/
     │
     ▼
Maven packages into AEM content package
     │
     ▼
Deployed as ClientLib in AEM
```

### aem-clientlib-generator
Config in `clientlib.config.js`:
```javascript
module.exports = {
    context: __dirname,
    clientLibRoot: "../ui.apps/src/main/content/jcr_root/apps/myapp/clientlibs",
    libs: {
        name: "clientlib-site",
        allowProxy: true,
        categories: ["myapp.site"],
        serializationFormat: "xml",
        assets: {
            js: ["dist/**/*.js"],
            css: ["dist/**/*.css"]
        }
    }
};
```

---

## Multi-Site Manager (MSM)

MSM enables managing multiple related sites from a single source.

### Key Concepts

| Term | Definition |
|------|-----------|
| **Blueprint** | Source/master site (template) |
| **Live Copy** | Site created from blueprint (linked) |
| **Rollout** | Push changes from blueprint → live copies |
| **Synchronization** | Pulling changes into a live copy |
| **Inheritance** | Live copy inherits content from blueprint |
| **Detach** | Break inheritance (live copy becomes independent) |

### How MSM Works
```
Blueprint: /content/mysite/en (Master)
     │
     ├── Rollout ──→ /content/mysite/fr (French live copy)
     ├── Rollout ──→ /content/mysite/de (German live copy)
     └── Rollout ──→ /content/mysite/es (Spanish live copy)
```

### Rollout Configurations
Define WHEN and HOW content is synchronized:

| Config | Trigger | Action |
|--------|---------|--------|
| Standard | On rollout | Update content + subpages |
| Push on modify | On blueprint save | Auto-push changes |
| Activate on publish | On blueprint activate | Activate live copy |

### Inheritance Behavior
- **Inherited** — Content syncs from blueprint (green lock icon)
- **Overridden** — Locally modified (broken inheritance for that field)
- **Cancelled** — Inheritance fully broken for the component
- **Suspended** — Temporarily paused (can resume)
- **Detached** — Permanently disconnected

### MSM Java API
```java
@Reference
private LiveRelationshipManager liveRelManager;

// Check if a resource is a live copy
LiveRelationship relationship = liveRelManager.getLiveRelationship(resource, false);
if (relationship != null) {
    String blueprintPath = relationship.getSyncPath();
    boolean isDeep = relationship.isDeep();
}

// Rollout programmatically
RolloutManager rolloutManager;
rolloutManager.rollout(blueprintResource, targetPath, true);
```

---

## Internationalization (i18n)

### Translation Dictionary
Location: `/apps/myapp/i18n/`
```
/apps/myapp/i18n/
├── en.json    → {"greeting": "Hello", "button.submit": "Submit"}
├── fr.json    → {"greeting": "Bonjour", "button.submit": "Soumettre"}
└── de.json    → {"greeting": "Hallo", "button.submit": "Einreichen"}
```

### Using i18n in HTL
```html
<!-- Basic translation -->
<p>${'greeting' @ i18n}</p>

<!-- With locale hint -->
<p>${'button.submit' @ i18n, locale='fr'}</p>

<!-- In Sling Model -->
<!-- Use I18n API with request locale -->
```

### i18n in Java
```java
@Self
private SlingHttpServletRequest request;

@PostConstruct
protected void init() {
    I18n i18n = new I18n(request);
    String greeting = i18n.get("greeting");
    String submit = i18n.get("button.submit");
}
```

---

## Language Manager & Translation

### Translation Integration Framework
1. Connect AEM to translation provider (Adobe/SDL/custom)
2. Create translation project from language master
3. Submit for translation
4. Review and approve translations
5. Publish translated content

### Language Copy Structure
```
/content/mysite/
├── en/           → Language master
│   ├── home
│   └── about
├── fr/           → French (translated)
│   ├── home
│   └── about
└── de/           → German (translated)
    ├── home
    └── about
```

---

## MCQs — ClientLibs, Frontend & MSM

### Q1: What property merges one ClientLib into another (single file output)?
- A) dependencies
- B) embed ✅ **CORRECT**
- C) include
- D) merge

> **Explanation:** `embed` merges another lib's files into this lib's output — one HTTP request.

### Q2: What does `allowProxy=true` do for a ClientLib?
- A) Allows CDN caching
- B) Serves via /etc.clientlibs/ path (dispatcher cacheable) ✅ **CORRECT**
- C) Enables proxy servers
- D) Allows cross-origin access

> **Explanation:** `allowProxy` exposes the lib at `/etc.clientlibs/...` instead of `/apps/...` which Dispatcher blocks.

### Q3: In MSM, what is the process of pushing blueprint changes to live copies?
- A) Replication
- B) Synchronization
- C) Rollout ✅ **CORRECT**
- D) Publishing

> **Explanation:** Rollout pushes changes from blueprint to its live copies based on rollout configurations.

### Q4: What happens when you "detach" a live copy page?
- A) It gets deleted
- B) Inheritance is permanently broken ✅ **CORRECT**
- C) It moves to a new location
- D) It merges with the blueprint

> **Explanation:** Detaching permanently breaks the live relationship. The page becomes fully independent.

### Q5: Which file lists the CSS files to include in a ClientLib?
- A) styles.txt
- B) css.txt ✅ **CORRECT**
- C) includes.css
- D) manifest.json

> **Explanation:** `css.txt` lists CSS files in order; `js.txt` lists JavaScript files.

### Q6: How do you include a ClientLib's CSS in an HTL template?
- A) `<link href="clientlib.css"/>`
- B) `data-sly-call="${clientlib.css @ categories='myapp.base'}"` ✅ **CORRECT**
- C) `<sly data-sly-include="clientlib"/>`
- D) `${clientlib.load('css')}`

> **Explanation:** Use the clientlib template helper with `data-sly-call` and specify the category.

### Q7: What does `?debugClientLibs=true` do?
- A) Shows errors in console
- B) Loads unminified files separately ✅ **CORRECT**
- C) Enables source maps
- D) Logs load times

> **Explanation:** Debug mode skips minification and serves individual files for easier debugging.

### Q8: In i18n, how do you translate a string in HTL?
- A) `${translate('key')}`
- B) `${'key' @ i18n}` ✅ **CORRECT**
- C) `${i18n.get('key')}`
- D) `${'key' @ locale}`

> **Explanation:** HTL uses the `@ i18n` expression option to look up the translation for the current locale.
