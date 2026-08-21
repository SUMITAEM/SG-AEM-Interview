# AEM Interview Preparation — Part 2: OSGi Framework & Services

---

## What is OSGi?

OSGi (Open Services Gateway initiative) is a modular system for Java that allows:
- **Dynamic module loading** without restarting the JVM
- **Service-oriented architecture** within a single JVM
- **Version management** for dependencies
- **Lifecycle management** of components

AEM uses **Apache Felix** as its OSGi container.

---

## OSGi Bundle

A bundle is a JAR file with special metadata in `META-INF/MANIFEST.MF`:

```
Bundle-SymbolicName: com.mycompany.myproject.core
Bundle-Version: 1.0.0
Export-Package: com.mycompany.myproject.api
Import-Package: org.osgi.service.component
```

### Bundle States
```
INSTALLED → RESOLVED → STARTING → ACTIVE
                                     ↓
                                  STOPPING → UNINSTALLED
```

| State | Meaning |
|-------|---------|
| INSTALLED | Bundle is installed but dependencies not resolved |
| RESOLVED | All dependencies satisfied |
| STARTING | `BundleActivator.start()` called |
| ACTIVE | Bundle is running |
| STOPPING | `BundleActivator.stop()` called |

---

## Declarative Services (DS)

The modern way to write OSGi services in AEM:

### Creating a Service

**Step 1: Define the interface**
```java
public interface GreetingService {
    String greet(String name);
}
```

**Step 2: Implement with @Component**
```java
import org.osgi.service.component.annotations.Component;

@Component(service = GreetingService.class, immediate = true)
public class GreetingServiceImpl implements GreetingService {
    
    @Override
    public String greet(String name) {
        return "Hello, " + name;
    }
}
```

### Key Annotations

| Annotation | Purpose |
|------------|---------|
| `@Component` | Declares a class as an OSGi component |
| `@Reference` | Injects a service dependency |
| `@Activate` | Called when component is activated |
| `@Deactivate` | Called when component is deactivated |
| `@Modified` | Called when configuration changes |
| `@Designate` | Links component to configuration interface |

---

## OSGi Configuration

### Creating a Configuration Interface
```java
import org.osgi.service.metatype.annotations.ObjectClassDefinition;
import org.osgi.service.metatype.annotations.AttributeDefinition;

@ObjectClassDefinition(
    name = "My Service Configuration",
    description = "Configuration for my custom service"
)
public @interface MyServiceConfig {
    
    @AttributeDefinition(
        name = "API Endpoint",
        description = "The URL of the external API"
    )
    String apiEndpoint() default "http://localhost:8080";
    
    @AttributeDefinition(
        name = "Timeout",
        description = "Connection timeout in milliseconds"
    )
    int timeout() default 5000;
    
    @AttributeDefinition(
        name = "Enabled",
        description = "Enable or disable the service"
    )
    boolean enabled() default true;
}
```

### Using Configuration in a Service
```java
@Component(service = MyService.class, immediate = true)
@Designate(ocd = MyServiceConfig.class)
public class MyServiceImpl implements MyService {
    
    private String apiEndpoint;
    private int timeout;
    
    @Activate
    @Modified
    protected void activate(MyServiceConfig config) {
        this.apiEndpoint = config.apiEndpoint();
        this.timeout = config.timeout();
    }
}
```

### Configuration File Naming
Config files in `ui.config` follow naming convention:
```
com.mycompany.MyServiceImpl.cfg.json
```

Example content:
```json
{
    "apiEndpoint": "https://api.example.com",
    "timeout": 3000,
    "enabled": true
}
```

### Run-mode Specific Configs
```
ui.config/
├── src/main/content/jcr_root/apps/myapp/osgiconfig/
│   ├── config/                    → All environments
│   ├── config.author/             → Author only
│   ├── config.publish/            → Publish only
│   ├── config.author.dev/         → Author + Dev
│   └── config.publish.prod/       → Publish + Prod
```

---

## Service References

