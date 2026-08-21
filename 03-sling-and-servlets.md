# AEM Interview Preparation — Part 3: Apache Sling & Servlets

---

## Apache Sling Overview

Sling is a REST-based web framework that maps URLs to content resources (JCR nodes), not to servlets directly.

### Sling Request Processing Flow

```
HTTP Request
     │
     ▼
┌─────────────────────┐
│ 1. URL Decomposition │ → Break URL into parts
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 2. Resource Resolution│ → Find JCR node
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 3. Servlet/Script    │ → Find rendering script
│    Resolution        │    based on resourceType
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 4. Request Processing│ → Filters → Servlet → Response
└─────────────────────┘
```

### Sling URL Anatomy

```
/content/site/page.selector1.selector2.html/suffix/path?key=value

Parts:
- Resource Path:  /content/site/page
- Selectors:      selector1.selector2
- Extension:      html
- Suffix:         /suffix/path
- Query String:   key=value
```

### Script Resolution Order

For resource type `myapp/components/hero` with selector `mobile` and extension `html`:

```
1. hero/mobile.html          (selector match)
2. hero/hero.html            (name match)
3. hero/html.html            (extension match)
4. hero/GET.html             (method match)
5. sling:resourceSuperType   (inheritance)
```

---

## Sling Servlets

Two types of registration:

### 1. Path-Based Servlet
```java
@Component(service = Servlet.class, property = {
    "sling.servlet.paths=/bin/myapp/data",
    "sling.servlet.methods=GET"
})
public class DataServlet extends SlingSafeMethodsServlet {
    
    @Override
    protected void doGet(SlingHttpServletRequest request,
                         SlingHttpServletResponse response)
            throws IOException {
        
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        
        JsonObject json = new JsonObject();
        json.addProperty("status", "success");
        json.addProperty("message", "Hello from servlet");
        
        response.getWriter().write(json.toString());
    }
}
```

### 2. Resource-Type Based Servlet
```java
@Component(service = Servlet.class, property = {
    "sling.servlet.resourceTypes=myapp/components/search",
    "sling.servlet.methods=POST",
    "sling.servlet.selectors=submit",
    "sling.servlet.extensions=json"
})
public class SearchServlet extends SlingAllMethodsServlet {
    
    @Override
    protected void doPost(SlingHttpServletRequest request,
                          SlingHttpServletResponse response)
            throws IOException {
        
        String query = request.getParameter("q");
        // process search...
    }
}
```

### Path-Based vs Resource-Type Based

| Aspect | Path-Based | Resource-Type Based |
|--------|-----------|-------------------|
| Registration | `sling.servlet.paths` | `sling.servlet.resourceTypes` |
| URL | Fixed path (e.g., /bin/...) | Follows resource URL |
| Security | Needs explicit ACL | Inherits resource permissions |
| Use Case | API endpoints | Component-specific actions |
| Sling features | No selectors/suffix | Full Sling URL features |
| Recommendation | Sparingly | Preferred |

---

## Sling Resource Resolver

The gateway to accessing JCR content programmatically:

### Getting a Resource Resolver

**In a Servlet (from request):**
```java
ResourceResolver resolver = request.getResourceResolver();
```

**As a Service (service user):**
```java
@Reference
private ResourceResolverFactory resolverFactory;

Map<String, Object> params = new HashMap<>();
params.put(ResourceResolverFactory.SUBSERVICE, "my-service-user");
ResourceResolver resolver = resolverFactory.getServiceResourceResolver(params);
// ALWAYS close in finally block!
```

### Service User Mapping
File: `org.apache.sling.serviceusermapping.impl.ServiceUserMapperImpl.amended-myservice.cfg.json`
```json
{
    "user.mapping": [
        "com.myapp.core:my-service-user=myapp-service-user"
    ]
}
```

### Common Resource Resolver Operations
```java
// Get a resource
Resource resource = resolver.getResource("/content/mysite/en/home");

// Adapt to Node or ValueMap
ValueMap properties = resource.getValueMap();
String title = properties.get("jcr:title", String.class);

// List children
Iterator<Resource> children = resource.listChildren();

// Create/modify content
resolver.create(parentResource, "newNode", properties);
resolver.commit(); // save changes

// Query
Iterator<Resource> results = resolver.findResources(
    "SELECT * FROM [cq:Page] WHERE ISDESCENDANTNODE('/content/mysite')",
    "JCR-SQL2"
);
```

---

## Sling Models

Sling Models map JCR content to Java objects using annotations:

### Basic Sling Model
```java
@Model(
    adaptables = {Resource.class, SlingHttpServletRequest.class},
    adapters = HeroModel.class,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL
)
public class HeroModelImpl implements HeroModel {
    
    @ValueMapValue
    private String title;
    
    @ValueMapValue
    @Default(values = "Learn More")
    private String ctaText;
    
    @ValueMapValue
    private String imagePath;
    
    @Inject
    @Source("child-resource")
    private Resource childResource;
    
    @OSGiService
    private ExternalizerService externalizer;
    
    @Self
    private SlingHttpServletRequest request;
    
    @PostConstruct
    protected void init() {
        // Post-construction logic
        if (imagePath == null) {
            imagePath = "/content/dam/default-hero.jpg";
        }
    }
    
    @Override
    public String getTitle() {
        return title;
    }
    
    @Override
    public String getCtaText() {
        return ctaText;
    }
}
```

### Injection Annotations

| Annotation | Source | Example |
|-----------|--------|---------|
| `@ValueMapValue` | Resource properties | `jcr:title`, custom props |
| `@ChildResource` | Child node as Resource | Multifield items |
| `@OSGiService` | OSGi service registry | Custom services |
| `@Self` | The adaptable itself | Request, Resource |
| `@SlingObject` | Sling objects | ResourceResolver, Response |
| `@RequestAttribute` | Request attributes | Passed from HTL |
| `@ScriptVariable` | Script bindings | currentPage, currentNode |

### Model Exporters (JSON)
```java
@Model(
    adaptables = Resource.class,
    adapters = {ArticleModel.class, ArticleExporter.class},
    resourceType = "myapp/components/article"
)
@Exporter(name = "jackson", extensions = "json")
public class ArticleModelImpl implements ArticleModel {
    
    @ValueMapValue
    private String headline;
    
    @ValueMapValue
    private String body;
}
```
Access via: `/content/mysite/article.model.json`

---

## Sling Filters

```java
@Component(
    service = Filter.class,
    property = {
        "sling.filter.scope=REQUEST",
        "service.ranking:Integer=100"
    }
)
public class LoggingFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        long start = System.currentTimeMillis();
        chain.doFilter(request, response);
        long duration = System.currentTimeMillis() - start;
        LOG.debug("Request took {}ms", duration);
    }
}
```

**Filter Scopes:**
- `REQUEST` — Incoming requests
- `COMPONENT` — Include calls (within a request)
- `ERROR` — Error handling
- `INCLUDE` — RequestDispatcher includes
- `FORWARD` — RequestDispatcher forwards

---

## MCQs — Sling & Servlets

### Q1: How does Sling determine which script renders a resource?
- A) By URL path matching
- B) By sling:resourceType property of the node ✅ **CORRECT**
- C) By file extension only
- D) By servlet-mapping in web.xml

> **Explanation:** Sling reads the `sling:resourceType` from the JCR node and finds the matching script/servlet.

### Q2: Which servlet class should you extend for a POST-handling servlet?
- A) HttpServlet
- B) SlingSafeMethodsServlet
- C) SlingAllMethodsServlet ✅ **CORRECT**
- D) GenericServlet

> **Explanation:** `SlingAllMethodsServlet` supports GET, POST, PUT, DELETE. `SlingSafeMethodsServlet` only supports GET/HEAD.

### Q3: What is the correct way to get a service-based ResourceResolver?
- A) new ResourceResolver()
- B) resolverFactory.getServiceResourceResolver(params) ✅ **CORRECT**
- C) resolverFactory.getAdministrativeResourceResolver()
- D) ResourceResolver.create()

> **Explanation:** `getServiceResourceResolver` with a service user mapping is the secure modern approach. `getAdministrativeResourceResolver` is deprecated.

### Q4: In Sling Models, which annotation injects an OSGi service?
- A) @Reference
- B) @Inject
- C) @OSGiService ✅ **CORRECT**
- D) @Service

> **Explanation:** In Sling Models, use `@OSGiService`. `@Reference` is for OSGi Declarative Services (not models).

### Q5: What is a Sling selector used for?
- A) Filtering database queries
- B) Choosing alternate renditions of the same resource ✅ **CORRECT**
- C) Selecting JCR nodes
- D) User authentication

> **Explanation:** Selectors allow different views of the same content (e.g., `page.mobile.html` vs `page.html`).

### Q6: Which injection strategy means all injections are optional by default?
- A) DefaultInjectionStrategy.REQUIRED
- B) DefaultInjectionStrategy.OPTIONAL ✅ **CORRECT**
- C) DefaultInjectionStrategy.NULLABLE
- D) DefaultInjectionStrategy.LAZY

> **Explanation:** `OPTIONAL` means fields won't cause model adaptation failure if they can't be injected.

### Q7: What must you always do with a service ResourceResolver?
- A) Cache it for reuse
- B) Store it as a field
- C) Close it in a finally block ✅ **CORRECT**
- D) Commit it before closing

> **Explanation:** Service resolvers open JCR sessions — you must close them to prevent resource leaks.

### Q8: Which property registers a resource-type based servlet?
- A) sling.servlet.paths
- B) sling.servlet.resourceTypes ✅ **CORRECT**
- C) sling.servlet.types
- D) sling.servlet.resource

> **Explanation:** `sling.servlet.resourceTypes` registers a servlet that responds when its resource type is requested.
