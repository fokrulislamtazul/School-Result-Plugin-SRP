একদম ঠিক সিদ্ধান্ত 👍
এখন **একটা clean, permanent documentation** থাকলে তুমি future-এ কখনো হারাবে না।

নিচে আমি তোমাকে **FULL DATABASE DOCUMENTATION** দিচ্ছি —
👉 **table create SQL**
👉 **mock data insert SQL**
👉 **rules + notes**

এটা তুমি চাইলে **Notion / Google Doc / README.md** তে রেখে দিতে পারো।

---

# 📘 SCHOOL RESULT SYSTEM

## Database Structure & Mock Data Documentation

---

## 🔑 CORE IDENTIFICATION RULE

একজন student uniquely identify হবে:

```
(class_id, roll_no, academic_year)
```

❌ roll globally unique না
✅ roll class-wise unique

---

# 🗂 DATABASE OVERVIEW

| Table          | Purpose                    |
| -------------- | -------------------------- |
| classes        | Class list                 |
| subjects       | Subject master             |
| class_subjects | Class-wise subject mapping |
| exams          | Exam list                  |
| students       | Student info               |
| marks          | Exam marks                 |

---

# 🧱 1️⃣ `classes` table

### Purpose

Class master table (Class 1–5)

### Create SQL

```sql
CREATE TABLE classes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    class_name VARCHAR(20) NOT NULL
);
```

### Mock Data

```sql
INSERT INTO classes (class_name) VALUES
('Class 1'),
('Class 2'),
('Class 3'),
('Class 4'),
('Class 5');
```

---

# 🧱 2️⃣ `subjects` table

### Purpose

All subjects list

### Create SQL

```sql
CREATE TABLE subjects (
    id INT AUTO_INCREMENT PRIMARY KEY,
    subject_name VARCHAR(50) NOT NULL
);
```

### Mock Data

```sql
INSERT INTO subjects (subject_name) VALUES
('Bangla'),
('Bangla Grammar'),
('English'),
('English Grammar'),
('Math'),
('General Knowledge'),
('Science'),
('Social Science'),
('Religion');
```

---

# 🧱 3️⃣ `class_subjects` table ⭐⭐⭐

### Purpose

Which subjects belong to which class

### Rules

* Class 1–2 → 4 subjects
* Class 3–5 → all subjects

### Create SQL

```sql
CREATE TABLE class_subjects (
    id INT AUTO_INCREMENT PRIMARY KEY,
    class_id INT NOT NULL,
    subject_id INT NOT NULL,
    full_marks INT DEFAULT 100,

    UNIQUE KEY uniq_class_subject (class_id, subject_id),

    FOREIGN KEY (class_id) REFERENCES classes(id),
    FOREIGN KEY (subject_id) REFERENCES subjects(id)
);
```

---

### Mock Data – Class 1 & 2 (limited subjects)

```sql
INSERT INTO class_subjects (class_id, subject_id, full_marks)
SELECT c.id, s.id, 100
FROM classes c
JOIN subjects s
WHERE c.id IN (1,2)
  AND s.subject_name IN (
    'Bangla',
    'English',
    'Math',
    'General Knowledge'
  );
```

---

### Mock Data – Class 3–5 (all subjects)

```sql
INSERT INTO class_subjects (class_id, subject_id, full_marks)
SELECT c.id, s.id, 100
FROM classes c
JOIN subjects s
WHERE c.id IN (3,4,5);
```

---

# 🧱 4️⃣ `exams` table

### Purpose

Exam master

### Create SQL

```sql
CREATE TABLE exams (
    id INT AUTO_INCREMENT PRIMARY KEY,
    exam_name VARCHAR(50) NOT NULL,
    academic_year INT NOT NULL
);
```

### Mock Data

```sql
INSERT INTO exams (exam_name, academic_year) VALUES
('1st Term', 2025),
('2nd Term', 2025),
('Final Exam', 2025);
```

---

# 🧱 5️⃣ `students` table ⭐⭐⭐ MOST IMPORTANT

### Purpose

Student basic information

### Rules

* Same roll allowed in different class
* Same roll + class + year ❌ not allowed

### Create SQL

```sql
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    class_id INT NOT NULL,
    roll_no INT NOT NULL,
    academic_year INT NOT NULL,

    UNIQUE KEY uniq_student (class_id, roll_no, academic_year),

    FOREIGN KEY (class_id) REFERENCES classes(id)
);
```

---

### Mock Data (15 students, 3 per class)

```sql
INSERT INTO students (name, class_id, roll_no, academic_year) VALUES
-- Class 1
('Class1 Student1', 1, 1, 2025),
('Class1 Student2', 1, 2, 2025),
('Class1 Student3', 1, 3, 2025),

-- Class 2
('Class2 Student1', 2, 1, 2025),
('Class2 Student2', 2, 2, 2025),
('Class2 Student3', 2, 3, 2025),

-- Class 3
('Class3 Student1', 3, 1, 2025),
('Class3 Student2', 3, 2, 2025),
('Class3 Student3', 3, 3, 2025),

-- Class 4
('Class4 Student1', 4, 1, 2025),
('Class4 Student2', 4, 2, 2025),
('Class4 Student3', 4, 3, 2025),

-- Class 5
('Class5 Student1', 5, 1, 2025),
('Class5 Student2', 5, 2, 2025),
('Class5 Student3', 5, 3, 2025);
```

---

# 🧱 6️⃣ `marks` table ⭐⭐⭐

### Purpose

Student exam marks

### Rules

* One student → one subject → one exam → one mark
* Draft / Locked system

### Create SQL

```sql
CREATE TABLE marks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT NOT NULL,
    exam_id INT NOT NULL,
    class_subject_id INT NOT NULL,
    marks_obtained INT NOT NULL,
    status ENUM('draft','locked') DEFAULT 'draft',

    UNIQUE KEY uniq_marks (student_id, exam_id, class_subject_id),

    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (exam_id) REFERENCES exams(id),
    FOREIGN KEY (class_subject_id) REFERENCES class_subjects(id)
);
```

---

### Mock Marks Insert (AUTO, RULE-SAFE)

```sql
INSERT INTO marks (
    student_id,
    exam_id,
    class_subject_id,
    marks_obtained,
    status
)
SELECT
    s.id,
    2,                         -- 2nd Term
    cs.id,
    FLOOR(35 + RAND() * 66),   -- Marks 35–100
    'locked'
FROM students s
JOIN class_subjects cs
    ON cs.class_id = s.class_id;
```

---

# 🧪 VERIFICATION QUERIES

### Marks count per class

```sql
SELECT c.class_name, COUNT(m.id) total_marks
FROM marks m
JOIN students s ON s.id = m.student_id
JOIN classes c ON c.id = s.class_id
GROUP BY c.id;
```

Expected:

* Class 1 → 12
* Class 2 → 12
* Class 3 → 27
* Class 4 → 27
* Class 5 → 27

---

# ✅ SYSTEM GUARANTEES

✔ No duplicate students
✔ Correct subject per class
✔ GPA calculation safe
✔ Public result search reliable
✔ Production-ready DB foundation

---

## 🏁 FINAL NOTE

> **এই documentation থাকলে তুমি ৬ মাস পরেও system rebuild করতে পারবে ৩০ মিনিটে।**

---

## 🔜 When you are ready, next steps:

1️⃣ Merit Position (class rank)
2️⃣ Promotion system
3️⃣ Subject-wise full marks variation
4️⃣ Attendance integration

বললেই আমরা next chapter শুরু করবো 📚💪
