![display](https://repository-images.githubusercontent.com/699158532/739a6a67-40b5-4eb0-8be6-989cc73c1700)

This guide walks you through the process of building a Spring boot application that uses Spring Security and Spring Data JPA. Applying the new way to **configure** Spring Security without the [WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

## What You Will build
You will build a Spring Boot application with Spring Security basic authentication and Spring Data Jpa for managing the users.

## What You Need
- A favorite text editor or IDE
- JDK 1.8 or later
- Gradle 4+ or Maven 3.2+

## Setup Project With Spring Initializr   

- Navigate to https://start.spring.io

- define the project name example: `spring-boot-security-basic-auth`
- Choose Project **Maven** and the language  **Java**.
- Choose Your **Java** version ex: **17**
- Click add dependencies and select:
  - Spring Web
  - Spring Security
  - Spring Data JPA
  - H2 Database
  - Lombok

- Click Generate.

Unzip the Downloaded Zip and open the Project using your favorite text editor or IDE


## Start the implementation
Define the User Entity, implement the **UserDetails** interface and override the implemented methods
```java
@Entity(name = "users")
@Getter
@Setter
public class User implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstName;

    private String lastName;

    @Column(unique = true)
    private String username;

    private String password;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.emptyList();
    }

    @Override
    public String getPassword() {
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

}
```
Define the User Repository and the query method **findOneByUsername** 

Lean more about query methods [here](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findOneByUsername(String username);
}

```
Define the User Details Service implementation that will manage user authentication.
```java
public class CustomUserDetailsService implements UserDetailsService {

    private UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findOneByUsername(username).orElseThrow(() -> {
            throw new UsernameNotFoundException(username);
        });
    }

}
```
Define the Spring Security Configuration, You will notice that we didn't extend the **WebSecurityConfigurerAdapter** we just define beans at this level
```java
@Configuration
public class SecurityConfiguration {

    /**
     * Define the password encoder.
     * 
     * @return {@link BCryptPasswordEncoder}
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * Define the user service to retrieve and authenticate the user.
     * 
     * @param userRepository
     * @return {@link UserDetails}
     */
    @Bean
    CustomUserDetailsService customUserDetailsService(UserRepository userRepository) {
        return new CustomUserDetailsService(userRepository);
    }

    /**
     * Define the filer chain.
     * 
     * @param http
     * @return {@link SecurityFilterChain}
     * @throws Exception
     */
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(
                (authz) -> authz.antMatchers("/secured").authenticated().anyRequest().permitAll())
                .httpBasic();
        return http.build();
    }

}
```
Define the Rest Controller with some routes

```java
@RestController
public class AppResource {

    @GetMapping("/**")
    public String any() {
        return "Hello World";
    }

    @GetMapping("/secured")
    public String secured(Authentication auth) {
        User user = ((User) auth.getPrincipal());
        return "Hello " + user.getFirstName() + " " + user.getLastName();
    }

}
```

Create `data.sql` under the resources folder to populate our table

```sql
INSERT INTO USERS (FIRST_NAME, LAST_NAME, USERNAME, PASSWORD) VALUES
                              -- password is '123'
    ('john', 'doe', 'admin', '$2a$12$vl2OJQMtIZutojuJVhaRXuLmkyFV1QgE24HqFPWoYTmPOXa6wOUbi');
```

When running the application, by default the `data.sql` will be executed before the entity creation into the database, to prevent that add the following property under the `application.properties`

```properties
spring.jpa.defer-datasource-initialization=true
```

## Testing

Write some test cases to check that the uri  **/secured** is working as expected
 - Test 1 : check with correct credentials
 - Test 2 : check with wrong credentials 

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SpringBootSecurityBasicAuthApplicationTests {

	private final String USERNAME = "admin";
	private final String PASSWORD = "123";
	private final String WRONG_PASSWORD = "bvfsdg";

	@Autowired
	TestRestTemplate template;

	@Test
	public void accessPrivateResourceSuccess() throws Exception {
		ResponseEntity<String> response = template.withBasicAuth(USERNAME, PASSWORD)
				.getForEntity("/secured", String.class);

		assertEquals(HttpStatus.OK, response.getStatusCode());
	}

	@Test
	public void accessPrivateResourceFaildGivingWrongPassword() throws Exception {
		ResponseEntity<String> response = template.withBasicAuth(USERNAME, WRONG_PASSWORD)
				.getForEntity("/secured", String.class);

		assertEquals(HttpStatus.UNAUTHORIZED, response.getStatusCode());
	}

}
```
## Run The Application

Run the Java application as a `SpringBootApplication` with your IDE or use the following command line

```shell
 ./mvnw spring-boot:run
```
Now, you can open the URL below on your browser, default port is `8080` you can set it under the `application.properties`
```
http://localhost:8080/
```
When you access the secured URI **/secured**, a prompt shows up pass in the username & password
 - username : admin
 - password : 123

```
http://localhost:8080/secured
```
## Summary

Congratulations 🎉 ! You've created a Spring Security application with basic authentication using Spring Boot & Spring Data JPA

## Github
The tutorial can be found here on [GitHub](https://github.com/nonestack-blog/spring-security-basic-authentication) 👋

## Blog
Check new tutorials on [nonestack](https://www.nonestack.com) 👋