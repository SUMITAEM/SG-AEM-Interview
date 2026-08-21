# AEM Interview Preparation — Part 4: JCR, Node Types & Queries

---

## Java Content Repository (JCR)

JCR (JSR-283) is a hierarchical content store. AEM uses **Apache Jackrabbit Oak** as its implementation.

### Core Concepts
- **Workspace** — A view of the entire repository tree
- **Node** — An element in the tree (like a folder/file)
- **Property** — Data attached to a node (key-value)
- **Node Type** — Schema defining allowed children and properties

---

## Node Types

### Common AEM Node Types

| Node Type | Purpose |
|-----------|---------|
| `nt:unstructured` | Flexible, no constraints (most common) |
| `cq:Page` | An AEM page |
| `cq:PageContent` | Page content node (`jcr:content`) |
| `nt:folder` | Basic folder |
| `sling:Folder` | Sling-managed folder |
| `sling:OrderedFolder` | Ordered folder (maintains child order) |
| `dam:Asset` | DAM asset |
| `dam:AssetContent` | Asset metadata node |
| `cq:Component` | Component definition |
| `cq:Template` | Template definition |
| `rep:User` | User node |

### Page Structure in JCR
```
/content/mysite/en/home          [cq:Page]
  └── jcr:content                [cq:PageContent]
       ├── jcr:title = "Home"
       ├── sling:resourceType = "myapp/components/page"
       ├── cq:template = "/conf/myapp/settings/wcm/templates/page"
       └── root                  [nt:unstructured]
            └── container        [nt:unstructured]
                 ├── hero        [nt:unstructured]
                 │    ├── sling:resourceType = "myapp/components/hero"
                 │    ├── title = "Welcome"
                 │    └── imagePath = "/content/dam/hero.jpg"
                 └── text        [nt:unstructured]
                      ├── sling:resourceType = "myapp/components/text"
                      └── text = "<p>Hello World</p>"
```

---

## JCR Properties

### Property Types

| Type | Java Type | Example |
|------|-----------|---------|
| STRING | String | "Hello" |
| LONG | Long | 42 |
| DOUBLE | Double | 3.14 |
| BOOLEAN | Boolean | true |
| DATE | Calendar | 2024-01-15T10:30:00 |
| BINARY | Binary | File data |
| PATH | String | /content/dam/image.jpg |
| REFERENCE | String | UUID reference to another node |
| NAME | String | Qualified name |

### Important System Properties

| Property | Purpose |
|----------|---------|
| `jcr:primaryType` | Node type |
| `jcr:mixinTypes` | Additional types mixed in |
| `jcr:title` | Display title |
| `jcr:description` | Description |
| `jcr:created` | Creation date |
| `jcr:createdBy` | Creator user |
| `cq:lastModified` | Last modification date |
| `cq:lastModifiedBy` | Last modifier |
| `sling:resourceType` | Component type for rendering |
| `sling:resourceSuperType` | Parent component type |

---

## JCR Queries

### Query Languages in AEM

1. **JCR-SQL2** (recommended)
2. **XPath** (legacy but still widely used)
3. **QueryBuilder** (AEM-specific API)

### JCR-SQL2 Examples

```sql
-- Find all pages under a path
SELECT * FROM [cq:Page] 
WHERE ISDESCENDANTNODE('/content/mysite')

-- Find pages with specific property
SELECT * FROM [nt:unstructured] AS content
WHERE ISDESCENDANTNODE(content, '/content/mysite')
AND content.[sling:resourceType] = 'myapp/components/article'
AND content.[jcr:title] LIKE '%AEM%'

-- Full-text search
SELECT * FROM [cq:PageContent]
WHERE ISDESCENDANTNODE('/content/mysite')
AND CONTAINS(*, 'search term')

-- Date range query
SELECT * FROM [cq:PageContent] AS c
WHERE ISDESCENDANTNODE(c, '/content/mysite')
AND c.[cq:lastModified] > CAST('2024-01-01T00:00:00.000Z' AS DATE)

-- Order results
SELECT * FROM [cq:PageContent] AS c
WHERE ISDESCENDANTNODE(c, '/content/mysite')
ORDER BY c.[cq:lastModified] DESC
```

### XPath Examples

```xpath
/jcr:root/content/mysite//element(*, cq:Page)

/jcr:root/content/mysite//*[@sling:resourceType='myapp/components/article']

/jcr:root/content/mysite//element(*, cq:PageContent)
    [jcr:contains(., 'search term')]
```

---

## QueryBuilder API

AEM's high-level query API with predicate-based syntax:

### HTTP Interface
```
GET /bin/querybuilder.json?
    path=/content/mysite
    &type=cq:Page
    &property=jcr:content/sling:resourceType
    &property.value=myapp/components/article
    &orderby=@jcr:content/cq:lastModified
    &orderby.sort=desc
    &p.limit=10
```

### Java API
```java
@Reference
private QueryBuilder queryBuilder;

public List<Resource> findArticles(ResourceResolver resolver) {
    Map<String, String> predicates = new HashMap<>();
    predicates.put("path", "/content/mysite");
    predicates.put("type", "cq:Page");
    predicates.put("property", "jcr:content/sling:resourceType");
    predicates.put("property.value", "myapp/components/article");
    predicates.put("orderby", "@jcr:content/cq:lastModified");
    predicates.put("orderby.sort", "desc");
    predicates.put("p.limit", "10");
    
    Query query = queryBuilder.createQuery(
        PredicateGroup.create(predicates),
        resolver.adaptTo(Session.class)
    );
    
    SearchResult result = query.getResult();
    List<Resource> resources = new ArrayList<>();
    for (Hit hit : result.getHits()) {
        resources.add(hit.getResource());
    }
    return resources;
}
```

