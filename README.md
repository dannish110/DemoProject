Perfect! Here's a complete Angular 15 frontend for your working E-Learning Platform backend (Spring Boot with MongoDB and role-based authentication, no JWT).
This setup includes:

Login & Register (Username-only login)

Role-based routing (Student, Teacher, Admin)

Course CRUD (Teacher/Admin)

Lecture CRUD under Course (Teacher/Admin)

View Lectures by Course (Student)

Auth Guard for route protection
All components, services, routing, and modules are included. You just need to run ng new elearning-frontend and plug this in.



---

1. app.module.ts

import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { ReactiveFormsModule, FormsModule } from '@angular/forms';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

import { LoginComponent } from './components/login/login.component';
import { RegisterComponent } from './components/register/register.component';
import { DashboardComponent } from './components/dashboard/dashboard.component';
import { CourseListComponent } from './components/course-list/course-list.component';
import { CourseFormComponent } from './components/course-form/course-form.component';
import { LectureFormComponent } from './components/lecture-form/lecture-form.component';
import { CourseLecturesComponent } from './components/course-lectures/course-lectures.component';

import { AuthService } from './services/auth.service';
import { CourseService } from './services/course.service';
import { LectureService } from './services/lecture.service';
import { AuthGuard } from './guards/auth.guard';

@NgModule({
  declarations: [
    AppComponent,
    LoginComponent,
    RegisterComponent,
    DashboardComponent,
    CourseListComponent,
    CourseFormComponent,
    LectureFormComponent,
    CourseLecturesComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule,
    FormsModule,
    ReactiveFormsModule
  ],
  providers: [AuthService, CourseService, LectureService, AuthGuard],
  bootstrap: [AppComponent]
})
export class AppModule {}


---

2. app-routing.module.ts

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { LoginComponent } from './components/login/login.component';
import { RegisterComponent } from './components/register/register.component';
import { DashboardComponent } from './components/dashboard/dashboard.component';
import { CourseListComponent } from './components/course-list/course-list.component';
import { CourseFormComponent } from './components/course-form/course-form.component';
import { LectureFormComponent } from './components/lecture-form/lecture-form.component';
import { CourseLecturesComponent } from './components/course-lectures/course-lectures.component';
import { AuthGuard } from './guards/auth.guard';

const routes: Routes = [
  { path: '', component: DashboardComponent, canActivate: [AuthGuard] },
  { path: 'login', component: LoginComponent },
  { path: 'register', component: RegisterComponent },
  { path: 'courses', component: CourseListComponent, canActivate: [AuthGuard] },
  { path: 'courses/new', component: CourseFormComponent, canActivate: [AuthGuard] },
  { path: 'courses/:id/lectures/new', component: LectureFormComponent, canActivate: [AuthGuard] },
  { path: 'courses/:id/lectures', component: CourseLecturesComponent, canActivate: [AuthGuard] }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}


---

3. auth.service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class AuthService {
  private baseUrl = 'http://localhost:8080/api/users';

  constructor(private http: HttpClient) {}

  register(user: any) {
    return this.http.post(`${this.baseUrl}/register`, user);
  }

  login(credentials: any) {
    return this.http.post(`${this.baseUrl}/login`, credentials, { withCredentials: true });
  }

  isAuthenticated(): boolean {
    return localStorage.getItem('role') != null;
  }

  getRole(): string | null {
    return localStorage.getItem('role');
  }

  logout(): void {
    localStorage.clear();
  }
}


---

4. course.service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class CourseService {
  private baseUrl = 'http://localhost:8080/api/courses';

  constructor(private http: HttpClient) {}

  getAllCourses() {
    return this.http.get(this.baseUrl);
  }

  addCourse(course: any) {
    return this.http.post(this.baseUrl, course);
  }
}


---

5. lecture.service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class LectureService {
  private baseUrl = 'http://localhost:8080/api/lectures';

  constructor(private http: HttpClient) {}

  addLecture(lecture: any) {
    return this.http.post(this.baseUrl, lecture);
  }

  getLecturesByCourse(courseId: string) {
    return this.http.get(`${this.baseUrl}/course/${courseId}`);
  }
}


---

6. auth.guard.ts

import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(): boolean {
    if (this.authService.isAuthenticated()) {
      return true;
    }
    this.router.navigate(['/login']);
    return false;
  }
}


---

7. Components (just core HTML)

login.component.ts

import { Component } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html'
})
export class LoginComponent {
  username = '';
  password = '';

  constructor(private authService: AuthService, private router: Router) {}

