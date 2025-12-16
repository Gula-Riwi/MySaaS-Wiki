# Arquitectura Técnica - MeetLines Users Microservice

## Descripción General

MeetLines Users es un microservicio basado en **Spring Boot** que gestiona autenticación, sincronización de usuarios y control de acceso en el ecosistema MeetLines.

## Stack Tecnológico

| Tecnología | Versión | Propósito |
|-----------|---------|----------|
| Spring Boot | 3.0+ | Framework Java |
| Spring Security | 6.0+ | Seguridad y autenticación |
| Keycloak | 20+ | Identity Provider (OAuth2) |
| PostgreSQL | 15+ | Base de datos |
| Maven | 3.8+ | Gestión de dependencias |
| Docker | 20.10+ | Containerización |
| JWT | - | JSON Web Tokens |

## Estructura del Proyecto

```
MeetLines-Users/
├── src/
│   ├── main/
│   │   ├── java/com/Gula/MeetLines/
│   │   │   ├── auth/              # Autenticación y seguridad
│   │   │   │   ├── config/        # Configuración de seguridad
│   │   │   │   ├── controller/    # Endpoints de auth
│   │   │   │   ├── service/       # Servicios de auth
│   │   │   │   ├── dto/           # DTOs
│   │   │   │   └── exception/     # Excepciones personalizadas
│   │   │   ├── user/              # Gestión de usuarios
│   │   │   │   ├── entity/        # Entidades JPA
│   │   │   │   ├── repository/    # Repositorios
│   │   │   │   ├── service/       # Servicios de usuario
│   │   │   │   └── controller/    # Endpoints REST
│   │   │   ├── oauth/             # Integración OAuth
│   │   │   │   ├── discord/
│   │   │   │   ├── facebook/
│   │   │   │   └── google/
│   │   │   ├── sync/              # Sincronización de usuarios
│   │   │   │   └── service/
│   │   │   ├── config/            # Configuración global
│   │   │   ├── filter/            # Filtros HTTP
│   │   │   └── exception/         # Manejo de excepciones
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── application-dev.properties
│   │       ├── application-prod.properties
│   │       ├── db/migration/      # Flyway migrations
│   │       └── templates/         # Templates para emails
│   └── test/
│       └── java/...               # Tests unitarios e integración
├── pom.xml                        # Configuración Maven
├── Dockerfile                     # Docker image
├── docker-compose.yml             # Local development
└── README.md
```

## Autenticación y Seguridad

### Flujo de Autenticación

```
┌─────────────┐
│ Usuario     │
└──────┬──────┘
       │ Credentials
       ▼
┌──────────────────────┐
│ Login Endpoint       │
│ POST /auth/login     │
└──────┬───────────────┘
       │ Validate
       ▼
┌──────────────────────┐
│ Keycloak OAuth2      │
│ Token Generation     │
└──────┬───────────────┘
       │ JWT Token
       ▼
┌──────────────────────┐
│ Return Access Token  │
│ + Refresh Token      │
└──────────────────────┘
```

### Configuración de Seguridad

```java
// SecurityConfig.java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.decoder(jwtDecoder()))
            )
            .httpBasic();
        
        return http.build();
    }

    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withJwtProcessorProvider(
            JwkSetUriJwtProcessorProvider.withLazyInit(jwtIssuerUri)
        ).build();
    }
}
```

## Endpoints Principales

### Autenticación

#### Login
```http
POST /api/Auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password"
}

Response 200:
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "refresh_token_here",
  "expiresIn": 3600,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe"
  }
}
```

#### Registrar
```http
POST /api/Auth/register
Content-Type: application/json

{
  "email": "newuser@example.com",
  "password": "secure_password",
  "firstName": "John",
  "lastName": "Doe",
  "businessName": "My Business"
}
```

#### Refresh Token
```http
POST /api/Auth/refresh-token
Content-Type: application/json

{
  "refreshToken": "refresh_token_here"
}
```

#### OAuth (Discord)
```http
POST /api/Auth/oauth/discord
Content-Type: application/json

{
  "code": "discord_authorization_code",
  "redirectUri": "https://yourdomain.com/callback"
}
```