### Common QueryBuilder Predicates

| Predicate | Purpose | Example |
|-----------|---------|---------|
| `path` | Search root | `/content/mysite` |
| `type` | Node type | `cq:Page` |
| `property` | Property match | `jcr:content/jcr:title` |
| `fulltext` | Full-text search | `search term` |
| `daterange` | Date filtering | `.lowerBound=2024-01-01` |
| `orderby` | Sorting | `@jcr:content/cq:lastModified` |
| `p.limit` | Max results | `10` (-1 for unlimited) |
| `p.offset` | Pagination offset | `20` |
| `group` | Grouped conditions | `1_group.property=...` |
| `nodename` | Node name match | `article*` |
| `hasPermission` | Permission check | `jcr:write` |

---

## Oak Indexes

### Why Indexes Matter
- Without an index, queries do **traversal** (scan entire tree) — VERY SLOW
- Production AEM limits traversal to 100,000 nodes
- Custom queries need custom indexes

### Index Types

| Type | Use Case |
|------|----------|
| **Property Index** | Exact property match queries |
| **Lucene/Fulltext** | Full-text search, sorting, facets |
| **Ordered Index** | Range queries, sorting (deprecated in Oak 1.6+) |

### Creating a Custom Lucene Index
```
/oak:index/myCustomIndex          [oak:QueryIndexDefinition]
  ├── type = "lucene"
  ├── compatVersion = 2
  ├── async = "async"
  ├── evaluatePathRestrictions = true
  ├── includedPaths = ["/content/mysite"]
  └── indexRules                   [nt:unstructured]
       └── cq:PageContent          [nt:unstructured]
            └── properties         [nt:unstructured]
                 ├── title         [nt:unstructured]
                 │    ├── name = "jcr:title"
                 │    └── propertyIndex = true
                 └── resourceType  [nt:unstructured]
                      ├── name = "sling:resourceType"
                      └── propertyIndex = true
```

---

## CRXDE Lite & Content Operations

Access at: `http://localhost:4502/crx/de/index.jsp`

### JCR API in Java
```java
Session session = resolver.adaptTo(Session.class);

// Read a node
Node node = session.getNode("/content/mysite/en/home/jcr:content");
String title = node.getProperty("jcr:title").getString();

// Create node
Node parent = session.getNode("/content/mysite/en");
Node newPage = parent.addNode("new-page", "cq:Page");
Node content = newPage.addNode("jcr:content", "cq:PageContent");
content.setProperty("jcr:title", "New Page");
session.save();

// Delete node
session.getNode("/content/mysite/en/old-page").remove();
session.save();
```

---

## MCQs — JCR & Queries

### Q1: What node type is used for AEM pages?
- A) nt:unstructured
- B) cq:Page ✅ **CORRECT**
- C) sling:Page
- D) jcr:Page

> **Explanation:** `cq:Page` is the node type for AEM pages. The content is in the child `jcr:content` node.

### Q2: Which query language is recommended for new AEM development?
- A) XPath
- B) JCR-SQL
- C) JCR-SQL2 ✅ **CORRECT**
- D) HQL

> **Explanation:** JCR-SQL2 is the recommended standard. XPath works but is considered legacy.

### Q3: What happens if you run a query without a matching Oak index in production?
- A) Query runs slowly but completes
- B) Query fails with traversal limit exception ✅ **CORRECT**
- C) AEM creates an index automatically
- D) Query returns empty results

> **Explanation:** Production AEM limits traversal. Without an index, queries exceeding the limit throw an exception.

### Q4: In QueryBuilder, what does `p.limit=-1` do?
- A) Returns zero results
- B) Returns one result
- C) Returns all results (no limit) ✅ **CORRECT**
- D) Throws an error

> **Explanation:** `-1` removes the result limit. Use cautiously — it can return thousands of results.

### Q5: Where does page content (title, components) live in JCR?
- A) Directly on the cq:Page node
- B) In the jcr:content child node ✅ **CORRECT**
- C) In a separate /content/data node
- D) In the properties table

> **Explanation:** The `cq:Page` node has a `jcr:content` child (type `cq:PageContent`) that holds all the content.

### Q6: Which function checks if a node is below a given path in JCR-SQL2?
- A) CHILDOF()
- B) BELOW()
- C) ISDESCENDANTNODE() ✅ **CORRECT**
- D) ISUNDER()

> **Explanation:** `ISDESCENDANTNODE('/path')` filters to nodes that are descendants of the given path.

### Q7: What is the recommended index type for full-text search in AEM?
- A) Property Index
- B) Ordered Index
- C) Lucene Index ✅ **CORRECT**
- D) B-Tree Index

> **Explanation:** Lucene indexes support full-text search, sorting, facets, and complex queries.

### Q8: What property determines which component renders a content node?
- A) jcr:primaryType
- B) cq:template
- C) sling:resourceType ✅ **CORRECT**
- D) jcr:mixinTypes

> **Explanation:** `sling:resourceType` tells Sling which component/script to use for rendering.
