# Java Spring Boot Web Security Spec (Spring Boot 3.x, Spring Security 6.x, Java 21+)

This document is designed as a **security spec** that supports:

1. **Secure-by-default code generation** for new Spring Boot code.
2. **Security review / vulnerability hunting** in existing Spring Boot code (passive "notice issues while working" and active "scan the repo and report findings").

It is intentionally written as a set of **normative requirements** ("MUST/SHOULD/MAY") plus **audit rules** (what bad patterns look like, how to detect them, and how to fix/mitigate them).

Spring Boot is commonly deployed as a standalone JAR with an embedded server (Tomcat/Jetty/Undertow) and typically uses Spring Security, Spring Data JPA, and Spring MVC or Spring WebFlux, so this spec covers those layers where they affect security.

---

## 0) Safety, boundaries, and anti-abuse constraints (MUST FOLLOW)

* MUST NOT request, output, log, or commit secrets (API keys, passwords, private keys, session cookies, signing keys, database URLs with credentials, JWT signing keys).
* MUST NOT "fix" security by disabling protections (e.g., `csrf().disable()`, weakening auth, making CORS permissive with wildcard + credentials, skipping signature checks, disabling validation, adding `permitAll()` everywhere).
* MUST provide **evidence-based findings** during audits: cite file paths, code snippets, and configuration values that justify the claim.
* MUST treat uncertainty honestly: if a protection might exist in infrastructure (reverse proxy, WAF, CDN, service mesh), report it as "not visible in app code; verify at runtime/config".
* MUST treat browser controls correctly:
  * CORS is **not** an auth mechanism; it only affects browsers.
  * CSRF defenses apply when the browser automatically attaches credentials (cookies); they are usually not relevant for purely header-token (JWT Bearer) APIs.

---

## 1) Operating modes

### 1.1 Generation mode (default)

When asked to write new Spring Boot code or modify existing code:

* MUST follow every **MUST** requirement in this spec.
* SHOULD follow every **SHOULD** requirement unless the user explicitly says otherwise.
* MUST prefer safe-by-default APIs and proven libraries over custom security code.
* MUST avoid introducing new risky sinks (shell execution, unsafe deserialization, dynamic JPQL/SQL string building, untrusted template rendering, unsafe file serving, unsafe redirects, arbitrary outbound fetching).

### 1.2 Passive review mode (always on while editing)

While working anywhere in a Spring Boot repo (even if the user did not ask for a security scan):

* MUST "notice" violations of this spec in touched/nearby code.
* SHOULD mention issues as they come up, with a brief explanation + safe fix.

### 1.3 Active audit mode (explicit scan request)

When the user asks to "scan", "audit", or "hunt for vulns":

* MUST systematically search the codebase for violations of this spec.
* MUST output findings in a structured format (see §2.3).

Recommended audit order:

1. Build and deployment entrypoints: `pom.xml` / `build.gradle`, Dockerfiles, Kubernetes manifests, CI workflows.
2. Spring Boot version and dependency management; `spring-boot-starter-parent` version, CVE-tracked dependencies.
3. `application.properties` / `application.yml`: actuator exposure, debug flags, datasource credentials.
4. Spring Security configuration (`SecurityFilterChain`, `WebSecurityCustomizer`, `HttpSecurity`).
5. Authentication and authorization design (JWT, session, OAuth2, `@PreAuthorize`, method security).
6. Session and cookie security attributes.
7. CSRF protection for cookie-authenticated state-changing endpoints.
8. Input validation (`@Valid`, `@Validated`, Bean Validation, `@RequestBody` binding).
9. Data access layer: JPA repositories, native queries, `@Query` annotations, JPQL concatenation.
10. Template rendering and XSS (Thymeleaf auto-escaping, `th:utext`).
11. File handling: upload validation, path traversal in file serving.
12. Injection classes: OS command execution, SSRF, open redirects.
13. Security headers, CORS, and proxy configuration.
14. Actuator endpoints and management interface exposure.
15. Serialization / deserialization (Jackson polymorphic types, Java serialization).

---

## 2) Definitions and review guidance

### 2.1 Untrusted input (treat as attacker-controlled unless proven otherwise)

Examples include:

* `@RequestParam`, `@PathVariable`, `@RequestHeader`, `@CookieValue`
* `@RequestBody` (JSON/XML/form bodies)
* `HttpServletRequest` fields: `getParameter()`, `getHeader()`, `getInputStream()`
* Multipart file uploads (`MultipartFile`)
* Any data from external systems (webhooks, third-party APIs, message queues)
* Any persisted user content (DB rows) that originated from users
* Configuration values that might be attacker-influenced in some deployments (headers set by upstream proxies, environment variables in multi-tenant systems)

### 2.2 State-changing request

A request is state-changing if it can create/update/delete data, change auth/session state, trigger side effects (purchase, email send, webhook send), or initiate privileged actions.

### 2.3 Required audit finding format

For each issue found, output:

* Rule ID:
* Severity: Critical / High / Medium / Low
* Location: file path + class/method name + line(s)
* Evidence: the exact code/config snippet
* Impact: what could go wrong, who can exploit it
* Fix: safe change (prefer minimal diff)
* Mitigation: defense-in-depth if immediate fix is hard
* False positive notes: what to verify if uncertain

---

## 3) Secure baseline: minimum production configuration (MUST in production)

This is the smallest "production baseline" that prevents common Spring Boot misconfigurations.

Baseline goals:

