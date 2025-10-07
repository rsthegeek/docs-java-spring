# Spring Boot Tutorial for Beginners [2025]
- [source](https://www.youtube.com/watch?v=gJrjgg1KVL4)

## Auto build
- add Spring dev tools
- enable auto build
- enable auto make even while running

## Using resources/application.properties
- @Value("${app.name}")

What is Annotations?!

----
# Building a rest api with spring boot (SpringAcademy)
- To test a rest endpoint annotate test with `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)`
- This will allow us to make HTTP requests to the locally running application:
    ```java
        @Autowired
        TestRestTemplate restTemplate;
    ```
- What is `java.lang.Number` Class?

## Pagination
`/cashcards?page=1&size=3&sort=amount,desc`
```java
@GetMapping
private ResponseEntity<List<CashCard>> findAll(Pageable pageable) {
    Page<CashCard> page = cashCardRepository.findAll(
      PageRequest.of(
        pageable.getPageNumber(),
        pageable.getPageSize(),
        pageable.getSortOr(Sort.by(Sort.Direction.DESC, "amount"))
      )
    );
  return ResponseEntity.ok(page.getContent());
}
```

- Json paths
  - `$.length()` -> int
  - `$..id` -> JSONArray (get `id` field of an array of objects)
  - `$[*]` gets JSONArray instance of objects.
  - `$[0].amount` reads amount field of the object at index 0 of array.
- New Test method annotations:
  - `@DirtiesContext` for the method
  - `@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)` for the class
- Sending PUT request with `TestRestTemplate`:
    ```java
        CashCard kumarsCard = new CashCard(null, 333.33, null);
        HttpEntity<CashCard> request = new HttpEntity<>(kumarsCard);
        ResponseEntity<Void> response = restTemplate
                .withBasicAuth("sarah1", "abc123")
                .exchange("/cashcards/102", HttpMethod.PUT, request, Void.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    ```
- Steel Thread
- Red, Green, Refactor