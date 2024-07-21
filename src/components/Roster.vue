<template>
  <div class="container">
    <header>
      <h1>Course Management System</h1>
      <nav>
        <button @click="currentPage = 'courses'">Courses</button>
        <button @click="currentPage = 'add-course'">Add Course</button>
      </nav>
    </header>

    <div v-if="currentPage === 'courses'">
      <h2>Course List</h2>
      <ul>
        <li v-for="course in courses" :key="course.code">
          <button @click="selectCourse(course)">{{ course.code }}: {{ course.title }}</button>
          <button @click="deleteCourse(course)">Delete</button>
        </li>
      </ul>
    </div>

    <div v-if="currentPage === 'add-course'">
      <h2>Add a New Course</h2>
      <form @submit.prevent="addCourse">
        <input v-model="newCourseCode" placeholder="Enter course code" required />
        <input v-model="newCourseTitle" placeholder="Enter course title" required />
        <input v-model="newCourseDescription" placeholder="Enter course description" required />
        <button type="submit">Add Course</button>
      </form>
    </div>

    <div v-if="currentPage === 'course-details'">
      <h2>Course Details</h2>
      <h1 id="course-title">{{ selectedCourse.code }}: {{ selectedCourse.title }}</h1>
      <p id="course-description">{{ selectedCourse.description }}</p>
      <p id="course-teacher">
        <span v-if="selectedCourse.teacher">
          {{ selectedCourse.teacher.honorific }} {{ selectedCourse.teacher.name }}
          <button @click="deleteTeacher">Delete Teacher</button>
        </span>
        <span v-else>Not Set</span>
      </p>
      
      <div>
        <h3>Add Student</h3>
        <form @submit.prevent="addStudent">
          <input v-model="studentName" placeholder="Enter Student full name" required />
          <input v-model="studentEmail" type="email" placeholder="Enter student email" required />
          <button type="submit">Add Student</button>
        </form>
      </div>

      <div>
        <h3>Add Teacher</h3>
        <form @submit.prevent="setTeacher">
          <input v-model="teacherName" placeholder="Enter full teacher name" required />
          <input v-model="teacherEmail" type="email" placeholder="Enter teacher email" required />
          <input v-model="teacherHonorific" placeholder="Enter honorific" required />
          <button type="submit">Add Teacher</button>
        </form>
      </div>

      <table id="roster">
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Attendance</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="student in selectedCourse.students" :key="student.username">
            <td>{{ student.name }}</td>
            <td>{{ student.email }}</td>
            <td>{{ student.calculateAttendance() }}</td>
            <td>
              <button class="present" @click="markAttendance(student.username)">Present</button>
              <button class="absent" @click="markAttendance(student.username, 'absent')">Absent</button>
              <button @click="deleteStudent(student.username)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';

class Person {
  constructor(name, email) {
    this.name = name;
    this.email = email;
    this.username = email.split('@')[0];
  }
}

class Student extends Person {
  constructor(name, email) {
    super(name, email);
    this.attendance = [];
  }
  calculateAttendance() {
    if (this.attendance.length > 0) {
      let counter = 0;
      for (let mark of this.attendance) {
        counter += mark;
      }
      let attendancePercentage = (counter / this.attendance.length) * 100;
      return `${attendancePercentage.toFixed(2)}%`;
    } else {
      return '0%';
    }
  }
}

class Teacher extends Person {
  constructor(name, email, honorific) {
    super(name, email);
    this.honorific = honorific;
  }
}

class Course {
  constructor(courseCode, courseTitle, courseDescription) {
    this.code = courseCode;
    this.title = courseTitle;
    this.description = courseDescription;
    this.teacher = null;
    this.students = [];
  }

  addStudent(name, email) {
    let newStudent = new Student(name, email);
    this.students.push(newStudent);
  }

  setTeacher(name, email, honorific) {
    this.teacher = new Teacher(name, email, honorific);
  }

  markAttendance(username, status = "present") {
    let foundStudent = this.findStudent(username);
    if (status === "present") {
      foundStudent.attendance.push(1);
    } else {
      foundStudent.attendance.push(0);
    }
  }

  findStudent(username) {
    return this.students.find(student => student.username === username);
  }

  deleteStudent(username) {
    this.students = this.students.filter(student => student.username !== username);
  }

  deleteTeacher() {
    this.teacher = null;
  }
}

const currentPage = ref('courses');
const courses = reactive([]);
const selectedCourse = ref(null);

const newCourseCode = ref('');
const newCourseTitle = ref('');
const newCourseDescription = ref('');

const studentName = ref('');
const studentEmail = ref('');
const teacherName = ref('');
const teacherEmail = ref('');
const teacherHonorific = ref('');

const updateCourseInfo = () => {
  selectedCourse.value.updateCourseInfo(newCourseCode.value, newCourseTitle.value, newCourseDescription.value);
  newCourseCode.value = '';
  newCourseTitle.value = '';
  newCourseDescription.value = '';
};

const addCourse = () => {
  const newCourse = new Course(newCourseCode.value, newCourseTitle.value, newCourseDescription.value);
  courses.push(newCourse);
  newCourseCode.value = '';
  newCourseTitle.value = '';
  newCourseDescription.value = '';
  currentPage.value = 'courses';
};

const selectCourse = (course) => {
  selectedCourse.value = course;
  currentPage.value = 'course-details';
};

const addStudent = () => {
  selectedCourse.value.addStudent(studentName.value, studentEmail.value);
  studentName.value = '';
  studentEmail.value = '';
};

const setTeacher = () => {
  selectedCourse.value.setTeacher(teacherName.value, teacherEmail.value, teacherHonorific.value);
  teacherName.value = '';
  teacherEmail.value = '';
  teacherHonorific.value = '';
};

const markAttendance = (username, status = "present") => {
  selectedCourse.value.markAttendance(username, status);
};

const deleteCourse = (course) => {
  if (confirm(`Are you sure you want to delete the course ${course.title}?`)) {
    courses.splice(courses.indexOf(course), 1);
    if (selectedCourse.value === course) {
      selectedCourse.value = null;
      currentPage.value = 'courses';
    }
  }
};

const deleteStudent = (username) => {
  if (confirm(`Are you sure you want to delete the student ${username}?`)) {
    selectedCourse.value.deleteStudent(username);
  }
};

const deleteTeacher = () => {
  if (confirm(`Are you sure you want to delete the teacher?`)) {
    selectedCourse.value.deleteTeacher();
  }
};
</script>

<style scoped>
.container {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: auto;
  padding: 20px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

nav button {
  margin: 0 5px;
  padding: 5px 10px;
  border: none;
  background-color: #007bff;
  color: white;
  cursor: pointer;
  border-radius: 5px;
}

nav button:hover {
  background-color: #0056b3;
}

form {
  margin-bottom: 20px;
}

form input {
  margin: 5px 0;
  padding: 10px;
  width: 100%;
  box-sizing: border-box;
}

form button {
  padding: 10px;
  width: 100%;
  background-color: #28a745;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

form button:hover {
  background-color: #218838;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

button.present, button.absent, button.delete {
  margin: 0 2px;
  padding: 5px 10px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

button.present {
  background-color: #28a745;
  color: white;
}

button.absent {
  background-color: #dc3545;
  color: white;
}

button.delete {
  background-color: #ffc107;
  color: white;
}

button.present:hover {
  background-color: #218838;
}

button.absent:hover {
  background-color: #c82333;
}

button.delete:hover {
  background-color: #e0a800;
}
</style>