### Injecting Services
```java
@Component(service = Servlet.class)
public class MyServlet extends SlingSafeMethodsServlet {
    
    // Mandatory reference (default)
    @Reference
    private ResourceResolverFactory resolverFactory;
    
    // Optional reference
    @Reference(cardinality = ReferenceCardinality.OPTIONAL)
    private CacheService cacheService;
    
    // Multiple references
    @Reference(cardinality = ReferenceCardinality.MULTIPLE,
               policy = ReferencePolicy.DYNAMIC)
    private List<Plugin> plugins;
}
```

### Reference Policies

| Policy | Behavior |
|--------|----------|
| STATIC | Component restarts when reference changes |
| DYNAMIC | Reference updated without restart |

### Reference Cardinality

| Cardinality | Meaning |
|-------------|---------|
| MANDATORY (1..1) | Exactly one required (default) |
| OPTIONAL (0..1) | Zero or one |
| MULTIPLE (0..n) | Zero or more |
| AT_LEAST_ONE (1..n) | One or more |

---

## Service Factories

For creating multiple instances of the same service with different configs:
```java
@Component(
    service = DataSourceService.class,
    configurationPolicy = ConfigurationPolicy.REQUIRE
)
@Designate(ocd = DataSourceConfig.class, factory = true)
public class DataSourceServiceImpl implements DataSourceService {
    // Multiple instances possible via OSGi config factory
}
```

---

## OSGi Web Console

Accessible at: `http://localhost:4502/system/console`

Important sections:
- **Bundles** (`/system/console/bundles`) — View/manage bundles
- **Components** (`/system/console/components`) — DS components
- **Configuration** (`/system/console/configMgr`) — OSGi configs
- **Services** (`/system/console/services`) — Service registry
- **Log Support** (`/system/console/slinglog`) — Configure loggers

---

## MCQs — OSGi & Services

### Q1: Which annotation is used to inject an OSGi service into another component?
- A) @Inject
- B) @Autowired
- C) @Reference ✅ **CORRECT**
- D) @Service

> **Explanation:** In OSGi Declarative Services, `@Reference` is used to inject service dependencies.

### Q2: What is the default reference cardinality in OSGi DS?
- A) OPTIONAL (0..1)
- B) MANDATORY (1..1) ✅ **CORRECT**
- C) MULTIPLE (0..n)
- D) AT_LEAST_ONE (1..n)

> **Explanation:** Default is MANDATORY — the component won't activate until the reference is satisfied.

### Q3: Which lifecycle method is called when an OSGi configuration changes?
- A) @Activate
- B) @Deactivate
- C) @Modified ✅ **CORRECT**
- D) @Updated

> **Explanation:** `@Modified` is called when config changes without restart. If not present, component is deactivated and reactivated.

### Q4: Where is the OSGi web console accessible by default?
- A) /admin/console
- B) /system/console ✅ **CORRECT**
- C) /osgi/web
- D) /felix/console

> **Explanation:** The Felix web console is at `/system/console` (default credentials: admin/admin).

### Q5: What does `configurationPolicy = ConfigurationPolicy.REQUIRE` mean?
- A) Configuration is optional
- B) Component only activates when a configuration exists ✅ **CORRECT**
- C) Configuration is ignored
- D) Component uses default values

> **Explanation:** REQUIRE means the component won't activate unless an OSGi config is present.

### Q6: What format is used for OSGi config files in AEM as a Cloud Service?
- A) .xml
- B) .properties
- C) .cfg.json ✅ **CORRECT**
- D) .yaml

> **Explanation:** AEM as a Cloud Service uses `.cfg.json` format for OSGi configurations.

### Q7: Which OSGi container does AEM use?
- A) Eclipse Equinox
- B) Apache Felix ✅ **CORRECT**
- C) Knopflerfish
- D) Apache Karaf

> **Explanation:** AEM uses Apache Felix as its OSGi runtime container.

### Q8: What annotation links an OSGi component to its configuration interface?
- A) @Configuration
- B) @Designate ✅ **CORRECT**
- C) @Config
- D) @ObjectClassDefinition

> **Explanation:** `@Designate(ocd = MyConfig.class)` connects a component to its configuration.
