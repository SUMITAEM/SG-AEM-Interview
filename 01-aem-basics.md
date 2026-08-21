# AEM Interview Preparation — Part 1: Basics & Architecture

---

## What is AEM?

Adobe Experience Manager (AEM) is an enterprise-grade Content Management System (CMS) built on:
- **Apache Sling** (web framework)
- **Apache Jackrabbit Oak** (content repository - JCR)
- **OSGi** (modular Java framework - Apache Felix)

AEM enables organizations to create, manage, and deliver digital content across web, mobile, and IoT channels.

---

## AEM Architecture Layers

```
┌─────────────────────────────────────┐
│         AEM Application             │
│  (Components, Templates, Workflows) │
├─────────────────────────────────────┤
│         Apache Sling                │
│  (Request Processing, Scripting)    │
├─────────────────────────────────────┤
│         JCR (Jackrabbit Oak)        │
│  (Content Repository, Node Storage) │
├─────────────────────────────────────┤
│         OSGi (Apache Felix)         │
│  (Module System, Service Registry)  │
├─────────────────────────────────────┤
│         JVM (Java 8/11)             │
└─────────────────────────────────────┘
```

---

## Key Concepts

### 1. Content Repository (JCR)
- Everything in AEM is stored as **nodes** and **properties** in a tree structure
- Root node: `/`
- Content lives under `/content`
- Applications under `/apps`
- Libraries under `/libs`
- DAM assets under `/content/dam`

### 2. Sling Resource Resolution
Sling maps HTTP requests to content resources:
```
URL: /content/mysite/en/home.html
→ Resource: /content/mysite/en/home
→ Resource Type: mysite/components/page/homepage
→ Script: /apps/mysite/components/page/homepage/homepage.html
```

**Sling URL Decomposition:**
```
/content/mysite/en/home.selector.html/suffix?param=value
│                    │        │      │    │       │
└─ Resource Path     │        │      │    │       └─ Query String
                     └─ Node  │      │    └─ Suffix
                              └─ Selector  └─ Extension
```

### 3. OSGi (Open Services Gateway initiative)
- Modular system for Java
- Bundles = JAR files with metadata
- Services = interfaces registered in the service registry
- Life cycle: INSTALLED → RESOLVED → STARTING → ACTIVE → STOPPING

### 4. Run Modes
- `author` — Content authoring instance
- `publish` — Public-facing delivery instance
- `dev`, `stage`, `prod` — Environment-specific configurations

---

## AEM Instance Types

| Feature | Author | Publish |
|---------|--------|---------|
| Purpose | Content creation | Content delivery |
| Port (default) | 4502 | 4503 |
| Access | Restricted (authors) | Public |
| Workflows | Yes | No |
| Replication | Source | Target |
| DAM processing | Yes | Read-only |

---

## Important AEM Directories

| Path | Purpose |
|------|---------|
| `/apps` | Custom application code |
| `/libs` | AEM product code (DO NOT modify) |
| `/content` | Authored content pages |
| `/content/dam` | Digital assets |
| `/conf` | Configurations (editable templates) |
| `/etc` | Legacy configs, designs, workflows |
| `/home` | Users and groups |
| `/var` | Audit logs, workflow instances |
| `/oak:index` | Index definitions |
| `/tmp` | Temporary data |

---

## AEM Build & Deployment

### Maven Project Structure
```
my-project/
├── pom.xml (reactor POM)
├── core/           → Java OSGi bundle
├── ui.apps/        → Components, templates (immutable)
├── ui.content/     → Sample/initial content (mutable)
├── ui.config/      → OSGi configs
├── ui.frontend/    → Frontend build (webpack/npm)
├── dispatcher/     → Dispatcher configs
└── all/            → Container package
```

### Key Maven Commands
```bash
# Full build + deploy to author
mvn clean install -PautoInstallSinglePackage

# Deploy only core bundle
mvn clean install -pl core -PautoInstallBundle

# Build without tests
mvn clean install -DskipTests

# Deploy to publish
mvn clean install -PautoInstallSinglePackagePublish
```

---

## MCQs — AEM Basics

### Q1: Which is NOT a foundational technology of AEM?
- A) Apache Sling
- B) Spring Framework ✅ **CORRECT**
- C) Apache Jackrabbit Oak
- D) OSGi (Apache Felix)

> **Explanation:** AEM is built on Sling + JCR (Oak) + OSGi. Spring is not part of AEM's stack.

### Q2: What is the default port for AEM Author instance?
- A) 8080
- B) 4503
- C) 4502 ✅ **CORRECT**
- D) 9090

> **Explanation:** Author = 4502, Publish = 4503 by default.

### Q3: Where should custom component code be placed?
- A) /libs
- B) /apps ✅ **CORRECT**
- C) /content
- D) /etc

> **Explanation:** `/apps` is for custom code. `/libs` is product code and should never be modified directly.

### Q4: What does Sling use to resolve a URL to a component script?
- A) Servlet mapping in web.xml
- B) Resource type of the content node ✅ **CORRECT**
- C) File extension only
- D) URL rewrite rules

> **Explanation:** Sling resolves the URL to a resource, reads its `sling:resourceType`, then finds the rendering script.

### Q5: In OSGi lifecycle, what state comes after RESOLVED?
- A) ACTIVE
- B) INSTALLED
- C) STARTING ✅ **CORRECT**
- D) STOPPING

> **Explanation:** The lifecycle is: INSTALLED → RESOLVED → STARTING → ACTIVE → STOPPING → UNINSTALLED.

### Q6: Which run mode identifies a content delivery instance?
- A) author
- B) publish ✅ **CORRECT**
- C) prod
- D) live

> **Explanation:** AEM uses `author` and `publish` as primary run modes for instance type.

### Q7: What content structure does JCR use?
- A) Relational tables
- B) Document collections
- C) Hierarchical nodes and properties ✅ **CORRECT**
- D) Key-value pairs

> **Explanation:** JCR stores everything as a tree of nodes. Each node has properties (key-value) and child nodes.

### Q8: What is the purpose of the reactor POM in an AEM Maven project?
- A) Deploys to the server
- B) Manages all sub-modules and build order ✅ **CORRECT**
- C) Contains application code
- D) Defines OSGi configurations

> **Explanation:** The reactor POM is the root/parent that orchestrates multi-module build.
