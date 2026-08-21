# AEM Interview Preparation — Part 10: Interview Tips & Bonus MCQs

---

## Common Interview Topics Summary

### Top 20 Questions You MUST Know

1. **What is AEM and its architecture?** → Sling + JCR + OSGi stack
2. **Explain Sling resource resolution** → URL → Resource → Script
3. **How do OSGi services work?** → @Component, @Reference, lifecycle
4. **What are Sling Models?** → Annotation-based JCR → Java mapping
5. **HTL vs JSP** → Secure by default, separation of concerns
6. **Editable vs Static Templates** → /conf vs /apps, policies
7. **Content Fragments vs Experience Fragments** → Data vs Layout
8. **How does Dispatcher caching work?** → Filesystem cache + .stat invalidation
9. **What is MSM/Live Copy?** → Blueprint → rollout → inheritance
10. **How do you create custom OSGi configs?** → @ObjectClassDefinition + @Designate
11. **AEM 6.5 vs AEMaaCS differences** → Immutable /apps, Cloud Manager, microservices
12. **Query optimization** → Indexes, path restriction, specific node types
13. **Service User Mapping** → No admin resolver, mapped permissions
14. **Workflow customization** → WorkflowProcess, ParticipantChooser
15. **ClientLib management** → Categories, dependencies, embed, allowProxy
16. **Replication process** → Author → Publish → Dispatcher flush
17. **Security best practices** → Input validation, CUG, CSRF, Dispatcher filters
18. **DAM asset processing** → Upload → Workflow/Compute → Renditions
19. **AEM headless** → Content Services, GraphQL, JSON exporter
20. **Performance troubleshooting** → Slow queries, memory leaks, caching

---

## Scenario-Based Questions

### Scenario 1: Component Not Rendering
**Q: Your component shows a blank area on the page. How do you debug?**

Steps:
1. Check browser console for JS errors
2. Check error.log for server-side exceptions
3. Verify `sling:resourceType` on the content node matches component path
4. Check if component dialog has required fields that are empty
5. Check if Sling Model has `@PostConstruct` errors
6. Verify component is allowed in the template policy
7. Check if `data-sly-test` condition is evaluating to false

### Scenario 2: Slow Page Load
**Q: A page takes 8 seconds to load. How do you investigate?**

Steps:
1. Check if Dispatcher cache is working (X-Dispatcher header)
2. Look at query performance logs (`/libs/granite/operations/content/diagnosis/tool.html`)
3. Check for unclosed ResourceResolvers (session leak)
4. Profile Sling Model initialization time
5. Check ClientLib size (network tab)
6. Verify no synchronous external API calls in rendering path
7. Check Oak indexes for traversal warnings

### Scenario 3: Content Not Appearing on Publish
**Q: Content is authored but not visible on publish. What do you check?**

Steps:
1. Check replication queue (is it blocked?)
2. Verify page is activated (green dot in Sites console)
3. Check user permissions on publish
4. Check Dispatcher cache (flush and retry)
5. Verify no CUG restriction on publish
6. Check if all referenced assets are also published
7. Check publish instance logs for errors

---

## Bonus MCQs — Mixed Topics

### Q1: What is the correct order of Sling request processing?
- A) Script Resolution → Resource Resolution → URL Decomposition
- B) URL Decomposition → Resource Resolution → Script Resolution ✅ **CORRECT**
- C) Resource Resolution → URL Decomposition → Script Resolution
- D) URL Decomposition → Script Resolution → Resource Resolution

### Q2: Which tool should you use to check OSGi bundle status?
- A) CRXDE Lite
- B) Package Manager
- C) Felix Console (/system/console/bundles) ✅ **CORRECT**
- D) Sites Console

### Q3: What is `sling:resourceSuperType` equivalent to in OOP?
- A) Interface implementation
- B) Class inheritance ✅ **CORRECT**
- C) Object composition
- D) Method overloading

### Q4: How do you make a servlet respond only to POST requests?
- A) Set sling.servlet.methods=POST in properties ✅ **CORRECT**
- B) Only implement doPost()
- C) Use @POST annotation
- D) Configure in web.xml

### Q5: What happens if two OSGi services implement the same interface?
- A) Error — only one allowed
- B) Higher service.ranking wins ✅ **CORRECT**
- C) Both execute sequentially
- D) Random selection

> **Explanation:** OSGi uses `service.ranking` (higher = preferred) to resolve ambiguity. Default is 0.

### Q6: In AEMaaCS, where should custom Oak indexes be defined?
- A) Created via CRXDE at runtime
- B) In the Git repository under ui.apps ✅ **CORRECT**
- C) Via Cloud Manager UI
- D) Cannot create custom indexes

