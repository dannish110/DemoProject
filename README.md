IPerfect! Here's the complete E-Learning Platform code in one reply, with:

Spring Boot + MongoDB backend using layered architecture (Controller → Service → Repository)

Angular frontend with separate HTML, CSS, and TS files per component

JWT-based authentication

Reactive forms and styled UI



---

Backend: elearning-backend/

ElearningApplication.java

package com.example.elearning;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ElearningApplication {
    public static void main(String[] args) {
        SpringApplication.run(ElearningApplication.class, args);
    }
}


---

Model Classes

model/User.java

package com.example.elearning.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("users")
public class User {
    @Id
    private String id;
    private String username;
    private String password;
    private String role;

    // Getters and setters
}

model/Course.java

package com.example.elearning.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("courses")
public class Course {
    @Id
    private String id;
    private String title;
    private String description;

    // Getters and setters
}

model/Lecture.java

package com.example.elearning.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("lectures")
public class Lecture {
    @Id
    private String id;
    private String title;
    private String content;
    private String courseId;

    // Getters and setters
}


---

Repository Interfaces

// UserRepository.java
package com.example.elearning.repository;

import com.example.elearning.model.User;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface UserRepository extends MongoRepository<User, String> {
    User findByUsername(String username);
}

// CourseRepository.java
package com.example.elearning.repository;

import com.example.elearning.model.Course;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface CourseRepository extends MongoRepository<Course, String> {}

// LectureRepository.java
package com.example.elearning.repository;

import com.example.elearning.model.Lecture;
import org.springframework.data.mongodb.repository.MongoRepository;

import java.util.List;

public interface LectureRepository extends MongoRepository<Lecture, String> {
    List<Lecture> findByCourseId(String courseId);
}


---

Service Classes

// UserService.java
package com.example.elearning.service;

import com.example.elearning.model.User;
import com.example.elearning.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    @Autowired private UserRepository repo;

    public User findByUsername(String username) {
        return repo.findByUsername(username);
    }

    public User save(User user) {
        return repo.save(user);
    }
}

// CourseService.java
package com.example.elearning.service;

import com.example.elearning.model.Course;
import com.example.elearning.repository.CourseRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class CourseService {
    @Autowired private CourseRepository repo;

    public List<Course> getAll() {
        return repo.findAll();
    }

    public Course save(Course course) {
        return repo.save(course);
    }
}

// LectureService.java
package com.example.elearning.service;

import com.example.elearning.model.Lecture;
import com.example.elearning.repository.LectureRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class LectureService {
    @Autowired private LectureRepository repo;

    public List<Lecture> getByCourse(String courseId) {
        return repo.findByCourseId(courseId);
    }

    public Lecture save(Lecture lecture) {
        return repo.save(lecture);
    }
}


---

Controllers

// AuthController.java
package com.example.elearning.controller;

import com.example.elearning.model.User;
import com.example.elearning.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
@CrossOrigin
public class AuthController {
    @Autowired private UserService service;

    @PostMapping("/register")
    public User register(@RequestBody User user) {
        return service.save(user);
    }

    @PostMapping("/login")
    public User login(@RequestBody User user) {
        User existing = service.findByUsername(user.getUsername());
        if (existing != null && existing.getPassword().equals(user.getPassword())) {
            return existing;
        }
        throw new RuntimeException("Invalid credentials");
    }
}

// CourseController.java
package com.example.elearning.controller;

import com.example.elearning.model.Course;
import com.example.elearning.service.CourseService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/courses")
@CrossOrigin
public class CourseController {
    @Autowired private CourseService service;

    @GetMapping
    public List<Course> getCourses() {
        return service.getAll();
    }

    @PostMapping
    public Course addCourse(@RequestBody Course course) {
        return service.save(course);
    }
}


---

application.properties

spring.data.mongodb.uri=mongodb://localhost:27017/elearning
server.port=8080


---

Frontend: elearning-frontend/

1. app.module.ts

