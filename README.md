# Coursify

A desktop **Course Registration System** built in C++ with a Qt 6 GUI. Coursify models a university course registration workflow with separate student and administrator roles, prerequisite validation, credit-hour tracking, grade management, and JSON-based persistence.

The project is an academic exercise in applying **Object-Oriented Programming** and **Data Structures** (hash maps, ordered maps, lists, graphs-via-prerequisites) to a realistic problem domain.

---

## Table of Contents

- [Features](#features)
- [Screenshots](#screenshots)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Data Files](#data-files)
- [CSV Import Formats](#csv-import-formats)
- [Class Reference](#class-reference)
- [Development](#development)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## Features

### Student
- Register and log in with validated credentials
- Search courses with live, keyboard-navigable results
- Add courses to a personal plan with automatic checks for:
  - Prerequisite completion
  - Remaining credit hours
  - Duplicate enrollment
- View grades and GPA per course
- Generate and export a full course report as a text file

### Administrator
- Register and log in using an admin secret key
- Import courses from a CSV file
- Import/update student grades from a CSV file
- Assign letter grades to a specific student/course
- Add and remove prerequisites on any course
- View and manage prerequisite lists per course

### General
- Role-based access control (`student` vs `admin`)
- Hashed password storage and format validation
- Automatic data load on startup and save on exit
- Default `admin` account bootstrapped on first run
- Custom Qt stylesheet/palette and application icon

---

## Screenshots

> Add screenshots of the login, registration, student panel, and admin panel here.

The repository already includes `asu.png` and `Icon.png` which can be used as branding assets.

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Language | C++17 |
| GUI Framework | Qt 6 (Widgets, Core, Gui) |
| Build System | MSBuild / Visual Studio 2022 (`.vcxproj`, `.sln`) |
| JSON | [nlohmann/json](https://github.com/nlohmann/json) (header-only, vendored in `Includes/`) |
| Persistence | Flat JSON files under `data/` |
| IDE | Qt Visual Studio Tools (`QtVS_v304`) |

---

## Architecture

The application follows a layered, MVC-inspired design:

```
main.cpp
  └── Coursify (Qt QMainWindow, GUI + signal/slot wiring)
        └── courseSystem (business logic / facade)
              ├── user (abstract base)
              │     ├── student
              │     └── admin
              ├── course
              ├── grade
              └── FileUtils (JSON serialization / persistence)
```

- **`Coursify`** (`Coursify.cpp:13`) owns the UI and connects widgets to `courseSystem` operations via Qt signals/slots.
- **`courseSystem`** (`courseSystem.cpp:20`) is the central facade holding the in-memory data stores:
  - `unordered_map<string, student> students`
  - `map<long, course> courses`
  - `unordered_map<string, admin> admins`
- **`FileUtils`** (`FileUtils.h:11`) handles all JSON read/write, keeping persistence out of the domain classes.
- **`Utilities`** provides shared helpers such as `setupRadioButtonLogic` and `validateCredentials`.

---

## Project Structure

```
Coursifyy-master/
├── main.cpp                 # Entry point: creates courseSystem, shows Coursify window
├── Coursify.cpp / .h        # Main window, signal/slot wiring, search, panels
├── Coursify.ui              # Qt Designer UI (login, register, student & admin panels)
├── Coursify.qrc             # Qt resource file
├── Coursify.sln / .vcxproj  # Visual Studio solution & project
│
├── courseSystem.cpp / .h    # Core business logic / facade
├── user.cpp / .h            # Base user class (auth, hashing, validation)
├── student.cpp / .h         # Student entity (courses, GPA, credit hours)
├── admin.cpp / .h           # Admin entity (grade & course management)
├── course.cpp / .h          # Course entity (prerequisites, credit hours)
├── grade.cpp / .h           # Grade entity (letter <-> GPA conversion)
├── FileUtils.cpp / .h       # JSON serialization & persistence
├── Utilities.cpp / .h       # Shared UI/validation helpers
├── stdafx.cpp / .h          # Precompiled header (<QtWidgets>)
│
├── Includes/                # Third-party headers (nlohmann/json)
├── data/                    # Runtime JSON data store
│   ├── students.json
│   ├── courses.json
│   └── admins.json
├── students.json            # Sample/seed data
├── courses.json
├── admins.json
│
├── existing_course_grades.csv   # Sample grades CSV
├── new_course_grades.csv        # Sample grades CSV
├── new_courses.csv              # Sample courses CSV
├── student_1000_report.txt      # Sample generated reports
├── student_1006_report.pdf
├── student_1007_report.txt
│
├── asu.png / Icon.png       # Branding assets
└── README.md
```

---

## Prerequisites

- **Windows 10/11**
- **Visual Studio 2022** with the *Desktop development with C++* workload (toolset `v143`)
- **Qt 6.8.3** (`msvc2022_64`) with modules: `core`, `gui`, `widgets`
- **Qt Visual Studio Tools** extension

> The project is configured for Windows/MSBuild. Building on Linux/macOS requires creating a CMake/qmake project or porting the `.vcxproj`.

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/Coursifyy.git
cd Coursifyy
```

### 2. Install Qt 6.8.3 for MSVC 2022 64-bit

Download from the [Qt online installer](https://www.qt.io/download-qt-installer) and select the `MSVC 2022 64-bit` kit.

### 3. Configure include paths

The project references third-party includes via an absolute path in `Coursify.vcxproj`:

```xml
<AdditionalIncludeDirectories>D:\Coursifyy\Includes;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
```

Update this to the absolute path of the `Includes/` folder on your machine (ideally using a `$(SolutionDir)` macro):

```xml
<AdditionalIncludeDirectories>$(SolutionDir)Includes;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
```

### 4. Open and build

1. Open `Coursify.sln` in Visual Studio 2022.
2. Ensure the Qt version is set (Qt VS Tools → Qt Versions → `6.8.3_msvc2022_64`).
3. Select configuration **Debug | x64** or **Release | x64**.
4. **Build → Build Solution** (`Ctrl+Shift+B`).

### 5. Run

```
x64\Debug\Coursify.exe
```

---

## Usage

### First Run

On first launch, if no admin exists, a default administrator is created (`courseSystem.cpp:24`):

| Username | Password |
|----------|----------|
| `admin`  | `admin123` |

> Change this immediately in a real deployment.

### Registering

1. From the login screen, click **Sign Up**.
2. Select **Student** or **Admin**.
3. Fill in name, username, password, and confirm password.
4. Admins must additionally provide the correct **admin secret key**.

Validation rules:
- **Name:** letters, spaces, hyphens, apostrophes, dots only
- **Username:** letters and numbers, must contain at least one letter
- **Password:** min 6 chars, at least one uppercase, one lowercase, one digit

### Student Workflow

1. Log in as a student.
2. Use the **search bar** to find courses. Results appear in a live dropdown.
   - `Enter` selects the highlighted result
   - `↓` moves focus into the results list
   - `Esc` closes the results list
3. Adding a course runs prerequisite, credit-hour, and duplicate checks.
4. Open the **Grades** tab to view per-course grades.
5. Open the **Report** tab to view title, ID, credit hours, grade, and GPA, then click **Export** to write a `.txt` report.

### Administrator Workflow

1. Log in as an admin.
2. **Upload course descriptions** via CSV (`Upload Description`).
3. **Upload grades** via CSV (`Upload Grades`) — admins only.
4. **Assign a grade** by entering the student username and course ID and choosing a letter.
5. **Manage prerequisites**: pick a course, choose a prerequisite course, click **Add**; select a row and click **Remove** to delete it.

---

## Data Files

Data is persisted as JSON in the `data/` directory, which is created at runtime relative to the executable's working directory (`FileUtils.cpp:14`).

| File | Contents |
|------|----------|
| `data/students.json` | All students, their courses, and grades |
| `data/courses.json` | All courses and their prerequisites |
| `data/admins.json` | All administrators |

Data is loaded in the `courseSystem` constructor and on `Coursify` construction, and saved on application exit (`Coursify.cpp:229`).

---

## CSV Import Formats

### Courses CSV (`new_courses.csv`)

Imported via **Upload Description**. Expected columns:

```csv
title,description,instructor,semester,credit_hours,syllabus
```

### Grades CSV (`new_course_grades.csv`)

Imported via **Upload Grades**. The header **must** match exactly (`courseSystem.cpp:958`):

```csv
student_id,course_id,grade_letter
1000,101,A
1006,102,B
```

Rules:
- `grade_letter` must be one of `A`, `B`, `C`, `D`, `F`
- The student must exist and already be enrolled in the course
- Invalid rows are collected and reported in a summary dialog

---

## Class Reference

| Class | File | Responsibility |
|-------|------|----------------|
| `user` | `user.h` | Base entity: credentials, hashing, validation |
| `student` | `student.h` | Course plan, GPA, credit hours, reports |
| `admin` | `admin.h` | Grade/course management, CSV uploads |
| `course` | `course.h` | Course metadata and prerequisites |
| `grade` | `grade.h` | Letter grade and GPA conversion |
| `courseSystem` | `courseSystem.h` | Facade over all data and operations |
| `FileUtils` | `FileUtils.h` | JSON serialization and persistence |
| `Coursify` | `Coursify.h` | Main window and UI event handling |

### Key Methods

**`courseSystem`**
- `authenticateUser(...)` — validate login, returns `'S'`, `'A'`, or `'F'`
- `registerStudent(...)` / `registerAdmin(...)` — create accounts with validation
- `searchCourses(term)` — case-insensitive course search
- `addPrerequisiteToList(...)` / `removeSelectedPrerequisite(...)` — prerequisite editing
- `assignGradeToStudent(...)` — set a letter grade
- `importCoursesFromFile(...)` / `importGradesFromFile(...)` — CSV import
- `getStudentGrades(...)` / `getStudentCourseReport(...)` — formatted report data
- `loadData()` / `saveData()` — persistence entry points

**`course`**
- `checkPrerequisites(student)` — returns true if all prereqs are completed
- `addPrerequisite(...)` / `removePrerequisite(...)` / `clearPrerequisites()`

**`user`**
- `hashPassword(...)` — one-way password hashing
- `isValidPassword(...)` / `isValidUsername(...)` — format validation

---

## Development

### Coding Conventions

- C++17, `using namespace std;` used throughout (academic project style)
- Precompiled header `stdafx.h` includes `<QtWidgets>`
- Domain classes are kept UI-agnostic except where they take Qt widget pointers for convenience in `courseSystem`
- No comments policy: keep code self-documenting

### Building from the Command Line

```bash
msbuild Coursify.sln /p:Configuration=Release /p:Platform=x64
```

### Known Considerations

- The `.vcxproj` contains a hard-coded `D:\Coursifyy\Includes` include path — update it for your machine.
- `Coursify.h` still declares an unused `courseSystem* m_system;` member (`Coursify.h:34`); the implementation uses the global `Sys` (`Coursify.cpp:10`).
- Some methods (e.g. `highlightMatchingChars`) are currently no-ops/stubs.
- `data/` and the root seed JSON files are both present; runtime data lives in `data/`.

---

## Roadmap

- [ ] Add a CMake build for cross-platform support
- [ ] Replace JSON flat files with a database backend
- [ ] Add unit tests for `courseSystem` and domain classes
- [ ] Implement real search-term highlighting
- [ ] Add semester/term filtering for course plans
- [ ] Export reports as PDF (currently TXT)
- [ ] Add an undo/audit trail for grade changes

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please keep changes focused and consistent with the existing code style.

---

## License

No license file is currently included in this repository. Until one is added, all rights are reserved by the author(s). Consider adding an [MIT](https://choosealicense.com/licenses/mit/) or [Apache-2.0](https://choosealicense.com/licenses/apache-2.0/) license if you intend to open-source it.

---

## Acknowledgments

- Built with [Qt 6](https://www.qt.io/)
- JSON handling via [nlohmann/json](https://github.com/nlohmann/json)
- Developed as an academic project demonstrating OOP and data structures in C++
