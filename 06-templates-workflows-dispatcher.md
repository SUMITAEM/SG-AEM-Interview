# AEM Interview Preparation — Part 6: Templates, Workflows & Dispatcher

---

## Editable Templates

Modern AEM uses **Editable Templates** (in `/conf`) rather than Static Templates (in `/apps`).

### Template Structure
```
/conf/myapp/settings/wcm/templates/
└── article-page/
    ├── jcr:content
    │    ├── cq:templateType = ".../page"
    │    └── status = "enabled"
    ├── initial/            → Initial content for new pages
    │    └── jcr:content/
    ├── structure/          → Locked structure (header, footer)
    │    └── jcr:content/
    ├── policies/           → Content policies
    │    └── jcr:content/
    └── thumbnail.png       → Template preview
```

### Template Types vs Templates
- **Template Type** = Blueprint (in `/libs` or `/apps`)
- **Template** = Created by template authors from a type (in `/conf`)
- **Page** = Created by content authors from a template (in `/content`)

### Template Policies
Policies define allowed components and styles for layout containers:
- Which components are allowed in a given container
- Design-level configurations (like RTE toolbar options)
- Style System classes

### Template Editor URL
```
http://localhost:4502/editor.html/conf/myapp/settings/wcm/templates/article-page/structure.html
```

---

## Static Templates (Legacy)

Stored in `/apps`:
```xml
<!-- /apps/myapp/templates/page/.content.xml -->
<jcr:root
    jcr:primaryType="cq:Template"
    jcr:title="Base Page"
    jcr:description="A basic page template"
    allowedPaths="[/content/myapp(/.*)?]"
    ranking="{Long}100">
    <jcr:content
        jcr:primaryType="cq:PageContent"
        sling:resourceType="myapp/components/page/base"/>
</jcr:root>
```

---

## AEM Workflows

Workflows automate business processes (content approval, asset processing, etc.).

### Workflow Components
1. **Workflow Model** — Defines steps and flow
2. **Workflow Instance** — Running execution
3. **Workflow Step** — Individual action (process, participant, split, etc.)
4. **Workflow Launcher** — Triggers workflow on events

### Workflow Step Types

| Step Type | Purpose |
|-----------|---------|
| **Process Step** | Executes Java code automatically |
| **Participant Step** | Assigns task to a user/group |
| **Dynamic Participant** | Calculates assignee at runtime |
| **OR Split** | Conditional branching |
| **AND Split** | Parallel execution |
| **Container Step** | Sub-workflow |
| **Goto Step** | Loops back to a previous step |

### Custom Workflow Process Step
```java
@Component(service = WorkflowProcess.class, property = {
    "process.label=My Custom Process"
})
public class MyWorkflowProcess implements WorkflowProcess {
    
    private static final Logger LOG = LoggerFactory.getLogger(MyWorkflowProcess.class);
    
    @Override
    public void execute(WorkItem workItem, WorkflowSession workflowSession,
                        MetaDataMap metaDataMap) throws WorkflowException {
        
        String payloadPath = workItem.getWorkflowData().getPayload().toString();
        LOG.info("Processing payload: {}", payloadPath);
        
        // Get arguments from dialog
        String arg = metaDataMap.get("PROCESS_ARGS", String.class);
        
        // Business logic here
        ResourceResolver resolver = workflowSession.adaptTo(ResourceResolver.class);
        Resource resource = resolver.getResource(payloadPath);
        
        if (resource != null) {
            ModifiableValueMap props = resource.adaptTo(ModifiableValueMap.class);
            props.put("reviewed", true);
            resolver.commit();
        }
    }
}
```

### Custom Participant Chooser
```java
@Component(service = ParticipantStepChooser.class, property = {
    "chooser.label=My Dynamic Participant"
})
public class MyParticipantChooser implements ParticipantStepChooser {
    
    @Override
    public String getParticipant(WorkItem workItem, WorkflowSession session,
                                  MetaDataMap metaData) throws WorkflowException {
        // Return user/group ID based on logic
        String path = workItem.getWorkflowData().getPayload().toString();
        if (path.contains("/marketing/")) {
            return "marketing-approvers";
        }
        return "content-approvers";
    }
}
```

### Workflow Launchers
Configure at: `/libs/settings/workflow/launcher`

| Field | Purpose |
|-------|---------|
| Event Type | Created, Modified, Deleted |
| Node Type | `cq:Page`, `dam:Asset` |
| Path Pattern | Glob pattern (e.g., `/content/mysite/.*`) |
| Condition | JCR property condition |
| Workflow Model | Path to workflow model |

---

## AEM Dispatcher

Dispatcher is Apache HTTP Server module for **caching** and **load balancing** in front of AEM Publish.

### Dispatcher Functions
1. **Caching** — Stores rendered pages as static files
2. **Load Balancing** — Distributes across publish instances
3. **Security** — URL filtering, header management

### Architecture
```
User → CDN → Dispatcher (Apache) → AEM Publish
                  │
            Cache (filesystem)
```

### Key Dispatcher Config Files

```
dispatcher/
├── conf.d/
│   ├── available_vhosts/
│   │   └── mysite.vhost          → Virtual host config
│   └── dispatcher_vhost.conf     → Includes
├── conf.dispatcher.d/
│   ├── cache/
│   │   └── rules.any             → What to cache
│   ├── clientheaders/
│   │   └── clientheaders.any     → Headers to pass through
│   ├── filters/
│   │   └── filters.any           → URL allow/deny rules
│   └── renders/
│       └── default_renders.any   → Publish instance URLs
└── dispatcher.any                → Main config
```