* Spring Boot running in production profile with `debug=false` and no dev-only tools active.
* Spring Security configured explicitly — not relying on auto-configuration defaults for production.
* Actuator endpoints restricted; sensitive endpoints not publicly reachable.
* Authentication enforced on all non-public routes; deny-by-default posture.
* CSRF protection enabled for cookie-authenticated apps; explicitly disabled only for stateless JWT APIs.
* Request size limits in place to prevent memory-based DoS.
* Passwords hashed with a strong slow algorithm (BCrypt, Argon2).
* Dependencies patched promptly; `spring-boot-starter-parent` kept up-to-date.

---

## 4) Rules (generation + audit)

Each rule contains: required practice, insecure patterns, detection hints, and remediation.

---

### SPRING-DEPLOY-001: Do not use dev-only tools or profiles in production

Severity: High (if production)

Required:

* MUST NOT run production with `spring-boot-devtools` on the classpath or `spring.devtools.restart.enabled=true`.
* MUST NOT use `mvn spring-boot:run` or `gradle bootRun` as the production entrypoint; use the packaged JAR/WAR.
* MUST set `spring.profiles.active=prod` (or equivalent) and ensure production profile disables dev-only beans.

Insecure patterns:

* `spring-boot-devtools` in `dependencies` scope (not `optional` / `developmentOnly`) in `pom.xml` / `build.gradle`.
* Docker `CMD` or `ENTRYPOINT` running `mvn` / `./gradlew` directly.
* `spring.h2.console.enabled=true` in a non-dev profile.

Detection hints:

* Search `pom.xml` / `build.gradle` for `spring-boot-devtools` without `optional` / `developmentOnly`.
* Search `application*.yml` / `application*.properties` for `spring.h2.console.enabled=true`, `devtools`, or `spring.profiles.active=dev`.
* Inspect Dockerfiles for `mvn`, `gradlew` run commands in `CMD`/`ENTRYPOINT`.

Fix:

* Move `spring-boot-devtools` to `developmentOnly` scope in Gradle or mark `<optional>true</optional>` in Maven (it is then excluded from fat JAR).
* Use `java -jar app.jar --spring.profiles.active=prod` in production containers.

---

### SPRING-DEPLOY-002: Debug and stack trace disclosure MUST be disabled in production

Severity: Critical

Required:

* MUST set `server.error.include-stacktrace=never` (Spring Boot default) and confirm it is not overridden to `always` or `on_param`.
* MUST set `server.error.include-message=never` or `on_param` only with explicit justification.
* MUST NOT expose raw exception messages to API consumers in production.

Insecure patterns:

* `server.error.include-stacktrace=always` in `application.yml`.
* `server.error.include-message=always`.
* Custom `@ExceptionHandler` returning `e.getMessage()` or `e.getStackTrace()` directly in the HTTP response body.

Detection hints:

* Search `application*.yml` / `application*.properties` for `include-stacktrace`, `include-message`, `include-binding-errors`.
* Search `@ExceptionHandler` methods for `e.getMessage()`, `e.toString()`, `printStackTrace` in response bodies.

Fix:

* Return generic error responses to clients (`{"error": "Internal server error"}`); log details with a correlation ID server-side.
* Use a centralized `@ControllerAdvice` / `ProblemDetail` handler that never leaks stack traces.

---

### SPRING-ACTUATOR-001: Actuator endpoints MUST be restricted in production

Severity: High (Critical for sensitive endpoints like `/actuator/env`, `/actuator/heapdump`, `/actuator/shutdown`)

Required:

* MUST NOT expose sensitive actuator endpoints publicly without authentication.
* SHOULD expose actuator on a separate management port (`management.server.port`) accessible only from internal networks.
* MUST explicitly allow only necessary endpoints via `management.endpoints.web.exposure.include`.
* MUST NOT enable `shutdown` endpoint unless specifically needed and protected.

Insecure patterns:

* `management.endpoints.web.exposure.include=*` in production.
* Actuator endpoints on the same port as the public API without auth.
* `management.endpoint.shutdown.enabled=true` without auth or network restrictions.

Detection hints:

* Search `application*.yml` / `application*.properties` for `management.endpoints.web.exposure.include`, `management.server.port`, `management.endpoint.shutdown`.
* Check `SecurityFilterChain` for whether `/actuator/**` routes are protected.

Fix:

* Set `management.endpoints.web.exposure.include=health,info` (minimal) or restrict to specific needs.
* Bind management to a separate internal port: `management.server.port=8081`.
* Require authentication for sensitive actuator endpoints in `SecurityFilterChain`:
  ```java
  .requestMatchers("/actuator/**").hasRole("ADMIN")
  ```

---

### SPRING-SECURITY-001: Spring Security MUST be configured explicitly with deny-by-default

Severity: Critical

Required:

* MUST define a `SecurityFilterChain` bean that configures `HttpSecurity` explicitly for every route.
* MUST use a deny-by-default posture: explicitly permit only known-public endpoints, require authentication for everything else.
* MUST NOT rely solely on Spring Boot's auto-configured `SecurityFilterChain` without reviewing its defaults.

Insecure patterns:

* No `SecurityFilterChain` bean present (relying entirely on auto-config).
* `http.authorizeHttpRequests(auth -> auth.anyRequest().permitAll())` — opens all routes.
* Individual `@PreAuthorize` annotations as the only protection with no global filter chain enforcement.
* `WebSecurityCustomizer` configured with `web.ignoring().anyRequest()`.

Detection hints:

* Search for `SecurityFilterChain`, `WebSecurityConfigurerAdapter` (deprecated).
* Search for `.anyRequest().permitAll()`, `.anyRequest().authenticated()`.
* Search for `web.ignoring()` patterns.
* If no `@Configuration` class with `HttpSecurity` is found, flag as missing explicit security config.

Fix:

* Define a `SecurityFilterChain` with explicit route mappings:
  ```java
  http.authorizeHttpRequests(auth -> auth
      .requestMatchers("/public/**", "/api/auth/**").permitAll()
      .anyRequest().authenticated()
  );
  ```