### Q7: What is the purpose of the Externalizer service?
- A) External authentication
- B) Generate absolute URLs for different environments ✅ **CORRECT**
- C) External API calls
- D) Export content

> **Explanation:** Externalizer creates proper absolute URLs (e.g., for emails) per environment (author/publish/custom).

### Q8: Which approach is recommended for accessing JCR in AEM 6.5+?
- A) JCR Session API directly
- B) Sling Resource API ✅ **CORRECT**
- C) SQL queries
- D) REST calls

> **Explanation:** Sling Resource API is preferred over direct JCR Session API — it's higher-level and more portable.

### Q9: What does `cq:lastReplicated` property indicate?
- A) Last modification date
- B) When the page was last published/activated ✅ **CORRECT**
- C) When cache was cleared
- D) Last login time

### Q10: What is the Style System in AEM?
- A) CSS framework built into AEM
- B) Allows authors to apply CSS classes to components via policies ✅ **CORRECT**
- C) Automated styling based on content
- D) Theme management system

> **Explanation:** Style System lets template authors define CSS classes that content authors can apply without code changes.

### Q11: Which is NOT a valid Sling Model injection annotation?
- A) @ValueMapValue
- B) @ChildResource
- C) @JcrProperty ✅ **CORRECT**
- D) @OSGiService

### Q12: What does `immediate=true` do in an OSGi @Component?
- A) Deploys faster
- B) Activates component even without service consumers ✅ **CORRECT**
- C) Skips validation
- D) Enables hot-reload

> **Explanation:** By default, DS components activate lazily (when first requested). `immediate=true` forces activation at bundle start.

### Q13: How does AEM determine page language for i18n?
- A) Browser Accept-Language header only
- B) Language property from the closest ancestor with jcr:language ✅ **CORRECT**
- C) User profile setting
- D) Domain name

### Q14: What is a Sling Job?
- A) A cron task
- B) Guaranteed processing of background work (retries on failure) ✅ **CORRECT**
- C) A workflow step
- D) An OSGi bundle lifecycle event

> **Explanation:** Sling Jobs guarantee execution — they persist and retry on failure, unlike Sling Events which are fire-and-forget.

### Q15: In a Dispatcher filter, which rule takes precedence?
- A) First matching rule
- B) Last matching rule ✅ **CORRECT**
- C) Most specific rule
- D) Deny always wins

> **Explanation:** Dispatcher filter rules are evaluated in order; the LAST matching rule determines allow/deny.

---

## Key Terminology Quick Reference

| Term | Definition |
|------|-----------|
| **HTL** | HTML Template Language (replaces JSP) |
| **CRX** | Content Repository Extreme (AEM's JCR) |
| **CQ** | Legacy name (Day CQ → Adobe CQ → AEM) |
| **DAM** | Digital Asset Management |
| **MSM** | Multi-Site Manager |
| **CUG** | Closed User Group |
| **XF** | Experience Fragment |
| **CF** | Content Fragment |
| **CFM** | Content Fragment Model |
| **ACS Commons** | Community open-source AEM utilities |
| **Core Components** | Adobe's reference component library |
| **Granite UI** | AEM's server-side UI framework |
| **Coral UI** | AEM's client-side UI component library |
| **Sling** | Web framework (REST-based resource resolution) |
| **Felix** | Apache OSGi container used by AEM |
| **Oak** | Apache Jackrabbit Oak (JCR implementation) |
| **TarMK** | Segment-based storage (single instance) |
| **MongoMK** | MongoDB-based storage (clustered, deprecated) |
| **Dispatcher** | Apache module for caching + load balancing |
| **Replication** | Content sync from Author → Publish |

---

## Study Plan Recommendation

### Week 1: Foundations
- AEM Architecture (Sling + JCR + OSGi)
- JCR node types, properties, paths
- OSGi bundles, services, configurations
- Sling resource resolution

### Week 2: Development
- HTL syntax and all block statements
- Component creation (dialog, model, template)
- Sling Models (all injection types)
- Servlets (path-based and resource-type)
- ClientLibs

### Week 3: Advanced
- Workflows (custom steps, launchers)
- MSM (blueprints, live copies, rollout)
- Content Fragments & Experience Fragments
- Queries (JCR-SQL2, QueryBuilder, indexes)
- Dispatcher configuration

### Week 4: Cloud & Practice
- AEM as a Cloud Service differences
- Security best practices
- Performance optimization
- Practice MCQs and scenario questions
- Hands-on: Build a component end-to-end

---

## Good Luck with Your Interview!

Key tips:
- Always mention **why** (not just what) when answering
- Use real examples from your project experience
- Know the difference between 6.5 and Cloud Service
- Practice explaining architecturally (draw diagrams)
- Hands-on practice is more valuable than memorization