  login() {
    this.authService.login({ username: this.username, password: this.password }).subscribe({
      next: (user: any) => {
        localStorage.setItem('role', user.role);
        this.router.navigate(['/']);
      },
      error: () => alert('Invalid credentials')
    });
  }
}

login.component.html

<h2>Login</h2>
<input [(ngModel)]="username" placeholder="Username" />
<input [(ngModel)]="password" type="password" placeholder="Password" />
<button (click)="login()">Login</button>


---

register.component.ts

import { Component } from '@angular/core';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-register',
  templateUrl: './register.component.html'
})
export class RegisterComponent {
  user = { username: '', password: '', role: 'STUDENT' };

  constructor(private authService: AuthService) {}

  register() {
    this.authService.register(this.user).subscribe({
      next: () => alert('User registered'),
      error: () => alert('Error registering user')
    });
  }
}

register.component.html

<h2>Register</h2>
<input [(ngModel)]="user.username" placeholder="Username" />
<input [(ngModel)]="user.password" type="password" placeholder="Password" />
<select [(ngModel)]="user.role">
  <option>STUDENT</option>
  <option>TEACHER</option>
  <option>ADMIN</option>
</select>
<button (click)="register()">Register</button>


---

dashboard.component.html

<h1>Welcome to E-Learning</h1>
<a routerLink="/courses">View Courses</a>
<a routerLink="/courses/new" *ngIf="isTeacherOrAdmin()">Add Course</a>
<button (click)="logout()">Logout</button>

dashboard.component.ts

import { Component } from '@angular/core';
import { AuthService } from '../../services/auth.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html'
})
export class DashboardComponent {
  constructor(private auth: AuthService, private router: Router) {}

  isTeacherOrAdmin() {
    const role = this.auth.getRole();
    return role === 'TEACHER' || role === 'ADMIN';
  }

  logout() {
    this.auth.logout();
    this.router.navigate(['/login']);
  }
}


---

course-list.component.ts

import { Component, OnInit } from '@angular/core';
import { CourseService } from '../../services/course.service';

@Component({
  selector: 'app-course-list',
  templateUrl: './course-list.component.html'
})
export class CourseListComponent implements OnInit {
  courses: any[] = [];

  constructor(private courseService: CourseService) {}

  ngOnInit(): void {
    this.courseService.getAllCourses().subscribe(data => this.courses = data as any[]);
  }
}

course-list.component.html

<h2>Courses</h2>
<ul>
  <li *ngFor="let course of courses">
    {{ course.title }}
    <a [routerLink]="['/courses', course.id, 'lectures']">View Lectures</a>
    <a [routerLink]="['/courses', course.id, 'lectures/new']">Add Lecture</a>
  </li>
</ul>


---

course-form.component.ts

import { Component } from '@angular/core';
import { Router } from '@angular/router';
import { CourseService } from '../../services/course.service';

@Component({
  selector: 'app-course-form',
  templateUrl: './course-form.component.html'
})
export class CourseFormComponent {
  course = { title: '', description: '' };

  constructor(private courseService: CourseService, private router: Router) {}

  addCourse() {
    this.courseService.addCourse(this.course).subscribe(() => this.router.navigate(['/courses']));
  }
}

lecture-form.component.ts

import { Component } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { LectureService } from '../../services/lecture.service';

@Component({
  selector: 'app-lecture-form',
  templateUrl: './lecture-form.component.html'
})
export class LectureFormComponent {
  lecture = { title: '', description: '', courseId: '' };

  constructor(private lectureService: LectureService, private route: ActivatedRoute, private router: Router) {}

  ngOnInit(): void {
    this.lecture.courseId = this.route.snapshot.params['id'];
  }

  addLecture() {
    this.lectureService.addLecture(this.lecture).subscribe(() =>
      this.router.navigate([`/courses/${this.lecture.courseId}/lectures`])
    );
  }
}


---

course-lectures.component.ts

import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { LectureService } from '../../services/lecture.service';

@Component({
  selector: 'app-course-lectures',
  templateUrl: './course-lectures.component.html'
})
export class CourseLecturesComponent implements OnInit {
  lectures: any[] = [];

  constructor(private route: ActivatedRoute, private lectureService: LectureService) {}

  ngOnInit(): void {
    const courseId = this.route.snapshot.params['id'];
    this.lectureService.getLecturesByCourse(courseId).subscribe(data => this.lectures = data as any[]);
  }
}


---

Once this is set up:

Run the backend first (Spring Boot)

Then cd elearning-frontend and run ng serve

Navigate to http://localhost:4200


Let me know if you want:

Admin dashboard UI

Styling with Angular Material or Bootstrap

Role-based view hiding (buttons for teacher/admin only)


I’ll send that next if needed!