---

### SPRING-SECURITY-002: CSRF protection MUST be correctly configured

Severity: High

Required:

* MUST NOT disable CSRF (`csrf().disable()` or `.csrf(AbstractHttpConfigurer::disable)`) for applications that use cookie-based session authentication.
* For stateless REST APIs that authenticate exclusively via `Authorization: Bearer` headers (JWT or API key in header only), disabling CSRF is acceptable — but MUST be documented and confirmed that no cookie auth exists.
* SHOULD use Spring Security's built-in `CsrfTokenRepository` (default `HttpSessionCsrfTokenRepository` or `CookieCsrfTokenRepository`).

Insecure patterns:

* `.csrf(AbstractHttpConfigurer::disable)` in a session-cookie authenticated app.
* CSRF disabled globally but some routes accept cookies (e.g., mixed auth scheme).

Detection hints:

* Search for `.csrf(AbstractHttpConfigurer::disable)`, `csrf().disable()`.
* Confirm whether the app uses `SESSION` cookies or JWT Bearer headers for auth.
* Check if any endpoints accept cookies for auth — if yes, CSRF must be enabled.

Fix:

* If the app uses session cookies, remove the CSRF disable and configure `CsrfTokenRepository`:
  ```java
  .csrf(csrf -> csrf.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()))
  ```
* If stateless (JWT only), document the rationale for disabling CSRF alongside the security config code.

---

### SPRING-AUTH-001: Authentication MUST be enforced explicitly and uniformly

Severity: Critical

Required:

* MUST use `SecurityFilterChain` to enforce authentication at the HTTP layer, not just at the controller/service layer.
* MUST enable method-level security (`@EnableMethodSecurity`) if using `@PreAuthorize` / `@Secured` as a supplementary layer — but this MUST NOT replace the HTTP filter layer.
* SHOULD NOT rely solely on controller-level `@PreAuthorize` checks as the only auth enforcement.

Insecure patterns:

* Controllers with missing `@PreAuthorize` and no filter-chain protection.
* Auth checks implemented only inside service methods without an outer HTTP layer gate.
* Forgetting to protect newly added routes when the `SecurityFilterChain` uses path-specific matchers.

Detection hints:

* Audit all `@RestController` and `@Controller` classes; for each endpoint, confirm there is either a filter chain rule or a method security annotation.
* Check whether `@EnableMethodSecurity` (or `@EnableGlobalMethodSecurity`) is present.

Fix:

* Prefer filter chain rules for broad protection and use `@PreAuthorize` for fine-grained authorization within authenticated areas:
  ```java
  @PreAuthorize("hasRole('ADMIN') or #userId == authentication.name")
  public void updateUser(String userId, ...) { ... }
  ```

---

### SPRING-AUTH-002: Passwords MUST be stored using a strong adaptive hash algorithm

Severity: Critical

Required:

* MUST use `BCryptPasswordEncoder`, `Argon2PasswordEncoder`, or `SCryptPasswordEncoder` (all provided by Spring Security).
* MUST NOT store plaintext passwords, MD5/SHA-1/SHA-256 without salt/KDF, or reversible encryption.
* SHOULD use `DelegatingPasswordEncoder` to support future algorithm upgrades.
* MUST NOT return password hashes in API responses.

Insecure patterns:

* `passwordEncoder.encode(password)` with `NoOpPasswordEncoder`.
* `MessageDigest.getInstance("MD5")` / `"SHA-1"` / `"SHA-256"` applied to passwords.
* Storing passwords with `{noop}` prefix in UserDetailsService.
* Password fields included in API response DTOs.

Detection hints:

* Search for `NoOpPasswordEncoder`, `{noop}`, `MessageDigest`, `MD5`, `SHA-1` near password fields.
* Search for `password` in response DTO fields or `@JsonProperty("password")`.
* Check `PasswordEncoder` bean definitions.

Fix:

* Use `BCryptPasswordEncoder` (strength 10-12) or Argon2:
  ```java
  @Bean
  public PasswordEncoder passwordEncoder() {
      return new BCryptPasswordEncoder(12);
  }
  ```
* Migrate existing hashes with `DelegatingPasswordEncoder` and a migration strategy.

---

### SPRING-AUTH-003: JWT tokens MUST be validated strictly

Severity: High

Required:

* MUST validate JWT signature using an algorithm allowlist; never accept `alg=none`.
* MUST validate standard claims: `exp` (expiry), `iss` (issuer), `aud` (audience) where applicable.
* MUST use a well-maintained library (e.g., `nimbus-jose-jwt`, `spring-security-oauth2-resource-server`).
* MUST NOT store secrets (private keys, passwords, internal tokens) in JWT payloads.
* SHOULD use short-lived access tokens (15 minutes to 1 hour) and refresh token rotation.

Insecure patterns:

* `Jwts.parserBuilder()` without `.setSigningKey(...)` or with empty key.
* Accepting `alg=none` or RS256/HS256 algorithm confusion.
* JWT expiry not validated.
* Secrets stored in JWT claims.
* Long-lived access tokens (days/weeks) without rotation.

Detection hints:

* Search for `Jwts.parser`, `Jwts.parserBuilder`, `JwtDecoder`, `JwtParser`.
* Confirm algorithm constraints are set; check for `allowedClockSkewSeconds` being overly large.
* Check JWT claim validation for `exp`, `iss`, `aud`.

Fix:

* Use Spring Security's built-in JWT support:
  ```java
  @Bean
  public JwtDecoder jwtDecoder() {
      NimbusJwtDecoder decoder = NimbusJwtDecoder.withSecretKey(secretKey).build();
      OAuth2TokenValidator<Jwt> validators = new DelegatingOAuth2TokenValidator<>(
          JwtValidators.createDefaultWithIssuer("https://my-issuer"),
          new JwtClaimValidator<List<String>>("aud", aud -> aud.contains("my-api"))
      );
      decoder.setJwtValidator(validators);
      return decoder;
  }
  ```

---

### SPRING-AUTH-004: Authorization MUST be verified server-side for every resource access

Severity: Critical

Required:

* MUST verify that the authenticated principal is authorized to access the specific resource being requested (object-level authorization / IDOR prevention).
* MUST NOT rely on the client to send only authorized resource IDs.
* SHOULD use Spring Security's `@PreAuthorize` with SpEL expressions to enforce ownership:

Insecure patterns:

* `findById(id)` without checking `id` belongs to the current user.
* Using sequential/numeric resource IDs without authorization check.
* Trusting `userId` from the request body instead of from `Authentication`.

Detection hints:

* Search for `repository.findById(id)`, `repository.getOne(id)`, `getById(id)` in service/controller layers.
* Check if the returned entity's owner is compared against `SecurityContextHolder.getContext().getAuthentication()`.

Fix:

* Always derive the owner identity from the security context, not from the request:
  ```java
  @PreAuthorize("#document.owner == authentication.name")
  public Document getDocument(Long id) {
      Document doc = documentRepo.findById(id)
          .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
      // ownership verified via @PreAuthorize
      return doc;
  }
  ```

---

### SPRING-SESSION-001: Session and cookie security attributes MUST be configured

Severity: High

Required:

* MUST set session cookie with `HttpOnly=true`, `Secure=true` (on TLS deployments), and `SameSite=Strict` or `Lax`.
* MUST set a reasonable `maxInactiveInterval` for sessions (default 30 minutes is usually acceptable).
* MUST invalidate sessions on logout using `SecurityContextLogoutHandler` or Spring Security's built-in logout support.
* SHOULD call `sessionManagement().sessionFixation().migrateSession()` (Spring Security default) to prevent session fixation.

Insecure patterns:

* `server.servlet.session.cookie.http-only=false`.
* `server.servlet.session.cookie.secure=false` in a TLS deployment.
* Custom logout that calls only `session.invalidate()` without clearing the `SecurityContext`.
* `sessionFixation().none()`.

Detection hints:

* Search `application*.yml` / `application*.properties` for `session.cookie`.
* Search `SecurityFilterChain` for `sessionManagement()`, `sessionFixation()`, `logout()`.
* Check logout handlers for `SecurityContextHolder.clearContext()` or equivalent.

Fix:

* Set cookie properties:
  ```yaml
  server:
    servlet:
      session:
        cookie:
          http-only: true
          secure: true
          same-site: strict
  ```
* Use Spring Security's logout support:
  ```java
  .logout(logout -> logout
      .logoutUrl("/api/auth/logout")
      .addLogoutHandler(new SecurityContextLogoutHandler())
      .invalidateHttpSession(true)
      .clearAuthentication(true)
  )
  ```

---

### SPRING-INPUT-001: Request body binding MUST be validated with Bean Validation

Severity: High

Required:

* MUST annotate controller method parameters with `@Valid` or `@Validated` when binding `@RequestBody`, `@ModelAttribute`, or form fields.
* MUST use constraint annotations (`@NotNull`, `@Size`, `@Pattern`, `@Email`, etc.) on DTO/command classes.
* MUST handle `MethodArgumentNotValidException` and `ConstraintViolationException` globally and return appropriate error responses without leaking internal details.
* MUST NOT trust client-supplied type conversions without explicit validation.

Insecure patterns:

* `@RequestBody UserDto user` without `@Valid` — all constraints silently ignored.
* `@ModelAttribute` without validation — mass assignment of all form fields to entity.
* Entity classes used directly as `@RequestBody` targets (exposes all fields including `id`, `role`, etc.).

Detection hints:

* Search for `@RequestBody` without `@Valid` or `@Validated`.
* Search for `@ModelAttribute` without `@Valid`.
* Check whether JPA `@Entity` classes are used directly as request body types.

Fix:

* Always add `@Valid`:
  ```java
  public ResponseEntity<Void> createUser(@Valid @RequestBody CreateUserRequest request) { ... }
  ```
* Use separate DTO/request classes (never expose entities directly to the HTTP layer).

---

### SPRING-INPUT-002: Mass assignment MUST be prevented

Severity: High

Required:

* MUST NOT bind HTTP request data directly to JPA entity objects.
* MUST use dedicated request DTOs and explicitly map only the fields the user is permitted to set.
* SHOULD use `@JsonIgnoreProperties` or constructor-based DTOs to prevent binding of unexpected fields.

Insecure patterns:

* `User user = objectMapper.convertValue(requestBody, User.class)` then `userRepo.save(user)`.
* `BeanUtils.copyProperties(request, entity)` where entity has privileged fields like `role`, `isAdmin`, `balance`.
* Spring Data REST exposure of JPA entities directly without projections.

Detection hints:

* Search for `@RepositoryRestResource` (Spring Data REST) — verify projections are used.
* Search for `BeanUtils.copyProperties` between request objects and entity classes.
* Inspect `@RequestBody` types — if they are `@Entity` classes, flag them.

Fix:

* Use explicit field mapping:
  ```java
  User user = new User();
  user.setName(request.getName()); // Only the fields the user can set
  user.setEmail(request.getEmail());
  userRepo.save(user);
  ```

---

### SPRING-SQL-001: SQL queries MUST use parameterized statements or the JPA criteria API

Severity: Critical

Required:

* MUST use JPA/Spring Data repository query methods, JPQL with named parameters, or `JdbcTemplate`/`NamedParameterJdbcTemplate` with parameters — never string concatenation.
* MUST NOT concatenate user input into JPQL, HQL, or native SQL strings.
* SHOULD prefer Spring Data JPA repository query methods (which generate safe queries) over manual query building.

Insecure patterns:

* `entityManager.createQuery("SELECT u FROM User u WHERE u.name = '" + name + "'")`.
* `jdbcTemplate.query("SELECT * FROM users WHERE name = '" + name + "'", ...)`.
* `@Query("SELECT u FROM User u WHERE u.name = '" + name + "'")` with JPQL concatenation.

Detection hints:

* Search for `createQuery(`, `createNativeQuery(`, `jdbcTemplate.query(`, `jdbcTemplate.execute(` followed by string concatenation (`+`).
* Search for `@Query` annotations where the query string includes `+` with method parameters.

Fix:

* Use named parameters:
  ```java
  // JPQL with named parameters
  Query query = em.createQuery("SELECT u FROM User u WHERE u.name = :name");
  query.setParameter("name", name);

  // Spring Data @Query
  @Query("SELECT u FROM User u WHERE u.name = :name")
  Optional<User> findByName(@Param("name") String name);

  // JdbcTemplate
  namedParamJdbcTemplate.query("SELECT * FROM users WHERE name = :name",
      Map.of("name", name), rowMapper);
  ```

---

### SPRING-SQL-002: Native SQL queries and stored procedures MUST use parameterization

Severity: Critical

Required:

* MUST use positional (`?1`) or named (`:name`) parameters for all `@Query(nativeQuery = true)` annotations.
* MUST NOT build native SQL strings from user input anywhere in the codebase.
* SHOULD validate, sanitize, and restrict allowed values for any dynamic query segment (e.g., column sort orders) using an allowlist.

Insecure patterns:

* `@Query(value = "SELECT * FROM users WHERE role = '" + role + "'", nativeQuery = true)`.
* `entityManager.createNativeQuery("SELECT * FROM " + tableName)` where `tableName` is user-controlled.
* Dynamic `ORDER BY` built from user-supplied column names without allowlisting.

Detection hints:

* Search for `nativeQuery = true` annotations and inspect the query string.
* Search for `createNativeQuery(` with string concatenation.
* Search for any dynamic query construction with sorting or filtering fields from request parameters.

Fix:

* For dynamic `ORDER BY`, use an allowlist:
  ```java
  private static final Set<String> ALLOWED_SORT_COLUMNS = Set.of("name", "createdAt", "email");

  public List<User> findUsersSorted(String sortBy) {
      if (!ALLOWED_SORT_COLUMNS.contains(sortBy)) {
          throw new IllegalArgumentException("Invalid sort column");
      }
      // Use the validated column name in a static query template
  }
  ```

---

### SPRING-XSS-001: Template rendering MUST use auto-escaping; `th:utext` MUST NOT be used with untrusted data

Severity: High

Required:

* MUST use Thymeleaf's `th:text` (auto-escaped) instead of `th:utext` (raw HTML) for any user-supplied data.
* MUST NOT use `th:utext`, `[(${...})]` (inlined unescaped), or `[[${...}]]` for untrusted data.
* If returning JSON APIs (no templates), this rule applies only if the app also serves HTML views.

Insecure patterns:

* `<p th:utext="${user.bio}"></p>` where `bio` is user-supplied.
* `<div>[(${comment.body})]</div>` — unescaped inline expression.
* Disabling Thymeleaf escaping globally.

Detection hints:

* Search for `th:utext=`, `[(...)]` (unescaped text inlining) in Thymeleaf templates.
* Check if any template variables are derived from user input.

Fix:

* Replace `th:utext` with `th:text`:
  ```html
  <!-- UNSAFE -->
  <p th:utext="${user.bio}"></p>

  <!-- SAFE -->
  <p th:text="${user.bio}"></p>
  ```
* If rich HTML is needed, sanitize with OWASP Java HTML Sanitizer before passing to `th:utext`.

---

### SPRING-SSTI-001: Dynamic Thymeleaf template execution from untrusted input MUST be prevented

Severity: Critical

Required:

* MUST NOT pass user-controlled strings as Thymeleaf template content to `templateEngine.process(templateString, context)` or `SpringTemplateEngine`.
* MUST NOT allow user input to determine a template path that resolves outside the template directory.
* Thymeleaf expressions like `${...}` and `__${...}__` support SpEL — user-controlled SpEL can execute arbitrary Java code.

Insecure patterns:

* `templateEngine.process(userInputString, context)` — direct template injection.
* Using user input in `__${''.class.forName('java.lang.Runtime').getMethod(...)}__` style expressions.
* Template file path derived from user request: `templateEngine.process("templates/" + userInput, context)`.

Detection hints:

* Search for `templateEngine.process(` or `SpringTemplateEngine.process(` with non-static first argument.
* Search for `TemplateEngine`, `ISpringWebFluxTemplateEngine` with dynamic template strings.

Fix:

* Always load templates from classpath (static template names), never from user input:
  ```java
  // SAFE: static template name from classpath
  templateEngine.process("email/welcome", context);

  // UNSAFE: never do this
  templateEngine.process(userSuppliedTemplateName, context);
  ```

---

### SPRING-CMD-001: OS command execution MUST NOT include untrusted input

Severity: Critical

Required:

* MUST NOT pass user-controlled input directly to `Runtime.exec()`, `ProcessBuilder`, or any shell-invoking wrapper.
* SHOULD avoid shell invocation entirely when a Java library can accomplish the task.
* If OS commands are genuinely needed, MUST use an argument array (not a shell string) and validate inputs against a strict allowlist.

Insecure patterns:

* `Runtime.getRuntime().exec("convert " + userFilename)`.
* `new ProcessBuilder("bash", "-c", userInput).start()`.
* Passing user data to external process wrappers like Apache Commons Exec with string interpolation.

Detection hints:

* Search for `Runtime.getRuntime().exec(`, `ProcessBuilder`, `Process`.
* Check all string arguments for inclusion of request parameters.

Fix:

* Use an argument array and validate each element:
  ```java
  // SAFE: argument array, no shell interpolation
  ProcessBuilder pb = new ProcessBuilder(
      "convert",
      validatedInputFile,   // validate against an allowlist/pattern
      validatedOutputFile
  );
  pb.redirectErrorStream(true);
  Process process = pb.start();
  ```

---

### SPRING-SSRF-001: Outbound HTTP requests MUST NOT use user-controlled URLs without validation

Severity: High

Required:

* MUST NOT pass user-supplied URLs directly to `RestTemplate`, `WebClient`, `HttpURLConnection`, or similar.
* MUST validate and restrict outbound URLs against an allowlist of permitted domains/schemes.
* MUST block requests to internal/private IP ranges (RFC 1918, loopback, link-local) and cloud metadata endpoints (169.254.169.254, etc.) unless explicitly needed.

Insecure patterns:

* `restTemplate.getForObject(userSuppliedUrl, String.class)`.
* `webClient.get().uri(request.getParam("url")).retrieve()`.
* Fetching user-supplied webhook callback URLs without validation.

Detection hints:

* Search for `restTemplate.get`, `restTemplate.exchange`, `WebClient`, `HttpURLConnection` with non-static URL arguments.
* Search for URL parameters like `url`, `callback`, `redirect`, `fetch`, `proxy`, `endpoint` passed to HTTP clients.

Fix:

* Validate URL against an allowlist before use:
  ```java
  private static final Set<String> ALLOWED_DOMAINS = Set.of("api.trusted.com", "hooks.trusted.com");

  public void fetchWebhook(String rawUrl) {
      URI uri = URI.create(rawUrl);
      if (!ALLOWED_DOMAINS.contains(uri.getHost())) {
          throw new IllegalArgumentException("URL not allowed");
      }
      // Proceed with the fetch
  }
  ```

---

### SPRING-REDIRECT-001: Open redirects MUST be prevented

Severity: Medium

Required:

* MUST NOT redirect to a URL derived directly from user input without validation.
* SHOULD validate redirect targets against an allowlist of permitted domains or enforce that only same-site relative paths are allowed.
* MUST ensure that `RedirectView` and `HttpServletResponse.sendRedirect()` do not receive untrusted data.

Insecure patterns:

* `return "redirect:" + request.getParameter("next")`.
* `response.sendRedirect(request.getParameter("returnUrl"))`.
* OAuth `redirect_uri` accepted from request without validation.

Detection hints:

* Search for `return "redirect:" + `, `response.sendRedirect(`, `RedirectView(`.
* Check whether the argument is derived from request parameters.

Fix:

* Enforce relative paths only or use an allowlist:
  ```java
  public String login(@RequestParam(defaultValue = "/dashboard") String next,
                      HttpServletResponse response) {
      // Only allow same-site relative paths
      if (!next.startsWith("/") || next.startsWith("//")) {
          next = "/dashboard";
      }
      return "redirect:" + next;
  }
  ```

---

### SPRING-FILE-001: File uploads MUST be validated for type, size, and stored safely

Severity: High

Required:

* MUST enforce maximum file size via `spring.servlet.multipart.max-file-size` and `max-request-size`.
* MUST validate file content type using both the declared content type and actual magic bytes/content inspection.
* MUST NOT use the client-supplied filename directly — sanitize using a safe filename generator or ignore it entirely.
* MUST NOT serve uploaded files from a location where they could be executed (e.g., a public web root with CGI).
* SHOULD store uploads outside the application deployment directory.

Insecure patterns:

* `file.getOriginalFilename()` used directly in a path without sanitization.
* No file size limit set.
* Accepting `application/x-php`, `text/html`, `application/x-sh` content types.
* Storing files at a path like `/var/www/public/uploads/` where web server might execute them.

Detection hints:

* Search for `getOriginalFilename()` usage in file persistence code.
* Check `application*.yml` / `application*.properties` for `max-file-size`, `max-request-size`.
* Inspect upload storage logic; check if paths are user-controlled.

Fix:

* Sanitize filename and validate type:
  ```java
  public String saveFile(MultipartFile file) throws IOException {
      // Validate content type against allowlist
      String contentType = file.getContentType();
      Set<String> allowed = Set.of("image/jpeg", "image/png", "application/pdf");
      if (!allowed.contains(contentType)) {
          throw new IllegalArgumentException("File type not allowed");
      }

      // Use a randomly generated filename, not the client-supplied name
      String filename = UUID.randomUUID() + getExtensionForType(contentType);
      Path dest = uploadDir.resolve(filename).normalize();

      // Ensure the destination is within the upload directory
      if (!dest.startsWith(uploadDir)) {
          throw new SecurityException("Path traversal detected");
      }
      Files.copy(file.getInputStream(), dest, StandardCopyOption.REPLACE_EXISTING);
      return filename;
  }
  ```

---

### SPRING-FILE-002: File downloads MUST prevent path traversal

Severity: High

Required:

* MUST normalize and validate file paths before serving files to ensure they resolve within the intended directory.
* MUST NOT resolve user-supplied filenames relative to a base directory without normalization.
* SHOULD use `Resource` abstraction and verify the resolved path starts with the expected base path.

Insecure patterns:

* `new File(baseDir + "/" + userFilename)` without `normalize()` / `getCanonicalPath()`.
* `Paths.get(uploadDir, userFilename)` where `userFilename` contains `../` sequences.
* Serving files by path without checking the canonical path is within the upload directory.