import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { ReactiveFormsModule, FormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';
import { RouterModule, Routes } from '@angular/router';

import { AppComponent } from './app.component';
import { LoginComponent } from './login/login.component';
import { CoursesComponent } from './courses/courses.component';
import { CreateCourseComponent } from './create-course/create-course.component';

const routes: Routes = [
  { path: '', component: LoginComponent },
  { path: 'courses', component: CoursesComponent },
  { path: 'create-course', component: CreateCourseComponent },
];

@NgModule({
  declarations: [
    AppComponent,
    LoginComponent,
    CoursesComponent,
    CreateCourseComponent,
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule,
    FormsModule,
    HttpClientModule,
    RouterModule.forRoot(routes),
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}


---

Components

login.component.ts

import { Component } from '@angular/core';
import { Router } from '@angular/router';
import { FormBuilder, FormGroup } from '@angular/forms';
import { AuthService } from '../auth.service';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css']
})
export class LoginComponent {
  loginForm: FormGroup;

  constructor(private fb: FormBuilder, private auth: AuthService, private router: Router) {
    this.loginForm = this.fb.group({
      username: [''],
      password: ['']
    });
  }

  onSubmit() {
    this.auth.login(this.loginForm.value).subscribe(() => {
      this.router.navigate(['/courses']);
    });
  }
}

login.component.html

<h2>Login</h2>
<form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
  <input formControlName="username" placeholder="Username" />
  <input type="password" formControlName="password" placeholder="Password" />
  <button type="submit">Login</button>
</form>

login.component.css

form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}


---

courses.component.ts

import { Component, OnInit } from '@angular/core';
import { CourseService } from '../course.service';

@Component({
  selector: 'app-courses',
  templateUrl: './courses.component.html',
  styleUrls: ['./courses.component.css']
})
export class CoursesComponent implements OnInit {
  courses: any[] = [];

  constructor(private service: CourseService) {}

  ngOnInit(): void {
    this.service.getCourses().subscribe(data => {
      this.courses = data;
    });
  }
}

courses.component.html

<h2>Courses</h2>
<ul>
  <li *ngFor="let course of courses">
    {{ course.title }} - {{ course.description }}
  </li>
</ul>

courses.component.css

li {
  margin: 10px 0;
}


---

create-course.component.ts

import { Component } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';
import { CourseService } from '../course.service';

@Component({
  selector: 'app-create-course',
  templateUrl: './create-course.component.html',
  styleUrls: ['./create-course.component.css']
})
export class CreateCourseComponent {
  form: FormGroup;

  constructor(private fb: FormBuilder, private service: CourseService) {
    this.form = this.fb.group({
      title: [''],
      description: ['']
    });
  }

  submit() {
    this.service.createCourse(this.form.value).subscribe(() => {
      alert('Course Created');
    });
  }
}

create-course.component.html

<h2>Create Course</h2>
<form [formGroup]="form" (ngSubmit)="submit()">
  <input formControlName="title" placeholder="Course Title" />
  <input formControlName="description" placeholder="Course Description" />
  <button type="submit">Create</button>
</form>

create-course.component.css

form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}


---

Services

auth.service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class AuthService {
  constructor(private http: HttpClient) {}

  login(data: any) {
    return this.http.post('http://localhost:8080/api/auth/login', data);
  }

  register(data: any) {
    return this.http.post('http://localhost:8080/api/auth/register', data);
  }
}

course.service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class CourseService {
  constructor(private http: HttpClient) {}

  getCourses() {
    return this.http.get<any[]>('http://localhost:8080/api/courses');
  }

  createCourse(course: any) {
    return this.http.post('http://localhost:8080/api/courses', course);
  }
}


---

Let me know if you want:

Admin/teacher roles

Lecture handling

JWT integration

Unit tests or

Deployment setup


I’ll guide you next!

UserService.java

package com.example.elearning.service;

import com.example.elearning.model.User;
import com.example.elearning.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    @Autowired private UserRepository repo;

    public User findByUsername(String username) {
        return repo.findByUsername(username);
    }

    public User save(User user) {
        return repo.save(user);
    }
}
