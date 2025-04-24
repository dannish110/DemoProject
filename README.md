// ========================= BACKEND (Spring Boot + MongoDB + JWT) =========================

// 1. pom.xml dependencies <dependencies> <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-web</artifactId> </dependency> <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-security</artifactId> </dependency> <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-data-mongodb</artifactId> </dependency> <dependency> <groupId>io.jsonwebtoken</groupId> <artifactId>jjwt</artifactId> <version>0.9.1</version> </dependency> </dependencies>

// 2. User.java @Document("users") public class User { @Id private String id; private String username; private String email; private String password; private String role; // STUDENT, TEACHER, ADMIN }

// 3. Course.java @Document("courses") public class Course { @Id private String id; private String title; private String description; private String teacherId; }

// 4. Lecture.java @Document("lectures") public class Lecture { @Id private String id; private String title; private String content; private String courseId; }

// 5. UserRepository.java public interface UserRepository extends MongoRepository<User, String> { Optional<User> findByUsername(String username); }

// 6. JWTUtil.java @Component public class JWTUtil { private final String secret = "secretkey";

public String generateToken(User user) {
    return Jwts.builder()
        .setSubject(user.getUsername())
        .claim("role", user.getRole())
        .signWith(SignatureAlgorithm.HS512, secret)
        .compact();
}

public String getUsername(String token) {
    return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody().getSubject();
}

public boolean validateToken(String token) {
    try {
        Jwts.parser().setSigningKey(secret).parseClaimsJws(token);
        return true;
    } catch (Exception e) {
        return false;
    }
}

}

// 7. UserService.java @Service public class UserService implements UserDetailsService { @Autowired private UserRepository userRepo; @Autowired private PasswordEncoder encoder;

public User register(User user) {
    user.setPassword(encoder.encode(user.getPassword()));
    return userRepo.save(user);
}

public User authenticate(String username, String rawPassword) {
    User user = userRepo.findByUsername(username).orElseThrow();
    if (encoder.matches(rawPassword, user.getPassword())) return user;
    else throw new RuntimeException("Invalid credentials");
}

@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepo.findByUsername(username).orElseThrow();
    return new org.springframework.security.core.userdetails.User(
        user.getUsername(), user.getPassword(), List.of(new SimpleGrantedAuthority("ROLE_" + user.getRole())));
}

}

// 8. AuthController.java @RestController @RequestMapping("/api/auth") public class AuthController { @Autowired private UserService userService; @Autowired private JWTUtil jwtUtil;

@PostMapping("/register")
public ResponseEntity<User> register(@RequestBody User user) {
    return ResponseEntity.ok(userService.register(user));
}

@PostMapping("/login")
public ResponseEntity<Map<String, String>> login(@RequestBody Map<String, String> body) {
    User user = userService.authenticate(body.get("username"), body.get("password"));
    String token = jwtUtil.generateToken(user);
    return ResponseEntity.ok(Map.of("token", token, "role", user.getRole(), "id", user.getId()));
}

}

// 9. JWTFilter.java @Component public class JWTFilter extends OncePerRequestFilter { @Autowired private JWTUtil jwtUtil; @Autowired private UserService userService;

@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
        throws ServletException, IOException {
    String authHeader = request.getHeader("Authorization");
    if (authHeader != null && authHeader.startsWith("Bearer ")) {
        String token = authHeader.substring(7);
        if (jwtUtil.validateToken(token)) {
            String username = jwtUtil.getUsername(token);
            UserDetails userDetails = userService.loadUserByUsername(username);
            UsernamePasswordAuthenticationToken auth =
                new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
    }
    chain.doFilter(request, response);
}

}

// 10. SecurityConfig.java @Configuration public class SecurityConfig { @Autowired private JWTFilter jwtFilter;

@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf().disable()
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**").permitAll()
            .requestMatchers("/api/courses/**", "/api/lectures/**").hasAnyRole("TEACHER", "ADMIN")
            .anyRequest().authenticated())
        .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
    return http.build();
}

@Bean
public PasswordEncoder encoder() {
    return new BCryptPasswordEncoder();
}

}

// ========================= FRONTEND (Angular 15) =========================

// 1. Angular File Structure src/ ├── app/ │   ├── services/ │   │   └── auth.service.ts │   ├── components/ │   │   ├── login/ │   │   │   ├── login.component.ts/html/css │   │   ├── course-list/ │   │   │   ├── course-list.component.ts/html/css │   ├── app-routing.module.ts │   ├── app.module.ts │   └── app.component.ts/html

// 2. auth.service.ts @Injectable({ providedIn: 'root' }) export class AuthService { private baseUrl = 'http://localhost:8080/api/auth';

constructor(private http: HttpClient) {}

login(credentials: any): Observable<any> { return this.http.post(${this.baseUrl}/login, credentials); }

register(user: any): Observable<any> { return this.http.post(${this.baseUrl}/register, user); }

isLoggedIn(): boolean { return !!localStorage.getItem('token'); } }

// 3. login.component.ts @Component({ selector: 'app-login', templateUrl: './login.component.html' }) export class LoginComponent { username = ''; password = '';

constructor(private auth: AuthService, private router: Router) {}

login() { this.auth.login({ username: this.username, password: this.password }).subscribe( res => { localStorage.setItem('token', res.token); localStorage.setItem('role', res.role); this.router.navigate(['/courses']); }, err => alert('Login failed') ); } }

// 4. login.component.html <input [(ngModel)]="username" placeholder="Username"> <input [(ngModel)]="password" type="password" placeholder="Password"> <button (click)="login()">Login</button>

// 5. app-routing.module.ts const routes: Routes = [ { path: 'login', component: LoginComponent }, { path: 'courses', component: CourseListComponent }, { path: '', redirectTo: '/login', pathMatch: 'full' } ];

@NgModule({ imports: [RouterModule.forRoot(routes)], exports: [RouterModule] }) export class AppRoutingModule {}

// 6. app.component.html

<nav>
  <a routerLink="/login">Login</a>
  <a routerLink="/courses" *ngIf="isLoggedIn()">Courses</a>
</nav>
<router-outlet></router-outlet>// 7. course-list.component.ts @Component({ selector: 'app-course-list', templateUrl: './course-list.component.html' }) export class CourseListComponent implements OnInit { courses: any[] = [];

ngOnInit() { const token = localStorage.getItem('token'); this.http.get('http://localhost:8080/api/courses', { headers: { Authorization: Bearer ${token} } }).subscribe(data => this.courses = data); }

constructor(private http: HttpClient) {} }

// 8. course-list.component.html

<ul>
  <li *ngFor="let course of courses">
    {{ course.title }} - {{ course.description }}
  </li>
</ul>// ========================= END =========================

Let me know if you want registration, logout, or role-based routing too!

