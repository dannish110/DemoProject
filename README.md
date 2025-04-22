Problem Statement:

Design and develop a Learning Management System (LMS) to facilitate seamless interaction between Admins, Teachers, and Students. The LMS should provide a structured platform for course creation, lecture delivery, assignment management, and student progress tracking.

The system will be divided into two major components:

1. Backend using Spring Boot:

Implement role-based access control for Admins, Teachers, and Students.

Admins should be able to manage users and approve or assign courses.

Teachers should be able to create, update, and delete courses, lectures, and assignments.

Students should be able to enroll in courses, view lectures, and submit assignments.

Integrate MongoDB for data persistence.

Follow a layered architecture (Controller → Service → Repository).

Include proper validation, error handling, and unit testing.



2. Frontend using Angular:

Provide dedicated dashboards for Admins, Teachers, and Students.

Enable reactive forms for user input and validations.

Implement responsive UI with separate HTML, CSS, and TS files for each component.

Ensure smooth navigation and user-friendly experience for course browsing, enrollment, lecture viewing, and assignment submission.