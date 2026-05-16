# Guía Completa de Configuración y Uso de Spring Security (JWT)

Esta guía detalla la implementación de seguridad basada en **JSON Web Tokens (JWT)** en el proyecto, incluyendo el código fuente de cada componente y su explicación técnica.

---

## 1. Configuración de Dependencias (`pom.xml`)

Para habilitar la seguridad y el soporte de JWT, se incluyeron las siguientes dependencias:

```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- JWT (jjwt) -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

---

## 2. Configuración de Propiedades (`application.properties`)

Es necesario configurar una clave secreta y el tiempo de expiración para los tokens:

```properties
# JWT Configuration
jwt.secret=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
jwt.expiration=86400000 # 24 horas en milisegundos
```

---

## 3. Lógica de JWT: `JwtService`

Esta clase gestiona la creación, extracción y validación de los tokens.

**Ubicación:** `src/main/java/com/example/demo_basic/config/JwtService.java`

```java
@Service
public class JwtService {
    // Métodos para generar token, extraer subject (username) y validar expiración
    // Utiliza HS256 para firmar los tokens con la clave secreta
}
```

---

## 4. Filtro de Autenticación: `JwtAuthenticationFilter`

Este filtro intercepta cada petición HTTP para validar la presencia de un token válido en la cabecera `Authorization`.

**Ubicación:** `src/main/java/com/example/demo_basic/config/JwtAuthenticationFilter.java`

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    // 1. Extrae el token del header "Authorization: Bearer <token>"
    // 2. Valida el token con JwtService
    // 3. Carga el usuario y lo establece en el SecurityContext
}
```

---

## 5. Controlador de Autenticación: `AuthController`

Permite a los usuarios obtener un token enviando sus credenciales.

**Ubicación:** `src/main/java/com/example/demo_basic/controller/AuthController.java`

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        // Valida credenciales y devuelve un token JWT si son correctas
    }
}
```

---

## 6. Configuración Central: `SecurityConfig`

Aquí se integra el filtro JWT y se define la política de seguridad como **Stateless**.

**Ubicación:** `src/main/java/com/example/demo_basic/config/SecurityConfig.java`

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll() // Permite login público
                .requestMatchers("/api-docs/**", "/swagger-ui/**", "/scalar/**", "/").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/**").hasAnyRole("ADMIN", "USER")
                .requestMatchers(HttpMethod.POST, "/api/**").hasRole("ADMIN")
                // ... otras reglas
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

---

## 7. Cómo realizar peticiones con JWT

A diferencia de la autenticación básica, ahora el proceso consta de dos pasos:

### Paso 1: Obtener el Token (Login)
**POST** `/auth/login`
```json
{
    "username": "admin",
    "password": "admin123"
}
```
**Respuesta:**
```json
{
    "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

### Paso 2: Usar el Token en rutas protegidas
Debes incluir el token en la cabecera `Authorization` con el prefijo `Bearer`.

**Ejemplo cURL:**
```bash
curl -H "Authorization: Bearer <TU_TOKEN>" http://localhost:8080/api/adoptantes
```

**Ejemplo Postman:**
1. Selecciona la pestaña **Authorization**.
2. Tipo: **Bearer Token**.
3. Pega el token obtenido en el paso 1.

---
### Resumen de Permisos por Rol
| Ruta / Método | Rol `USER` | Rol `ADMIN` |
| :--- | :---: | :---: |
| `GET /api/**` | ✅ Permitido | ✅ Permitido |
| `POST/PUT/DELETE /api/**` | ❌ Denegado | ✅ Permitido |

---
*Configuración actualizada a JWT por Antigravity AI.*
