# AEM Interview Preparation — Part 8: AEM as a Cloud Service, Security & Performance

---

## AEM as a Cloud Service (AEMaaCS)

### Key Differences from AEM 6.5

| Aspect | AEM 6.5 (On-Premise/AMS) | AEM as a Cloud Service |
|--------|--------------------------|----------------------|
| Infrastructure | Self-managed / Adobe Managed | Fully cloud-native (Azure) |
| Deployment | Package Manager / CRX | Cloud Manager CI/CD only |
| Scaling | Manual | Auto-scaling |
| Updates | Manual service packs | Continuous, automatic |
| Repository | MongoMK / TarMK | Cloud-native (Segment) |
| Mutable content | `/apps` writable at runtime | `/apps` is IMMUTABLE |
| OSGi configs | Package Manager | Git repo → Cloud Manager |
| Custom indexes | Install via package | Git repo → Cloud Manager |
| Workflows | Full workflow engine | Asset microservices |

### AEMaaCS Architecture
```
┌─────────────────────────────────────────┐
│             Cloud Manager                │
│  (CI/CD Pipeline, Environments)          │
├─────────────────────────────────────────┤
│     Author Tier    │    Publish Tier     │
│  (Auto-scaled)     │   (Auto-scaled)     │
├─────────────────────────────────────────┤
│              CDN (Fastly)                │
├─────────────────────────────────────────┤
│        Asset Compute Service             │
│  (Serverless asset processing)           │
└─────────────────────────────────────────┘
```

### Cloud Manager Pipeline Types
1. **Code Quality** — Runs tests without deploying
2. **Full Stack** — Deploys all code + configs
3. **Frontend** — Deploys only ui.frontend
4. **Config** — Deploys dispatcher/CDN configs
5. **Web Tier** — Dispatcher only

### Key AEMaaCS Constraints
- No Package Manager in production
- No CRXDE in production
- No direct JCR access
- `/apps` and `/libs` are read-only after deployment
- Only `.cfg.json` for OSGi configs
- No custom workflow process steps modifying DAM (use Asset Compute)
- Sling jobs run on leader instance only

---

## AEM Security

### User and Group Management

**System Users (for services):**
- Created in `/home/users/system/`
- Used with service user mappings
- No interactive login

**Service User Mapping:**
```json
{
    "user.mapping": [
        "com.myapp.core:myservice=[myapp-service-user]"
    ]
}
```

### Access Control Lists (ACLs)

```
- Allow read on /content/mysite       → content-authors
- Allow write on /content/mysite      → content-authors
- Deny delete on /content/mysite/en   → content-authors (protect structure)
- Allow replicate on /content/mysite  → content-publishers
```

### Security Best Practices

1. **Input Validation**
```java
// Always validate external input
String userInput = request.getParameter("query");
if (userInput == null || userInput.length() > 200) {
    response.setStatus(400);
    return;
}
// Sanitize for XSS
String safe = XSSAPI.encodeForHTML(userInput);
```

2. **CSRF Protection**
```java
// AEM includes CSRF token validation for POST requests
// Frontend must include the token:
// $.ajax({ headers: {"CSRF-Token": token} })
```

3. **Dispatcher Security Filters**
```
# Block sensitive paths
/0010 { /type "deny" /url "/crx/*" }
/0011 { /type "deny" /url "/system/*" }
/0012 { /type "deny" /url "*.infinity.json" }
/0013 { /type "deny" /url "*.tidy.json" }
/0014 { /type "deny" /url "*/jcr:content.*" }
/0015 { /type "deny" /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|[0-9-]+|jcr:content)' }
```

4. **Closed User Group (CUG)**
- Restricts read access to specific pages
- Only members of the CUG can view the content
- Configured on page properties → Permissions tab

5. **SSL/TLS**
- Enforce HTTPS via Dispatcher
- Use AEM's SSL wizard for author instances
- Configure HSTS headers

---

## Performance Optimization

### Caching Strategy (4 Layers)

```
Browser Cache (TTL headers)
     │
CDN Cache (Fastly/Akamai)
     │
Dispatcher Cache (filesystem)
     │
AEM Internal Cache (in-memory)
```

### AEM Performance Best Practices

**1. Query Optimization**
```java
// BAD: No index, traversal query
String query = "SELECT * FROM [nt:base] WHERE [myProp] = 'value'";

// GOOD: Specific node type + path restriction + indexed property
String query = "SELECT * FROM [cq:PageContent] " +
    "WHERE ISDESCENDANTNODE('/content/mysite') " +
    "AND [sling:resourceType] = 'myapp/components/article'";
```

**2. Resource Resolver Management**
```java
// BAD: Leaking resource resolver
ResourceResolver resolver = factory.getServiceResourceResolver(params);
// ... forgot to close

// GOOD: try-with-resources
try (ResourceResolver resolver = factory.getServiceResourceResolver(params)) {
    // use resolver
} // auto-closed
```

