# Security Filter Chain Internals

Фундамент Spring Security в веб-приложениях построен на обычных Servlet Filters.

1. **DelegatingFilterProxy**: Сервлет-контейнер (например, Tomcat) ничего не знает о бинах Spring. Чтобы связать мир сервлетов с миром Spring, используется этот прокси-фильтр. Он перехватывает все запросы и передает их специальному бину в контексте Spring.
2. **FilterChainProxy**: Тот самый бин, которому передается управление. Это ядро маршрутизации Spring Security. Он содержит список SecurityFilterChain (цепочек фильтров).
3. **Как выбирается цепочка:** У каждой SecurityFilterChain есть RequestMatcher (например, /api/). FilterChainProxy перебирает цепочки сверху вниз и отправляет запрос в первую совпавшую.
4. **Внутри цепочки:** Содержится от 10 до 30+ стандартных фильтров, расположенных в строго заданном порядке.
   - **SecurityContextPersistenceFilter** (восстанавливает контекст из сессии, если она есть).
   - **Фильтры аутентификации** (например, UsernamePasswordAuthenticationFilter или BearerTokenAuthenticationFilter).
   - **ExceptionTranslationFilter** (ловит AccessDeniedException и AuthenticationException, превращая их в HTTP 401 или 403).
   - **FilterSecurityInterceptor / AuthorizationFilter** (последний рубеж, проверяет права доступа к конкретному URL).

### Session management and stateless JWT configuration

Традиционные веб-приложения хранят состояние сессии на сервере (JSESSIONID в куках). Микросервисы и REST API требуют архитектуры без состояния (Stateless).

**Настройка Stateless**
Чтобы сказать Spring Security не создавать сессии, нужно явно указать это в конфигурации SecurityFilterChain:<br>
`http.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))`

---

## UserDetailsService и UserDetails

Это контракты для интеграции вашей базы пользователей со Spring Security. Spring Security абсолютно не волнует, как выглядит ваша таблица users в базе данных. Ему нужен только адаптер.

- UserDetails: Это интерфейс, описывающий минимально необходимую информацию о пользователе для системы безопасности.
  - Содержит методы: getUsername(), getPassword(), getAuthorities() (роли/права).
  - Флаги состояния аккаунта: isAccountNonExpired(), isAccountNonLocked(), isCredentialsNonExpired(), isEnabled().
- UserDetailsService: DAO-интерфейс (Data Access Object) всего с одним методом: loadUserByUsername(String username).

_Как это работает:_ Когда DaoAuthenticationProvider пытается проверить пароль, он вызывает UserDetailsService, получает объект UserDetails, а затем под капотом использует PasswordEncoder, чтобы сравнить хэш пароля из БД с тем паролем, который прислал пользователь.

--- 

## Method-level security: @PreAuthorize, @Secured

Безопасность можно настраивать не только на уровне URL (в конфигурации фильтров), но и на уровне бизнес-логики (Service Layer). Это реализуется через Spring AOP (Аспектно-Ориентированное Программирование).

- **@Secured**: Устаревшая (legacy) аннотация из стандарта Java. Поддерживает только простые проверки ролей (например, @Secured("ROLE_ADMIN")). Не умеет работать со сложными выражениями.
- **@PreAuthorize / @PostAuthorize**: Современный подход. Поддерживает SpEL (Spring Expression Language), что дает невероятную гибкость.
    - _Пример:_ @PreAuthorize("hasRole('ADMIN') or #user.id == authentication.principal.id") — доступ разрешен админу ИЛИ если пользователь запрашивает свои собственные данные.
    - **@PostAuthorize** позволяет выполнить метод, а затем проверить результат (удобно, если права зависят от того, какой объект вернулся из БД).

**Важно:** Поскольку безопасность методов построена на AOP-прокси, вызов защищенного метода из другого метода того же самого класса (внутренний вызов) проигнорирует аннотации безопасности. Прокси срабатывает только при вызове извне.