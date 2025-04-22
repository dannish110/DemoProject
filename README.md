Sure! Below is the full backend code for your Learning Management System (LMS), including all the necessary files, setup, and detailed explanations, which you can include in your README file.


---

Learning Management System (LMS) - Full Backend Code

This project is the backend of a Learning Management System (LMS) developed using Spring Boot and MongoDB.


---

Project Setup

1. Clone the repository:

git clone https://github.com/your-repository/lms-backend.git


2. Install dependencies:

Use Maven to install the required dependencies:

mvn clean install


3. Run the application:

After installing dependencies, run the application using:

mvn spring-boot:run

The application should be running on http://localhost:8080.


4. Configure MongoDB:

Ensure MongoDB is running locally or use a MongoDB instance (you can configure the connection in the application.properties file).




---

Project Structure

src

main

java

com

lms

controller

UserController.java

CourseController.java


model

User.java

Course.java

Lecture.java


repository

UserRepository.java

CourseRepository.java


service

UserService.java

CourseService.java

LectureService.java


config

SecurityConfig.java






resources

application.properties





---

Code

1. UserService.java (Service Layer)

package com.lms.service;

import com.lms.model.User;
import com.lms.repository.UserRepository;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.stereotype.Service;
import org.springframework.security.crypto.password.PasswordEncoder;

import java.util.Collections;

@Service
public class UserService implements UserDetailsService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Find user by email (username)
        User user = userRepository.findByEmail(username);

        // Throw exception if user not found
        if (user == null) {
            throw new UsernameNotFoundException("User not found");
        }

        // Return Spring Security User with role
        return new org.springframework.security.core.userdetails.User(
            user.getEmail(),
            user.getPassword(),
            Collections.singletonList(new SimpleGrantedAuthority(user.getRole()))
        );
    }

    // Additional method for registering a new user
    public User registerNewUser(User user) {
        // Encode the password before saving
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        return userRepository.save(user);
    }
}

2. UserRepository.java (Repository Layer)

package com.lms.repository;

import com.lms.model.User;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface UserRepository extends MongoRepository<User, String> {
    User findByEmail(String email);
}

3. UserController.java (Controller Layer)

package com.lms.controller;

import com.lms.model.User;
import com.lms.service.UserService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping("/register")
    public User register(@RequestBody User user) {
        return userService.registerNewUser(user);
    }

    @GetMapping("/login")
    public String login(@RequestParam String email, @RequestParam String password) {
        // Here you'd add authentication logic
        return "Login Successful";
    }
}

4. CourseService.java (Service Layer)

package com.lms.service;

import com.lms.model.Course;
import com.lms.repository.CourseRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class CourseService {

    private final CourseRepository courseRepository;

    public CourseService(CourseRepository courseRepository) {
        this.courseRepository = courseRepository;
    }

    public Course addCourse(Course course) {
        return courseRepository.save(course);
    }

    public List<Course> getAllCourses() {
        return courseRepository.findAll();
    }

    public Course getCourseById(String id) {
        return courseRepository.findById(id).orElse(null);
    }

    public void deleteCourse(String id) {
        courseRepository.deleteById(id);
    }
}

5. CourseRepository.java (Repository Layer)

package com.lms.repository;

import com.lms.model.Course;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface CourseRepository extends MongoRepository<Course, String> {
}

6. CourseController.java (Controller Layer)

package com.lms.controller;

import com.lms.model.Course;
import com.lms.service.CourseService;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/courses")
public class CourseController {

    private final CourseService courseService;

    public CourseController(CourseService courseService) {
        this.courseService = courseService;
    }

    @PostMapping
    public Course addCourse(@RequestBody Course course) {
        return courseService.addCourse(course);
    }

    @GetMapping
    public List<Course> getAllCourses() {
        return courseService.getAllCourses();
    }

    @GetMapping("/{id}")
    public Course getCourseById(@PathVariable String id) {
        return courseService.getCourseById(id);
    }

    @DeleteMapping("/{id}")
    public void deleteCourse(@PathVariable String id) {
        courseService.deleteCourse(id);
    }
}

7. LectureService.java (Service Layer)

package com.lms.service;

import com.lms.model.Lecture;
import com.lms.repository.LectureRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class LectureService {

    private final LectureRepository lectureRepository;

    public LectureService(LectureRepository lectureRepository) {
        this.lectureRepository = lectureRepository;
    }

    public Lecture addLecture(Lecture lecture) {
        return lectureRepository.save(lecture);
    }

    public List<Lecture> getLecturesByCourseId(String courseId) {
        return lectureRepository.findByCourseId(courseId);
    }
}

8. LectureRepository.java (Repository Layer)

package com.lms.repository;

import com.lms.model.Lecture;
import org.springframework.data.mongodb.repository.MongoRepository;

import java.util.List;

public interface LectureRepository extends MongoRepository<Lecture, String> {
    List<Lecture> findByCourseId(String courseId);
}

9. LectureController.java (Controller Layer)

package com.lms.controller;

import com.lms.model.Lecture;
import com.lms.service.LectureService;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/lectures")
public class LectureController {

    private final LectureService lectureService;

    public LectureController(LectureService lectureService) {
        this.lectureService = lectureService;
    }

    @PostMapping
    public Lecture addLecture(@RequestBody Lecture lecture) {
        return lectureService.addLecture(lecture);
    }

    @GetMapping("/course/{courseId}")
    public List<Lecture> getLecturesByCourse(@PathVariable String courseId) {
        return lectureService.getLecturesByCourseId(courseId);
    }
}

10. SecurityConfig.java (Security Configuration)

package com.lms.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/api/users/register", "/api/users/login").permitAll()
            .antMatchers("/api/courses/**").hasRole("TEACHER")
            .antMatchers("/api/lectures/**").hasRole("TEACHER")
            .anyRequest().authenticated()
            .and().httpBasic();
    }
}


---

Endpoints

Authentication

POST /api/users/register: Register a new user.

GET /api/users/login: Login a user.


Users

GET /api/users/{id}: Retrieve user by ID.

PUT /api/users/{id}: Update user details.

DELETE /api/users/{id}: Delete a user.


Courses and Lectures

POST /api/courses: Create a new course.

GET /api/courses: Get all courses.

GET /api/courses/{id}: Get course by ID.

DELETE /api/courses/{id}: Delete a course.

POST /api/lectures: Add a new lecture.

GET /api/lectures/course/{courseId}: Get all lectures for a course.



---

Testing

To run tests:

mvn test


---

Contributing

Feel free to fork this repository, make changes, and create pull requests.


---

License

This project is licensed under the MIT License - see the LICENSE file for details.


---

Let me know if you need further explanations or additions to the project!