### Cache Rules
```
/cache {
    /rules {
        /0000 { /type "deny" /glob "*" }
        /0001 { /type "allow" /glob "*.html" }
        /0002 { /type "allow" /glob "*.css" }
        /0003 { /type "allow" /glob "*.js" }
        /0004 { /type "allow" /glob "*.jpg" }
        /0005 { /type "allow" /glob "*.png" }
        /0006 { /type "deny" /glob "*.json" }  # Dynamic
    }
    /invalidate {
        /0000 { /type "deny" /glob "*" }
        /0001 { /type "allow" /glob "*.html" }
    }
}
```

### Filter Rules (Security)
```
/filter {
    /0001 { /type "deny" /url "*" }                   # Deny all
    /0002 { /type "allow" /url "/content/*" }         # Allow content
    /0003 { /type "allow" /url "/etc.clientlibs/*" }  # Allow clientlibs
    /0004 { /type "deny" /url "/bin/*" }              # Block servlets
    /0005 { /type "deny" /url "/crx/*" }              # Block CRX
    /0006 { /type "deny" /url "/system/*" }           # Block system
    /0007 { /type "deny" /selectors '(feed|rss|pages|languages|blueprint|hierarchypage|social|hierarchypage|hierarchyrss)' }
}
```

### Cache Invalidation

**Auto-invalidation (stat file):**
When content is published, a `.stat` file is touched. Next request checks if cache file is older than `.stat` → re-fetches.

**Flush Agent:**
Replication agent on Author sends invalidation requests to Dispatcher when content is activated.

**Manual Flush:**
```
curl -H "CQ-Action: Activate" \
     -H "CQ-Handle: /content/mysite/en" \
     -H "CQ-Path: /content/mysite/en" \
     http://dispatcher-host/dispatcher/invalidate.cache
```

---

## Replication & Publishing

### Replication Agents
- **Default Agent** — Author → Publish (forward replication)
- **Reverse Agent** — Publish → Author (e.g., user-generated content)
- **Dispatcher Flush** — Author → Dispatcher (cache invalidation)

### Replication Process
```
Author Content Change
       │
       ▼
Replication Agent (serializes content)
       │
       ▼
Transport Layer (HTTP/S)
       │
       ▼
Publish Instance (deserializes + installs)
       │
       ▼
Dispatcher Flush (invalidates cache)
```

### Custom Replication Preprocessor
```java
@Component(service = Preprocessor.class)
public class MyReplicationPreprocessor implements Preprocessor {
    
    @Override
    public void preprocess(ReplicationAction action, 
                           ReplicationOptions options) throws ReplicationException {
        if (action.getType() == ReplicationActionType.ACTIVATE) {
            String path = action.getPath();
            // Validation logic before publishing
        }
    }
}
```

---

## MCQs — Templates, Workflows & Dispatcher

### Q1: Where are Editable Templates stored?
- A) /apps/templates
- B) /conf/{project}/settings/wcm/templates ✅ **CORRECT**
- C) /content/templates
- D) /libs/templates

> **Explanation:** Editable Templates live in `/conf`. Static (legacy) templates live in `/apps`.

### Q2: What does a Dispatcher cache store?
- A) JCR nodes
- B) Static HTML files on filesystem ✅ **CORRECT**
- C) Database records
- D) Session data

> **Explanation:** Dispatcher caches rendered output as flat files in the filesystem for fast delivery.

### Q3: Which workflow step type assigns work to a specific user?
- A) Process Step
- B) Participant Step ✅ **CORRECT**
- C) AND Split
- D) Container Step

> **Explanation:** Participant Steps create inbox items for users/groups to review and complete.

### Q4: What triggers a Workflow Launcher?
- A) Cron schedule only
- B) JCR node events (create/modify/delete) ✅ **CORRECT**
- C) HTTP requests
- D) OSGi events

> **Explanation:** Launchers observe JCR events and trigger workflows when conditions match.

### Q5: How does Dispatcher invalidation work by default?
- A) Deletes all cache files
- B) Updates .stat file, stale files are re-fetched on next request ✅ **CORRECT**
- C) Sends email notification
- D) Restarts Apache

> **Explanation:** The stat file mechanism marks cache as potentially stale; actual re-fetch happens on next request.

### Q6: What does a filter rule with `/type "deny"` do in Dispatcher?
- A) Logs the request
- B) Blocks the request from reaching AEM ✅ **CORRECT**
- C) Caches the response
- D) Redirects the request

> **Explanation:** Deny filter rules prevent matching URLs from passing through to the publish instance.

### Q7: What interface must a custom workflow process step implement?
- A) WorkflowStep
- B) WorkflowProcess ✅ **CORRECT**
- C) ProcessStep
- D) StepExecutor

> **Explanation:** Custom process steps implement `com.adobe.granite.workflow.exec.WorkflowProcess`.

### Q8: What is the purpose of Template Policies?
- A) User authentication
- B) Define allowed components and style options per container ✅ **CORRECT**
- C) URL routing
- D) Caching rules

> **Explanation:** Policies control which components can be added and what styles are available in containers.