### Usuarios

#### Obtener Perfil
```http
GET /api/Profile
Authorization: Bearer TOKEN
```

#### Actualizar Perfil
```http
PUT /api/Profile
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "firstName": "John",
  "lastName": "Doe",
  "phoneNumber": "+34612345678"
}
```

#### Cambiar Contraseña
```http
PUT /api/Profile/change-password
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "currentPassword": "old_password",
  "newPassword": "new_secure_password"
}
```

## Entidades de Base de Datos

### User Entity

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String passwordHash;
    
    @Column(nullable = false)
    private String firstName;
    
    @Column(nullable = false)
    private String lastName;
    
    @Column
    private String phoneNumber;
    
    @Column
    private String profilePhotoUrl;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserStatus status;
    
    @Column(nullable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime createdAt;
    
    @Column
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime updatedAt;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<UserOAuth> oauthAccounts;
}
```

## Servicios

### AuthService

```java
@Service
public class AuthService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Autowired
    private JwtTokenProvider jwtTokenProvider;
    
    public LoginResponse login(LoginRequest request) {
        User user = userRepository.findByEmail(request.getEmail())
            .orElseThrow(() -> new AuthException("Invalid credentials"));
        
        if (!passwordEncoder.matches(request.getPassword(), user.getPasswordHash())) {
            throw new AuthException("Invalid credentials");
        }
        
        String accessToken = jwtTokenProvider.generateAccessToken(user);
        String refreshToken = jwtTokenProvider.generateRefreshToken(user);
        
        return new LoginResponse(accessToken, refreshToken, 3600, user);
    }
    
    public User register(RegisterRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new AuthException("Email already registered");
        }
        
        User user = new User();
        user.setEmail(request.getEmail());
        user.setPasswordHash(passwordEncoder.encode(request.getPassword()));
        user.setFirstName(request.getFirstName());
        user.setLastName(request.getLastName());
        user.setStatus(UserStatus.ACTIVE);
        user.setCreatedAt(LocalDateTime.now());
        
        return userRepository.save(user);
    }
}
```

## Migraciones de Base de Datos

```sql
-- V1__Initial_Schema.sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone_number VARCHAR(20),
    profile_photo_url VARCHAR(500),
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP,
    
    CONSTRAINT unique_email UNIQUE(email)
);

CREATE TABLE user_oauth (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    provider VARCHAR(50) NOT NULL,
    provider_user_id VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT unique_oauth UNIQUE(provider, provider_user_id)
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_user_oauth_user_id ON user_oauth(user_id);
```

## Configuración en Producción

### docker-compose.yml (Producción)

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: meetlines_users
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - meetlines-network

  keycloak:
    image: keycloak/keycloak:latest
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin_password
      DB_VENDOR: postgres
      DB_ADDR: postgres
      DB_DATABASE: keycloak
      DB_USER: postgres
      DB_PASSWORD: secure_password
    ports:
      - "8080:8080"
    networks:
      - meetlines-network

  users-service:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/meetlines_users
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secure_password
      KEYCLOAK_SERVER_URL: http://keycloak:8080
    ports:
      - "8081:8081"
    depends_on:
      - postgres
      - keycloak
    networks:
      - meetlines-network

networks:
  meetlines-network:
    driver: bridge

volumes:
  postgres-data:
```

## Testing

```java
@SpringBootTest
@AutoConfigureMockMvc
public class AuthControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private AuthService authService;
    
    @Test
    public void testLoginSuccess() throws Exception {
        LoginRequest request = new LoginRequest();
        request.setEmail("test@example.com");
        request.setPassword("password");
        
        mockMvc.perform(post("/api/Auth/login")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isOk());
    }
}
```

## Conclusión

El microservicio Users proporciona:

✅ Autenticación segura con JWT  
✅ Integración OAuth con Discord, Facebook, Google  
✅ Gestión de perfiles de usuario  
✅ Sincronización de usuarios entre servicios  
✅ Manejo de errores y excepciones  
✅ Testing completo  

## Referencias

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Security Documentation](https://spring.io/projects/spring-security)
- [Keycloak Documentation](https://www.keycloak.org/documentation.html)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