Detection hints:

* Search for `new File(`, `Paths.get(`, `new ClassPathResource(` with user-supplied components.
* Search for file serving endpoints that accept a `filename` or `path` request parameter.

Fix:

* Validate canonical path:
  ```java
  public Resource loadFile(String filename) throws IOException {
      Path file = uploadDir.resolve(filename).normalize();
      // Ensure path stays within uploadDir
      if (!file.startsWith(uploadDir.normalize())) {
          throw new SecurityException("Access denied: path traversal attempt");
      }
      Resource resource = new UrlResource(file.toUri());
      if (!resource.exists() || !resource.isReadable()) {
          throw new ResponseStatusException(HttpStatus.NOT_FOUND);
      }
      return resource;
  }
  ```

---

### SPRING-HEADERS-001: Security response headers MUST be configured

Severity: Medium

Required:

* MUST enable Spring Security's default security headers (enabled by default; verify they are not disabled).
* SHOULD customize `Content-Security-Policy` appropriate to the application.
* SHOULD set `X-Frame-Options: DENY` or `SAMEORIGIN` to prevent clickjacking (Spring Security default).
* MUST set `X-Content-Type-Options: nosniff` (Spring Security default).
* SHOULD set `Referrer-Policy: strict-origin-when-cross-origin`.

Insecure patterns:

* `.headers(AbstractHttpConfigurer::disable)` — disables all Spring Security default headers.
* `.headers(h -> h.frameOptions(HeadersConfigurer.FrameOptionsConfig::disable))` without a justification.
* Explicitly setting `Content-Security-Policy: default-src *`.

Detection hints:

* Search for `.headers(AbstractHttpConfigurer::disable)`, `headers().disable()`.
* Search for `frameOptions().disable()`.
* Check if `spring-security-web` is in the classpath and `SecurityFilterChain` is configured.

Fix:

* Keep Spring Security's default headers enabled and add CSP:
  ```java
  .headers(headers -> headers
      .contentSecurityPolicy(csp -> csp
          .policyDirectives("default-src 'self'; script-src 'self'; style-src 'self'"))
      .frameOptions(frame -> frame.deny())
      .referrerPolicy(ref -> ref
          .policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN))
  )
  ```

---

### SPRING-CORS-001: CORS MUST be configured strictly; wildcard origins with credentials are prohibited

Severity: High

Required:

* MUST NOT configure `allowedOrigins("*")` alongside `allowCredentials(true)` — browsers block this, and it indicates a misconfiguration.
* MUST use explicit origin allowlists (`allowedOrigins("https://app.example.com")`).
* SHOULD restrict `allowedMethods` and `allowedHeaders` to the minimum needed.
* MUST ensure CORS configuration is applied at the `SecurityFilterChain` level (or at least via `WebMvcConfigurer`) — not only per-controller annotations without global config review.

Insecure patterns:

* `config.addAllowedOrigin("*")` globally.
* `@CrossOrigin(origins = "*")` on a controller handling authenticated data.
* `allowedOriginPatterns("*")` with `allowCredentials(true)`.

Detection hints:

* Search for `addAllowedOrigin("*")`, `@CrossOrigin(origins = "*")`, `allowedOriginPatterns("*")`.
* Search `CorsConfigurationSource` beans and `WebMvcConfigurer.addCorsMappings`.

Fix:

* Use explicit origins:
  ```java
  @Bean
  public CorsConfigurationSource corsConfigurationSource() {
      CorsConfiguration config = new CorsConfiguration();
      config.setAllowedOrigins(List.of("https://app.example.com"));
      config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
      config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
      config.setAllowCredentials(true);
      UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
      source.registerCorsConfiguration("/api/**", config);
      return source;
  }
  ```

---

### SPRING-SECRETS-001: Secrets MUST NOT be hard-coded in source code or committed to version control

Severity: Critical

Required:

* MUST load all secrets (database passwords, JWT signing keys, API keys, OAuth client secrets) from environment variables, a secret manager (Vault, AWS Secrets Manager, etc.), or encrypted config files.
* MUST NOT hard-code secrets in `application.properties`, `application.yml`, Java source, or test fixtures committed to the repository.
* SHOULD use Spring Boot's `@ConfigurationProperties` with environment variable overrides for secret injection.
* MUST NOT log secret values; use redaction for any sensitive properties.

Insecure patterns:

* `spring.datasource.password=mysecretpassword` in a committed `application.properties`.
* `private static final String JWT_SECRET = "hardcoded-secret-key"`.
* `spring.security.oauth2.client.registration.*.client-secret=abc123` committed to repo.
* Logging `environment.getProperty("spring.datasource.password")`.

Detection hints:

* Search for passwords/keys in `application*.yml`, `application*.properties`.
* Search Java source for `String.*SECRET`, `String.*PASSWORD`, `String.*API_KEY` constants.
* Inspect `@Value("${...}")` injection points for secrets that do not use environment variable placeholders.
* Check `.gitignore` for `application-prod.yml` or similar secret files.

Fix:

* Reference environment variables:
  ```yaml
  spring:
    datasource:
      password: ${DB_PASSWORD}  # injected at runtime from environment
  jwt:
    secret: ${JWT_SECRET}
  ```
* For local dev, use `.env` files that are `.gitignore`d.

---

### SPRING-DESERIAL-001: Java serialization and Jackson polymorphic typing MUST be handled safely

Severity: Critical

Required:

