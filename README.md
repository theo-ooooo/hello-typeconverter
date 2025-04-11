# 🔄 스프링 타입 컨버터

## 1. 기본 타입 컨버터

- `String → Integer`, `Integer → String` 등 기본 컨버터는 스프링이 자동 등록

```java
@GetMapping("/hello")
public String hello(@RequestParam Integer data) {
    log.info("data = {}", data);
    return "ok";
}
```

---

## 2. 컨버터 인터페이스

- 스프링 타입 컨버터 구현을 위해 `org.springframework.core.convert.converter.Converter` 인터페이스 사용

```java
public class StringToIntegerConverter implements Converter<String, Integer> {
    @Override
    public Integer convert(String source) {
        return Integer.valueOf(source);
    }
}
```

---

## 3. 컨버전 서비스 등록

```java
@Configuration
public class ConverterConfig {

    @Bean
    public FormattingConversionService conversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        conversionService.addConverter(new StringToIntegerConverter());
        conversionService.addConverter(new IntegerToStringConverter());
        return conversionService;
    }
}
```

---

## 4. 웹 컨버터 적용

- 컨트롤러 `@RequestParam`, `@ModelAttribute`, `@PathVariable` 등에 자동 적용됨

```java
@GetMapping("/ip-port")
public String ipPort(@RequestParam IpPort ipPort) {
    log.info("ipPort = {}", ipPort);
    return "ok";
}
```

---

## 5. `@InitBinder`를 이용한 지역 컨버터 등록

```java
@InitBinder
public void init(WebDataBinder dataBinder) {
    dataBinder.addCustomFormatter(new IpPortFormatter());
}
```

---

## 6. 포맷터 (Formatter)

- 문자 ↔ 객체 변환
- `Converter`와 유사하지만, **문자열 특화** + 국제화 지원
- `org.springframework.format.Formatter` 인터페이스 사용

```java
public class MyNumberFormatter implements Formatter<Number> {
    @Override
    public Number parse(String text, Locale locale) throws ParseException {
        return NumberFormat.getInstance(locale).parse(text);
    }

    @Override
    public String print(Number object, Locale locale) {
        return NumberFormat.getInstance(locale).format(object);
    }
}
```

---

## 7. `ConversionService vs Formatter`

| 구분              | 설명                                         |
|-------------------|----------------------------------------------|
| `Converter`       | 타입 간 변환 (ex: String ↔ Integer)          |
| `Formatter`       | 문자열 ↔ 객체 변환, 국제화 지원               |
| `ConversionService` | 둘을 함께 등록해서 관리하는 스프링 컨테이너 |

---

## 8. 스프링 기본 컨버터/포맷터 등록 내역

- `DefaultFormattingConversionService`에는 주요 기본 포맷터와 컨버터가 이미 등록되어 있음
  - `String ↔ Boolean`, `String ↔ Number`, `String ↔ Enum`, `Date ↔ String` 등

---

## 9. Formatter 등록 시 유의사항

- `@NumberFormat`, `@DateTimeFormat` 어노테이션을 통해 포맷 지정 가능

```java
@Data
public class Form {
    @NumberFormat(pattern = "###,###")
    private Integer price;

    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private LocalDate date;
}
```

