# CORS

**CORS** — это правило браузера, которое говорит:<br>

    Этот сайт не может просто так читать ответы от другого сайта, если тот явно не разрешил это».

- Это не защита сервера,
- Это правило браузера.
- Сервер может отвечать всегда,
- Браузер решает: показывать ответ JS-коду или нет (JavaScript-коду на странице A разрешено получить доступ к содержимому HTTP-ответа от сайта B.).

## Как сервер «разрешает»

Сервер добавляет HTTP-заголовки, например:

    Access-Control-Allow-Origin: https://frontend.com

## Почему это вообще нужно

Без CORS:
- Любой сайт мог бы:
- читать твои данные
- использовать твои cookies
- делать запросы от твоего имени

CORS защищает пользователя, а не сервер.

---

## Где реально задаётся CORS

Только в HTTP-ответе сервера, через заголовки:

* Access-Control-Allow-Origin
* Access-Control-Allow-Methods
* Access-Control-Allow-Headers
* Access-Control-Allow-Credentials
* Access-Control-Expose-Headers

---

## Варианты настройки в Spring

### CrossOrigin

    @RestController
    @RequestMapping("/api")
    @CrossOrigin(
        origins = "https://frontend.com",
        allowCredentials = "true"
    )
    public class ApiController {
    }

Факты:
* Работает только для браузеров
* Добавляет нужные CORS-заголовки
* Подходит для простых случаев

Минусы:
* плохо масштабируется
* легко забыть
* не видно общей политики

### WebMvcConfigurer

      @Configuration
      public class CorsConfig implements WebMvcConfigurer {
    
          @Override
          public void addCorsMappings(CorsRegistry registry) {
              registry.addMapping("/api/**")
                  .allowedOrigins("https://frontend.com")
                  .allowedMethods("GET", "POST", "PUT", "DELETE")
                  .allowedHeaders("*")
                  .allowCredentials(true);
          }
      }

Факты:
* Централизованная политика
* Корректно обрабатывает preflight OPTIONS
* Чаще всего лучшее решение

### Spring Security

    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .cors(Customizer.withDefaults())
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
        return http.build();
    }
    
    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("https://frontend.com"));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
        config.setAllowedHeaders(List.of("*"));
        config.setAllowCredentials(true);
    
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        return source;
    }

Факты:
* Это фильтр, работает раньше контроллеров
* Обязательно для защищённых API
* Частая причина «CORS не работает»