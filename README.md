# DemoProject
Got it! Let's make a **super simple full-stack app** — like a **"Hello World" Task App** with **just create and view** functionality. Minimal code. Here's the smallest meaningful example:

---

## 🔧 Backend (Spring Boot)
### ✅ Features:
- Expose 2 APIs:
  - `GET /tasks` — Get all tasks
  - `POST /tasks` — Add a task

---

### 📁 Structure:
```
springboot-mini/
└── src/main/java/com/example/demo/
    ├── Task.java
    ├── TaskController.java
    └── DemoApplication.java
```

### 🔸 `Task.java`
```java
package com.example.demo;

public class Task {
    public String name;
    public Task() {}
    public Task(String name) { this.name = name; }
}
```

### 🔹 `TaskController.java`
```java
package com.example.demo;

import org.springframework.web.bind.annotation.*;
import java.util.*;

@RestController
@CrossOrigin
public class TaskController {
    List<Task> tasks = new ArrayList<>();

    @GetMapping("/tasks")
    public List<Task> getTasks() {
        return tasks;
    }

    @PostMapping("/tasks")
    public Task addTask(@RequestBody Task task) {
        tasks.add(task);
        return task;
    }
}
```

### 🧱 `DemoApplication.java`
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### ⚙️ `application.properties`
```properties
# Empty (uses default port 8080)
```

---

## 🖼️ Frontend (Angular)
### ✅ Features:
- Input task name
- Show task list

---

### 📄 `app.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent implements OnInit {
  taskName = '';
  tasks: any[] = [];

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.loadTasks();
  }

  loadTasks() {
    this.http.get<any[]>('http://localhost:8080/tasks').subscribe(data => this.tasks = data);
  }

  addTask() {
    if (!this.taskName.trim()) return;
    this.http.post('http://localhost:8080/tasks', { name: this.taskName })
      .subscribe(() => {
        this.taskName = '';
        this.loadTasks();
      });
  }
}
```

### 📄 `app.component.html`
```html
<input [(ngModel)]="taskName" placeholder="Enter task">
<button (click)="addTask()">Add Task</button>

<ul>
  <li *ngFor="let task of tasks">{{ task.name }}</li>
</ul>
```

### 🧱 `app.module.ts` (only changes)
```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';
import { FormsModule } from '@angular/forms';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule, FormsModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

## 🚀 Run It

### 1. Spring Boot
```bash
cd springboot-mini
./mvnw spring-boot:run
```

### 2. Angular
```bash
cd angular-app
npm install
ng serve
```

Then open: `http://localhost:4200`

---

Want me to send a downloadable zip for both frontend & backend?
