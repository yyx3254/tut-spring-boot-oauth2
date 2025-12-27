# CLAUDE.md - AI Assistant Guide

## Repository Overview

**Repository Name:** `tut-spring-boot-oauth2`
**Purpose:** Progressive tutorial demonstrating OAuth 2.0 integration with Spring Boot
**Type:** Educational multi-module Maven project
**Language:** Java 8 + JavaScript (jQuery)
**Framework:** Spring Boot 2.2.2.RELEASE, Spring Security OAuth2 Client
**Documentation Format:** AsciiDoc (.adoc)

This is a **Spring Guides** tutorial repository that teaches OAuth2 social login through 5 progressively complex examples. Each module is a standalone Spring Boot application that builds on the previous one.

## Project Structure

### Root Layout
```
tut-spring-boot-oauth2/
├── pom.xml                    # Parent POM (multi-module aggregator)
├── README.adoc                # Main tutorial documentation
├── overview.adoc              # Module descriptions
├── CONTRIBUTING.adoc          # DCO contribution requirements
├── mvnw, mvnw.cmd            # Maven wrapper scripts
├── .mvn/wrapper/             # Maven wrapper JAR
├── .github/                  # GitHub workflows (DCO config)
├── .travis.yml               # Travis CI configuration
├── .gitignore                # Git ignore rules
├── simple/                   # Module 1: Basic OAuth2 login
├── click/                    # Module 2: Explicit login link
├── logout/                   # Module 3: Logout + CSRF
├── two-providers/            # Module 4: Multiple providers
└── custom-error/             # Module 5: Custom error handling
```

### Module Structure (All 5 Modules Follow This Pattern)
```
<module-name>/
├── pom.xml                                    # Module build configuration
├── README.adoc                                # Step-by-step tutorial
└── src/
    ├── main/
    │   ├── java/com/example/
    │   │   └── SocialApplication.java         # Main + Security config
    │   └── resources/
    │       ├── application.yml                # OAuth2 client config
    │       └── static/
    │           └── index.html                 # Single-page frontend
    └── test/java/com/example/
        └── SocialApplicationTests.java        # Basic context test
```

## The 5 Modules

### 1. `simple` - Basic Auto-Redirect
- **Lines of Code:** ~28 (Java)
- **Features:** Auto-redirect to GitHub OAuth2 login
- **Key Classes:** `@SpringBootApplication` only
- **OAuth Providers:** GitHub only
- **Frontend:** Empty container div

### 2. `click` - Explicit Login Link
- **Lines of Code:** ~60 (Java)
- **Features:** Manual login button, user endpoint, security config
- **Key Classes:** `@SpringBootApplication`, `@RestController`, extends `WebSecurityConfigurerAdapter`
- **New Endpoints:** `GET /user`
- **OAuth Providers:** GitHub only
- **Frontend:** Login link, AJAX user fetch, show/hide authentication state