**3. Sling Model Efficiency**
```java
@Model(adaptables = Resource.class,
       defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)
public class EfficientModel {
    
    // Lazy initialization for expensive operations
    private List<Article> articles;
    
    @OSGiService
    private QueryBuilder queryBuilder;
    
    @Self
    private Resource resource;
    
    public List<Article> getArticles() {
        if (articles == null) {
            articles = loadArticles(); // Only load when needed
        }
        return articles;
    }
}
```

**4. ClientLib Optimization**
- Minify CSS/JS in production
- Use `allowProxy=true` for cacheable delivery
- Combine related scripts into single categories
- Defer non-critical JS: `async` or `defer`

**5. Dispatcher Cache Headers**
```apache
# Cache static assets aggressively
<LocationMatch "^/content/.*\.(css|js|png|jpg|gif|svg|woff2?)$">
    Header set Cache-Control "max-age=2592000, public"
</LocationMatch>

# Short cache for HTML pages
<LocationMatch "^/content/.*\.html$">
    Header set Cache-Control "max-age=300, public"
</LocationMatch>
```

---

### Common Performance Issues

| Issue | Symptom | Fix |
|-------|---------|-----|
| Missing index | Slow queries, traversal warnings | Create Oak index |
| Unclosed resolvers | Memory leak, session exhaustion | try-with-resources |
| Large payloads | Slow page load | Lazy loading, pagination |
| No dispatcher cache | Every request hits AEM | Configure cache rules |
| Synchronous requests | Blocked rendering | Async/deferred loading |
| Too many nodes | Slow author UI | Flat structure, pagination |

---

## AEM Monitoring & Debugging

### Key Health Checks
- `http://localhost:4502/system/console/healthcheck?tags=*`
- Bundle status, replication queues, disk space, query performance

### Useful Debug URLs
| URL | Purpose |
|-----|---------|
| `/system/console/bundles` | Bundle status |
| `/system/console/components` | DS Components |
| `/system/console/configMgr` | OSGi configs |
| `/system/console/slinglog` | Log configuration |
| `/system/console/jmx` | JMX MBeans |
| `/libs/granite/operations/content/diagnosis/tool.html/granite_queryperformance` | Slow queries |
| `/system/console/status-slingrequests` | Request stats |

### Log Configuration
```json
{
    "org.apache.sling.commons.log.file": "logs/myapp.log",
    "org.apache.sling.commons.log.level": "DEBUG",
    "org.apache.sling.commons.log.names": [
        "com.myapp.core"
    ]
}
```

---

## MCQs — Cloud, Security & Performance

### Q1: In AEM as a Cloud Service, how is code deployed?
- A) Package Manager
- B) CRXDE Lite
- C) Cloud Manager CI/CD pipeline ✅ **CORRECT**
- D) FTP upload

> **Explanation:** AEMaaCS only allows deployment through Cloud Manager pipelines from a Git repository.

### Q2: What is immutable in AEM as a Cloud Service?
- A) /content
- B) /apps (after deployment) ✅ **CORRECT**
- C) /home/users
- D) /var

> **Explanation:** `/apps` and `/libs` are immutable at runtime. Changes require a new deployment via Cloud Manager.

### Q3: What is the primary purpose of a Closed User Group (CUG)?
- A) Rate limiting
- B) Restrict read access to specific content ✅ **CORRECT**
- C) Limit API calls
- D) Block IP addresses

> **Explanation:** CUG restricts page read access to specific user groups — useful for gated content.

### Q4: Which caching layer is closest to the user?
- A) AEM internal cache
- B) Dispatcher cache
- C) Browser/CDN cache ✅ **CORRECT**
- D) JCR cache

> **Explanation:** Browser cache and CDN are closest to the user, reducing latency most effectively.

### Q5: What is the main danger of not closing a service ResourceResolver?
- A) Slow queries
- B) Session/memory leak ✅ **CORRECT**
- C) Security vulnerability
- D) Cache corruption

> **Explanation:** Unclosed service resolvers hold JCR sessions open, causing memory leaks and session exhaustion.

### Q6: How does AEMaaCS handle DAM asset processing?
- A) DAM Update Asset workflow
- B) Asset Compute microservices ✅ **CORRECT**
- C) Manual processing
- D) Third-party plugins

> **Explanation:** AEMaaCS uses serverless Asset Compute workers instead of traditional workflow-based processing.

### Q7: What query practice causes traversal warnings?
- A) Using indexes
- B) Queries without path restriction or matching index ✅ **CORRECT**
- C) Using QueryBuilder
- D) JCR-SQL2 syntax

> **Explanation:** Queries without indexes force full repository traversal, which is limited and logged as warnings.

### Q8: For a service to access JCR content, what must be configured?
- A) Admin password
- B) Service user mapping ✅ **CORRECT**
- C) Replication agent
- D) Workflow launcher

> **Explanation:** Services need a service user mapping to get a ResourceResolver with appropriate permissions.
