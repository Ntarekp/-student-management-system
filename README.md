# student-management-system
Hello, I am super excited to review and deepen my programming skills through projects, Today I am focusing on structures and file handling, by developing a student management system. I am again excited to document to you what it can do.

---

## **Step 1: Understanding the Basics**

### **What is a Structure in C?**
In C, a **structure** is a way to group different pieces of data (like numbers, text, etc.) under a single name. Think of it as a container that can hold multiple items, each with its own type and purpose. For example, if you want to store information about a student—like their name, semester number, and grades—you can use a structure to keep all these details together.

Here’s how we define a structure in C:
```c
struct Student {
    char name[50];  // A string to store the student's name (up to 49 characters)
    int semester;   // An integer for the current semester
    int class_roll; // An integer for the class roll number
};
```
- `struct` is the keyword to create a structure.
- `Student` is the name of the structure (you can choose any name).
- Inside the curly braces `{}`, we list the **members** (like `name`, `semester`, `class_roll`), each with its own data type (`char`, `int`, etc.).
- The semicolon `;` at the end is required.

To use this structure, you create a **variable** of this type:
```c
struct Student student1;
```
Now, `student1` can hold a name, semester, and class roll. You access or set these members using the dot (`.`) operator:
```c
strcpy(student1.name, "Alice"); // Set the name
student1.semester = 3;          // Set the semester
student1.class_roll = 101;      // Set the class roll
```

### **What is File Handling in C?**
File handling lets us store data (like our structures) in a file on our computer, so it’s saved even after the program ends. We can:
- **Write** data to a file (e.g., save student info).
- **Read** data from a file (e.g., load student info).
- **Update** or **delete** data in a file.

In C, we use the `FILE` type and functions like:
- `fopen()`: Opens a file (e.g., `"rb+"` to read and write in binary mode).
- `fwrite()`: Writes data to a file.
- `fread()`: Reads data from a file.
- `fseek()`: Moves the file pointer to a specific position.
- `fclose()`: Closes the file.

We’ll use binary files (not plain text) because they let us write structures directly, making it easier to manage complex data.

---

## **Step 2: Planning the Student Management System**

The project requires a system to store information about **students**, **teachers**, and **staff**, with specific details for students:
- **Name**: A string (text).
- **Semester**: An integer (e.g., 1, 2, 3).
- **Class Roll**: An integer (unique within a class).
- **Exam Roll**: An integer (unique identifier for exams).
- **Course Name and Grade Point Earned in the Current Semester**: Suggests multiple courses, each with a name and grade (e.g., 4.0, 3.7).
- **GPA Earned in Previous Semesters**: GPA for each past semester.
- **CGPA**: Cumulative GPA across all semesters.

For **teachers** and **staff**, the query doesn’t specify fields, so we’ll assume basic info like name, department, and an ID.

We need to:
1. **Define structures** for students, teachers, and staff.
2. Use **file handling** to save and manage this data.
3. Implement operations: **add**, **search**, **change**, and **remove**.

Since the file will store all three types (students, teachers, staff), we’ll use a **wrapper structure** with a type indicator to distinguish them.

---

## **Step 3: Designing the Structures**

### **Structure for Courses**
Students take multiple courses in the current semester, each with a name and grade point. Let’s define a structure for a course:
```c
struct Course {
    char name[50];    // Course name (e.g., "Math")
    float grade_point; // Grade point (e.g., 4.0 for an A)
};
```

### **Structure for Students**
For students, we need all the specified fields. Since they have multiple courses and previous GPAs, we’ll use arrays with a maximum size:
```c
struct Student {
    char name[50];              // Student's name
    int semester;               // Current semester (e.g., 3)
    int class_roll;             // Class roll number
    int exam_roll;              // Exam roll number (unique)
    struct Course current_courses[10]; // Array for up to 10 courses
    int num_current_courses;    // Number of courses taken this semester
    float previous_gpas[20];    // Array for GPAs of previous semesters (up to 20)
    int num_previous_semesters; // Number of previous semesters
    float cgpa;                 // Cumulative GPA
};
```
- `current_courses[10]`: Assumes a max of 10 courses per semester.
- `num_current_courses`: Tracks how many courses are actually used.
- `previous_gpas[20]`: Assumes a max of 20 semesters (more than enough).
- `num_previous_semesters`: Equals `semester - 1`, but we’ll store it for clarity.

### **Structure for Teachers and Staff**
Since the query doesn’t specify fields, let’s assume:
- **Teacher**:
```c
struct Teacher {
    char name[50];      // Teacher's name
    char department[50]; // Department (e.g., "Computer Science")
    char position[50];   // Position (e.g., "Professor")
    int employee_id;     // Unique ID
};
```
- **Staff**:
```c
struct Staff {
    char name[50];      // Staff's name
    char department[50]; // Department
    char role[50];       // Role (e.g., "Technician")
    int employee_id;     // Unique ID
};
```

### **Wrapper Structure for the File**
To store all three types in one file, we use a structure with a `type` field and a **union** (a union lets us store one of several types in the same memory space):
```c
struct Record {
    int type;  // 0: student, 1: teacher, 2: staff, -1: deleted
    union {
        struct Student student;
        struct Teacher teacher;
        struct Staff staff;
    } data;
};
```
- `type`: Indicates the record type or if it’s deleted.
- `union data`: Holds either a `student`, `teacher`, or `staff`. The file will reserve space for the largest type (`struct Student`).

---

## **Step 4: Calculating CGPA for Students**
The CGPA is the average GPA across all semesters, including the current one. Here’s how we’ll compute it:
1. **Current Semester GPA**: Average of `grade_point` values in `current_courses`.
   - Formula: `(sum of grade_points) / num_current_courses`
2. **Total GPA**: Sum of all previous GPAs plus the current GPA.
3. **CGPA**: `(sum of previous_gpas + current_gpa) / semester`

For simplicity, we assume each course has equal weight (e.g., 1 credit), and each semester contributes equally to the CGPA.

---

## **Step 5: Implementing File Handling Operations**

### **File Setup**
We’ll use a binary file `"database.bin"`:
- Open with `"rb+"` for reading and writing.
- If the file doesn’t exist, create it with `"wb"`, then reopen with `"rb+"`.
- Each record is `sizeof(struct Record)` bytes.

### **Add Operation**
To add a record:
1. Ask the user for the type (student, teacher, or staff).
2. Collect the data.
3. Write the `struct Record` to the file.

Example for adding a student:
- Input: name, semester, rolls, courses, previous GPAs.
- Calculate CGPA.
- Set `type = 0` and write.

### **Search Operation**
To search (e.g., by name):
1. Open the file with `"rb"`.
2. Read each `struct Record`.
3. If `type != -1` and the name matches, display the details.

### **Change Operation**
To change (e.g., a student by `exam_roll`):
1. Open with `"rb+"`.
2. Find the record by reading sequentially.
3. Update the fields, recalculate CGPA if needed.
4. Seek back and overwrite.

### **Remove Operation**
To remove:
1. Find the record.
2. Set `type = -1`.
3. Write back (marks it as deleted).

---


### **Notes on the Code**
- **Input Limitation**: `scanf("%s")` reads single words. For names with spaces, you’d need `fgets()`, but we’ll keep it simple.
- **ID Uniqueness**: The program assumes unique IDs; in practice, you’d check for duplicates.
- **Error Handling**: Basic; you could add more checks (e.g., file opening errors).
