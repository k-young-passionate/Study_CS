# Spring

## 관련 도구

### JMeter
- 서비스 성능 분석/측정 도구

### Thymeleaf
- Java XML, XHTML, HTML5 [템플릿 엔진](../Terms/Terms.md#Template-engine)



## Annotations

### @Autowired
- 타입에 해당하는 Bean 주입
- `생성자`, `setter`, `필드` 가능
- ```Java
    @Autowired
	public BookService(BookRepository books) {
		this.books = books;
        return books.findAll();
	} // BookRepository type의 books값으로 생성자 주입
  ```

### @SpringBootApplication
- 기본적인 설정 선언
- main [method](../Terms/Terms.md#Method)가 정의된 class에서 실행
- 설정 목록
    - @EnableAutoConfiguration
    - @ComponentScan
    - @Configuration

### @Entity
- 객체 - Table 매핑 ([ORM](../Terms/Terms.md#ORM))
- [@Table](#@Table)과 함께 사용
- 객체 담당
- [JPA](../Terms/Terms.md#JPA)가 관리

### @Table
- 객체 - Table 매핑 ([ORM](../Terms/Terms.md#ORM))
- [@Entity](#@Entity)과 함께 사용
- 테이블 담당
- table 이름, 스키마, 카탈로그 등 설정 가능

### @Column
- Table의 Column 매핑

### @Lombok
- 객체 생성 시, 필요한 [메소드](../Terms/Terms.md#Method), 생성자 생성

### @OneToOne, @OneToMany, @ManyToOne, @ManyToMany
- 1:1, 1:N, N:1, N:M 관계로 join된 table 객체 생성
- [@JoinTable](#@JoinTable)과 함꼐 사용

### @JoinTable
- table `join`에 사용
- 속성
    - `name`: join할 table 이름
    - `joinColumns`: 현제 Entity를 참고하는 `Foreign Key`
    - `inversejoinColumns`: 현재 Entity가 참고하는 `Foreign Key`

### @Controller
- view 반환 목적의 controller

### @RequestMapping

### @GetMapping, @PostMapping

### @RequestParam

### @PathVariable

### RestController
- [@Controller](#@Controller)에 [@ResponseBody](@ResponseBody) 추가
- Json 형태로 반환하는 것 목적

### @ResponseBody