* MUST NOT expose Java serialization (`ObjectInputStream`) endpoints to untrusted data — this is a well-known RCE vector.
* MUST NOT enable Jackson's global default typing (`mapper.enableDefaultTyping()` / `MapperFeature.DEFAULT_VIEW_INCLUSION`) on untrusted JSON.
* SHOULD disable or carefully audit `@JsonTypeInfo(use = Id.CLASS)` or `@JsonTypeInfo(use = Id.MINIMAL_CLASS)` as these can enable deserialization gadget chains.
* SHOULD use Jackson modules (`jackson-modules-java8`, etc.) rather than custom deserializers for common types.

Insecure patterns:

* `objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL)`.
* `@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS, ...)` on types deserialized from user input.
* `ObjectInputStream ois = new ObjectInputStream(request.getInputStream())`.
* Accepting serialized Java objects via HTTP endpoints.

Detection hints:

* Search for `enableDefaultTyping`, `activateDefaultTyping`, `DefaultTyping`.
* Search for `@JsonTypeInfo(use = Id.CLASS)`, `use = Id.MINIMAL_CLASS`.
* Search for `ObjectInputStream(` with request body or network data as input.

Fix:

* Disable global default typing and use explicit, limited polymorphism:
  ```java
  ObjectMapper mapper = new ObjectMapper();
  // DO NOT call: mapper.enableDefaultTyping(...)

  // If polymorphism is needed, use a strict named-type allowlist:
  PolymorphicTypeValidator ptv = BasicPolymorphicTypeValidator.builder()
      .allowIfSubType("com.example.myapp.dto.")
      .build();
  mapper.activateDefaultTyping(ptv, ObjectMapper.DefaultTyping.NON_FINAL);
  ```

---

### SPRING-SUPPLY-001: Dependencies MUST be kept up to date and scanned for vulnerabilities

Severity: Medium

NOTE: Upgrading dependencies can break projects. Flag findings and let the user decide — do not auto-upgrade unless asked.

Required:

* MUST use `spring-boot-starter-parent` or `spring-boot-dependencies` BOM to keep dependency versions aligned.
* SHOULD run OWASP Dependency-Check (`dependency-check-maven` or `dependency-check-gradle`) in CI.
* MUST treat `CRITICAL` and `HIGH` CVEs in direct dependencies as security-blocking.
* SHOULD keep the Spring Boot version within a supported release line.

Insecure patterns:

* Overriding `spring-boot-starter-parent` dependency versions with known-vulnerable older versions.
* No vulnerability scanning in CI pipeline.
* Spring Boot version more than two major/minor versions behind current.

Detection hints:

* Inspect `pom.xml` / `build.gradle` for `<version>` overrides on Spring Security, Spring Web, Tomcat, Netty, Log4j.
* Check CI configs for `dependency-check`, `snyk`, `trivy`, or `grype` scans.
* Check if `log4j-core` is a transitive dependency with a vulnerable version (CVE-2021-44228 series).

Fix:

* Add OWASP Dependency-Check to Maven:
  ```xml
  <plugin>
      <groupId>org.owasp</groupId>
      <artifactId>dependency-check-maven</artifactId>
      <version>9.x.x</version>
      <configuration>
          <failBuildOnCVSS>7</failBuildOnCVSS>
      </configuration>
  </plugin>
  ```

---

### SPRING-PROXY-001: Reverse proxy and forwarded header trust MUST be explicit

Severity: High

Required:

* MUST NOT trust `X-Forwarded-For`, `X-Forwarded-Proto`, `X-Forwarded-Host` from the open internet without configuring a trusted proxy.
* MUST use Spring's `ForwardedHeaderFilter` or `server.forward-headers-strategy` only when behind a confirmed trusted reverse proxy.
* MUST ensure absolute URL generation, cookie `Secure` flags, and redirect logic use the correct scheme/host.

Insecure patterns:

* `server.forward-headers-strategy=NATIVE` or `FRAMEWORK` when not behind a trusted proxy.
* Trusting `X-Forwarded-For` for rate limiting or IP-based access control without proxy trust configuration.
* Using `request.getScheme()` for redirect logic without confirming forwarded headers are trusted.

Detection hints:

* Search `application*.yml` / `application*.properties` for `forward-headers-strategy`.
* Search for `X-Forwarded-For`, `X-Forwarded-Proto` in custom filter/interceptor code.
* Check if IP-based access control reads from forwarded headers.

Fix:

* Enable forwarded header handling only if behind a confirmed trusted proxy:
  ```yaml
  server:
    forward-headers-strategy: framework  # Only enable when behind a trusted reverse proxy
  ```
* Or add the filter explicitly and configure trusted proxies at the network/WAF level.

---

### SPRING-LOGGING-001: Sensitive data MUST NOT be logged

Severity: High

Required:

* MUST NOT log passwords, tokens (JWT, API keys, session IDs), credit card numbers, national IDs, or other PII.
* MUST NOT log full request bodies on auth endpoints.
* SHOULD use structured logging (SLF4J + Logback/Log4j2) with a masking/redaction pattern for sensitive fields.
* MUST ensure `spring.jpa.show-sql=true` does not log sensitive query parameters in production.

Insecure patterns:

* `log.debug("User login attempt: username={}, password={}", username, password)`.
* `log.info("JWT token: {}", token)`.
* `spring.jpa.show-sql=true` in production with sensitive data in queries.
* Logging entire `HttpServletRequest` including headers (may include Authorization header).

Detection hints:

* Search log statements for `password`, `token`, `secret`, `Authorization`, `credential`.
* Search `application*.yml` for `show-sql=true` in the production profile.
* Inspect request/response logging filters for full-body logging.

Fix:

* Use masking in log statements:
  ```java
  log.info("User login attempt: username={}", username); // Never log password
  log.debug("JWT issued for user: {} (token redacted)", userId);
  ```
* Configure Logback masking patterns for sensitive fields.