### 3. `logout` - Logout Functionality
- **Lines of Code:** ~67 (Java)
- **Features:** Logout button, CSRF token handling
- **Key Classes:** Same as `click`
- **New Endpoints:** None (uses Spring Security's `/logout`)
- **OAuth Providers:** GitHub only
- **Frontend:** Logout button, CSRF token with js-cookie, jQuery AJAX setup
- **Dependencies Added:** `webjars:js-cookie:2.1.0`

### 4. `two-providers` - Multiple OAuth Providers
- **Lines of Code:** ~67 (Java)
- **Features:** Choice between GitHub and Google
- **Key Classes:** Same as `logout`
- **OAuth Providers:** GitHub + Google
- **Frontend:** Two login links (one per provider)
- **Config Change:** Added Google client registration

### 5. `custom-error` - Advanced Error Handling
- **Lines of Code:** ~135 (Java) - Most complex module
- **Features:** Custom OAuth2 user validation, error messages, WebClient API calls
- **Key Classes:** `@SpringBootApplication`, `@Controller` (not `@RestController`)
- **New Endpoints:** `GET /error`
- **Beans:** `WebClient`, custom `OAuth2UserService`
- **Business Logic:** Validates GitHub user is in "spring-projects" organization
- **OAuth Providers:** GitHub + Google
- **Frontend:** Error message display
- **Dependencies Added:** `spring-webflux`, `reactor-netty`

## Technology Stack

### Backend
- **Spring Boot:** 2.2.2.RELEASE
- **Java Version:** 1.8
- **Spring Security OAuth2 Client:** Native Spring Boot support
- **Build Tool:** Maven 3.x
- **Testing:** JUnit 4, Spring Boot Test

### Frontend
- **jQuery:** 2.1.1 (via WebJars)
- **Bootstrap:** 3.2.0 (via WebJars)
- **js-cookie:** 2.1.0 (logout, two-providers, custom-error only)
- **WebJars Locator:** Automatic version resolution

### Infrastructure
- **CI/CD:** Travis CI (OpenJDK 8, Maven build)
- **Server Port:** 8080 (localhost only by default)
- **Contribution Model:** DCO (Developer Certificate of Origin) - requires Signed-off-by trailer

## Key Conventions & Patterns

### Java Code Conventions

#### 1. Package Structure
- **Package:** Always `com.example`
- **No sub-packages** - flat structure
- **Single class per module:** `SocialApplication.java`

#### 2. Main Application Class Pattern
```java
@SpringBootApplication
// Module 1: no additional annotations
// Modules 2-4: @RestController
// Module 5: @Controller
public class SocialApplication extends WebSecurityConfigurerAdapter {
    // Modules 2-5 only

    public static void main(String[] args) {
        SpringApplication.run(SocialApplication.class, args);
    }
}
```

#### 3. Security Configuration Pattern
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests(a -> a
            .antMatchers("/", "/error", "/webjars/**").permitAll()
            .anyRequest().authenticated()
        )
        .exceptionHandling(e -> e
            .authenticationEntryPoint(new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED))
        )
        .oauth2Login(); // Basic form
}
```

**Always Permit These Paths:**
- `/` - Home page
- `/error` - Error page
- `/webjars/**` - Static assets

#### 4. User Endpoint Pattern (Modules 2-5)
```java
@GetMapping("/user")
@ResponseBody // Omit if using @RestController
public Map<String, Object> user(@AuthenticationPrincipal OAuth2User principal) {
    return Collections.singletonMap("name", principal.getAttribute("name"));
}
```

#### 5. CSRF Configuration (Modules 3-5)
```java
.csrf(c -> c
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
)
```

#### 6. Logout Configuration (Modules 3-5)
```java
.logout(l -> l
    .logoutSuccessUrl("/").permitAll()
)
```

### Configuration Conventions

#### 1. Always Use YAML (Never .properties)
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            client-id: your-github-client-id
            client-secret: your-github-client-secret
          google:  # two-providers and custom-error only
            client-id: your-google-client-id
            client-secret: your-google-client-secret
```

#### 2. Debug Logging (Optional, Commented by Default)
```yaml
# logging:
#   level:
#     org.springframework.security: DEBUG
```

### Frontend Conventions

#### 1. Single HTML File Pattern
- **File:** `src/main/resources/static/index.html`
- **No separate CSS or JS files** - everything inline or via WebJars
- **Bootstrap 3.x** styling
- **jQuery 2.x** for AJAX

#### 2. WebJars URL Pattern (Version-less)
```html
<script type="text/javascript" src="/webjars/jquery/jquery.min.js"></script>
<script type="text/javascript" src="/webjars/bootstrap/js/bootstrap.min.js"></script>
```
No version in path - `webjars-locator-core` resolves automatically

#### 3. Login Link Pattern
```html
<!-- Single provider -->
<a href="/oauth2/authorization/github">Login with GitHub</a>

<!-- Multiple providers -->
<a href="/oauth2/authorization/github">Login with GitHub</a>
<a href="/oauth2/authorization/google">Login with Google</a>
```

#### 4. User Info Fetch Pattern (Modules 2-5)
```javascript
$.get("/user", function(data) {
    $("#user").html(data.name);
    $(".unauthenticated").hide();
    $(".authenticated").show();
});
```

#### 5. CSRF Setup (Modules 3-5)
```javascript
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (settings.type == 'POST' || settings.type == 'PUT'
            || settings.type == 'DELETE') {
            xhr.setRequestHeader("X-XSRF-TOKEN", Cookies.get('XSRF-TOKEN'));
        }
    }
});
```

#### 6. Logout Function Pattern (Modules 3-5)
```javascript
var logout = function() {
    $.post("/logout", function() {
        $("#user").html('');
        $(".unauthenticated").show();
        $(".authenticated").hide();
    });
    return false;
}
```

### Maven/Build Conventions

#### 1. Parent POM Structure
```xml
<groupId>org.springframework.guides</groupId>
<artifactId>tut-spring-boot-oauth</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>pom</packaging>

<modules>
    <module>simple</module>
    <module>click</module>
    <module>logout</module>
    <module>two-providers</module>
    <module>custom-error</module>
</modules>
```

#### 2. Module POM Pattern
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
</parent>

<properties>
    <java.version>1.8</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

#### 3. Core Dependencies (All Modules)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>2.1.1</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>3.2.0</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator-core</artifactId>
</dependency>
```

#### 4. Build Commands
```bash
# Run from module directory
./mvnw spring-boot:run

# Or build and run JAR
./mvnw package
java -jar target/*.jar

# Run all modules from root
./mvnw clean install
```

### Testing Conventions

#### 1. Minimal Test Coverage
- **Purpose:** Context loading verification only
- **Framework:** JUnit 4 (`@RunWith(SpringJUnit4ClassRunner.class)`)
- **No integration tests** - this is a tutorial, not production code
- **No OAuth2 mocking** - not covered in tutorial scope

#### 2. Standard Test Pattern (All Modules Identical)
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class SocialApplicationTests {
    @Test
    public void contextLoads() {
        // Empty test - verifies app context starts
    }
}
```

## Important URLs & Endpoints

### Standard Spring Security OAuth2 URLs
- **Authorization:** `/oauth2/authorization/{registrationId}`
  - Example: `/oauth2/authorization/github`
- **Callback/Redirect:** `/login/oauth2/code/{registrationId}`
  - Auto-configured by Spring Security
- **Logout:** `/logout` (POST)

### Application Endpoints
- **Home:** `/` (always public)
- **User Info:** `/user` (authenticated, modules 2-5)
- **Error:** `/error` (public, custom-error module only)

### Local Development
- **Host:** `localhost`
- **Port:** `8080`
- **Base URL:** `http://localhost:8080`

## OAuth2 Configuration Requirements

### GitHub OAuth App Registration
1. Go to: https://github.com/settings/developers
2. Register new OAuth app
3. **Homepage URL:** `http://localhost:8080`
4. **Callback URL:** `http://localhost:8080/login/oauth2/code/github`
5. Copy Client ID and Secret to `application.yml`

### Google OAuth App Registration
1. Go to: https://console.developers.google.com/
2. Create credentials → OAuth 2.0 Client ID
3. **Authorized redirect URIs:** `http://localhost:8080/login/oauth2/code/google`
4. Copy Client ID and Secret to `application.yml`

**SECURITY WARNING:** Never commit real client credentials to source control! Use placeholders like `your-github-client-id` in version control.

## Documentation Conventions

### AsciiDoc Format
- **Main README:** `README.adoc` (aggregates all module READMEs)
- **Module READMEs:** `<module>/README.adoc` (step-by-step tutorials)
- **Overview:** `overview.adoc` (module descriptions)
- **Contributing:** `CONTRIBUTING.adoc` (DCO requirements)

### AsciiDoc Headers Used
```asciidoc
:toc: left
:icons: font
:source-highlighter: prettify
:image-width: 500
:doctype: book
```

### Include Pattern
```asciidoc
include::simple/README.adoc[leveloffset=+1]
include::click/README.adoc[leveloffset=+1]
```

## Git & Version Control

### .gitignore Pattern
```
.idea
*.iml
.factorypath
classes
target
bin/
.classpath
.project
.settings/
Gemfile.lock
```

### Contribution Requirements
- **Model:** DCO (Developer Certificate of Origin)
- **Required:** `Signed-off-by` trailer on all commits
- **Command:** `git commit -s`
- **Reference:** https://spring.io/blog/2025/01/06/hello-dco-goodbye-cla-simplifying-contributions-to-spring

### GitHub Actions
- **DCO Check:** `.github/dco.yml` - Disabled for org members

## CI/CD

### Travis CI Configuration
```yaml
language: java
jdk: openjdk8
cache:
  directories:
    - $HOME/.m2
script: ./mvnw --fail-at-end --update-snapshots clean install
```

**Build Command:** `./mvnw --fail-at-end --update-snapshots clean install`
- Builds all 5 modules
- Updates snapshots
- Fails at end (tests all modules even if one fails)

## Progressive Enhancement Pattern

When modifying this codebase, understand the progression:

### simple → click
- Add `@RestController` annotation
- Extend `WebSecurityConfigurerAdapter`
- Add `configure(HttpSecurity)` method
- Add `/user` endpoint
- Add login link to frontend
- Add AJAX user fetch

### click → logout
- Add `js-cookie` dependency
- Add CSRF configuration
- Add logout configuration
- Add logout button to frontend
- Add CSRF token handling in JavaScript

### logout → two-providers
- Add Google client registration to `application.yml`
- Add second login link to frontend
- No code changes needed!

### two-providers → custom-error
- Change `@RestController` to `@Controller`
- Add `spring-webflux` and `reactor-netty` dependencies
- Add `@Bean WebClient`
- Add `@Bean OAuth2UserService` with custom logic
- Add `/error` endpoint
- Add error handling to `configure(HttpSecurity)`
- Add error display to frontend

## Common Tasks for AI Assistants

### Adding a New OAuth2 Provider
1. Register app with provider (get client ID/secret)
2. Add registration to `application.yml`:
   ```yaml
   spring.security.oauth2.client.registration.<provider-name>:
     client-id: <id>
     client-secret: <secret>
   ```
3. Add login link: `/oauth2/authorization/<provider-name>`
4. **No code changes needed** if using standard OAuth2/OIDC

### Upgrading Spring Boot Version
1. Update `<version>` in module `<parent>` tag
2. Test all 5 modules: `./mvnw clean install`
3. Update README if API changes
4. Check for deprecated `WebSecurityConfigurerAdapter` (deprecated in 2.7.x)

### Adding New Endpoints
1. Add `@GetMapping` or `@PostMapping` to `SocialApplication.java`
2. Add to whitelist if public: `.antMatchers("/your-path").permitAll()`
3. Use `@AuthenticationPrincipal OAuth2User` for authenticated endpoints
4. Return JSON for REST endpoints (use `@ResponseBody` if `@Controller`)

### Debugging OAuth2 Issues
1. Uncomment logging in `application.yml`:
   ```yaml
   logging:
     level:
       org.springframework.security: DEBUG
   ```
2. Check browser network tab for:
   - `/oauth2/authorization/{provider}` redirect
   - Provider's authorization endpoint
   - `/login/oauth2/code/{provider}` callback
3. Verify client ID/secret are correct
4. Verify redirect URI matches provider configuration

### Adding Custom User Validation
See `custom-error/SocialApplication.java` for pattern:
```java
@Bean
public OAuth2UserService<OAuth2UserRequest, OAuth2User> oauth2UserService(WebClient rest) {
    DefaultOAuth2UserService delegate = new DefaultOAuth2UserService();
    return request -> {
        OAuth2User user = delegate.loadUser(request);

        // Add custom validation logic here
        // Throw OAuth2AuthenticationException to reject

        return user;
    };
}
```

## Security Best Practices

### 1. Never Commit Credentials
- Use placeholders in source control
- Store real credentials in:
  - Environment variables
  - External config server
  - Secret management tools
- Add to `.gitignore` if using local property files

### 2. CSRF Protection
- Always enabled for modules with logout
- Use `CookieCsrfTokenRepository.withHttpOnlyFalse()`
- Send token in `X-XSRF-TOKEN` header for AJAX requests

### 3. HTTPS in Production
- `localhost:8080` is for development only
- Production requires HTTPS for OAuth2
- Re-register OAuth apps with production URLs

### 4. Scope Minimization
- Request only needed OAuth2 scopes
- Default scopes are usually sufficient for basic profile info

## Code Style Guidelines

### Java
- **Indentation:** Tabs (as seen in existing code)
- **Line Length:** Reasonable (no strict limit, but keep readable)
- **Comments:** Minimal - code should be self-documenting
- **License Header:** Apache 2.0 header on all Java files
- **Formatter Comments:** Use `// @formatter:off` and `// @formatter:on` around complex fluent API chains

### HTML/JavaScript
- **Indentation:** 2 spaces
- **jQuery Style:** Classic jQuery 2.x patterns (not modern ES6+)
- **Bootstrap:** Version 3.x conventions

### YAML
- **Indentation:** 2 spaces
- **Format:** Always YAML, never properties files

## Module Independence

**CRITICAL:** Each module is completely independent:
- Can be built separately: `cd click && ../mvnw package`
- Can be run separately: `cd click && ../mvnw spring-boot:run`
- No shared code between modules (intentional duplication for tutorial clarity)
- Changes to one module don't affect others

## File Naming Conventions

### Java
- Main class: Always `SocialApplication.java`
- Test class: Always `SocialApplicationTests.java`
- Package: Always `com.example`

### Resources
- Config: Always `application.yml` (never `application.properties`)
- Frontend: Always `static/index.html`
- No profile-specific configs (e.g., no `application-dev.yml`)

### Documentation
- Module docs: `<module>/README.adoc`
- Root docs: `README.adoc`, `overview.adoc`, `CONTRIBUTING.adoc`
- Always AsciiDoc (.adoc), never Markdown (.md)

## Troubleshooting Guide

### Build Fails
1. Check Java version: `java -version` (must be 1.8+)
2. Clean build: `./mvnw clean install`
3. Update snapshots: `./mvnw -U clean install`
4. Check internet connection (downloads dependencies)

### OAuth2 Login Fails
1. Verify client ID/secret are correct
2. Check redirect URI matches provider config
3. Enable debug logging
4. Check browser console for errors
5. Verify provider app is not suspended/deleted

### Port Already in Use
1. Change port in `application.yml`:
   ```yaml
   server:
     port: 8081
   ```
2. Update OAuth app redirect URIs accordingly
3. Or kill process on port 8080: `lsof -ti:8080 | xargs kill -9` (macOS/Linux)

## Performance Considerations

This is a **tutorial codebase**, not production-ready:
- No database (in-memory session storage only)
- No Redis/distributed sessions
- No connection pooling configuration
- No caching
- No production logging/monitoring
- Single server (no load balancing)

For production, consider adding:
- Spring Session with Redis
- Actuator for monitoring
- Proper logging framework (Logback/Log4j2)
- Connection pool tuning
- Rate limiting
- API gateway

## Useful References

### Spring Boot
- **OAuth2 Client:** https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-security-oauth2-client
- **Spring Security:** https://spring.io/projects/spring-security
- **WebJars:** https://www.webjars.org/

### OAuth2 Providers
- **GitHub OAuth Apps:** https://github.com/settings/developers
- **Google OAuth 2.0:** https://developers.google.com/identity/protocols/oauth2

### Tutorial
- **Main Guide:** README.adoc in root
- **Source Code:** https://github.com/spring-guides/tut-spring-boot-oauth2

## Quick Reference Commands

```bash
# Build all modules
./mvnw clean install

# Run a specific module
cd simple && ../mvnw spring-boot:run

# Package a module
cd click && ../mvnw package
cd click && java -jar target/*.jar

# Run tests only
./mvnw test

# Skip tests
./mvnw clean install -DskipTests

# Update dependencies
./mvnw versions:display-dependency-updates

# Check for plugin updates
./mvnw versions:display-plugin-updates
```

## Summary for AI Assistants

When working with this codebase:

1. **Understand the progression:** Each module builds on the previous one
2. **Respect module independence:** Don't create shared libraries or utils
3. **Keep it simple:** This is tutorial code - favor clarity over clever solutions
4. **Follow existing patterns:** Match the style and structure of similar modules
5. **Use YAML not properties:** Configuration is always in `application.yml`
6. **Never commit credentials:** Use placeholders in source control
7. **Single file per concern:** One Java class, one HTML file, one YAML file per module
8. **AsciiDoc for docs:** Never use Markdown in this project
9. **Test minimally:** Context load tests only - this isn't production code
10. **WebJars for frontend:** No npm, no bundlers - keep frontend simple

This is an **educational repository** focused on teaching OAuth2 concepts progressively. Prioritize clarity and tutorial value over production best practices.
