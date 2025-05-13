-- SQL Schema for a Student Records Database Management System
-- Use Case: Student Records
-- Database System: MySQL

-- Create the database if it doesn't already exist
CREATE DATABASE IF NOT EXISTS `StudentRecordsDB`;

-- Select the database to use for subsequent operations
USE `StudentRecordsDB`;

-- Comments are provided to explain the purpose of each table and its columns.
-- Constraints such as Primary Keys (PK), Foreign Keys (FK), NOT NULL, and UNIQUE are used.
-- Relationships are established.

-- Order of table creation -  due to Foreign Key dependencies.
-- 1. Departments
-- 2. Instructors
-- 3. Students
-- 4. Courses
-- 5. Enrollments

-- Table `Departments`
-- Stores information about academic departments.
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `Departments` (
  `department_id` INT NOT NULL AUTO_INCREMENT,
  `department_name` VARCHAR(255) NOT NULL,
  `office_location` VARCHAR(100) NULL,
  `contact_email` VARCHAR(255) NULL,
  `contact_phone` VARCHAR(20) NULL,
  PRIMARY KEY (`department_id`),
  UNIQUE INDEX `idx_department_name_unique` (`department_name` ASC) VISIBLE
) ENGINE = InnoDB COMMENT = 'Stores information about academic departments.';

-- -----------------------------------------------------
-- Table `Instructors`
-- Stores information about instructors/teachers.
-- Each instructor belongs to one department.
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `Instructors` (
  `instructor_id` INT NOT NULL AUTO_INCREMENT,
  `first_name` VARCHAR(100) NOT NULL,
  `last_name` VARCHAR(100) NOT NULL,
  `email` VARCHAR(255) NOT NULL,
  `phone_number` VARCHAR(20) NULL,
  `hire_date` DATE NOT NULL,
  `office_room_number` VARCHAR(50) NULL,
  `department_id` INT NOT NULL COMMENT 'Foreign key referencing the department the instructor belongs to.',
  PRIMARY KEY (`instructor_id`),
  UNIQUE INDEX `idx_email_unique` (`email` ASC) VISIBLE,
  UNIQUE INDEX `idx_phone_number_unique` (`phone_number` ASC) VISIBLE,
  INDEX `idx_fk_instructor_department` (`department_id` ASC) VISIBLE,
  CONSTRAINT `fk_instructor_department`
    FOREIGN KEY (`department_id`)
    REFERENCES `Departments` (`department_id`)
    ON DELETE RESTRICT -- Prevent deleting a department if it has instructors.
    ON UPDATE CASCADE -- If department_id changes, update it here.
) ENGINE = InnoDB COMMENT = 'Stores information about instructors.';

-- -----------------------------------------------------
-- Table `Students`
-- Stores information about students.
-- Each student can major in one department (optional).
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `Students` (
  `student_id` INT NOT NULL AUTO_INCREMENT,
  `first_name` VARCHAR(100) NOT NULL,
  `last_name` VARCHAR(100) NOT NULL,
  `date_of_birth` DATE NULL,
  `email` VARCHAR(255) NOT NULL,
  `phone_number` VARCHAR(20) NULL,
  `address_line1` VARCHAR(255) NULL,
  `address_city` VARCHAR(100) NULL,
  `address_state` VARCHAR(100) NULL,
  `address_zip_code` VARCHAR(20) NULL,
  `enrollment_date` DATE NOT NULL COMMENT 'Date when the student first enrolled.',
  `major_department_id` INT NULL COMMENT 'Foreign key referencing the department of the student''s major. Can be NULL if undeclared.',
  PRIMARY KEY (`student_id`),
  UNIQUE INDEX `idx_email_unique` (`email` ASC) VISIBLE,
  INDEX `idx_fk_student_major_department` (`major_department_id` ASC) VISIBLE,
  CONSTRAINT `fk_student_major_department`
    FOREIGN KEY (`major_department_id`)
    REFERENCES `Departments` (`department_id`)
    ON DELETE SET NULL -- If a department is deleted, student's major becomes NULL (undeclared).
    ON UPDATE CASCADE -- If department_id changes, updates it here.
) ENGINE = InnoDB COMMENT = 'Stores information about students.';

-- -----------------------------------------------------
-- Table `Courses`
-- Stores information about courses offered.
-- Each course is offered by one department and can be taught by one primary instructor.
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `Courses` (
  `course_id` INT NOT NULL AUTO_INCREMENT,
  `course_code` VARCHAR(20) NOT NULL COMMENT 'e.g., CS101, MATH203',
  `course_name` VARCHAR(255) NOT NULL,
  `course_description` TEXT NULL,
  `credits` INT NOT NULL COMMENT 'Number of credit hours for the course.',
  `department_id` INT NOT NULL COMMENT 'Foreign key referencing the department offering the course.',
  `instructor_id` INT NULL COMMENT 'Foreign key referencing the primary instructor for the course. Can be NULL if not yet assigned.',
  PRIMARY KEY (`course_id`),
  UNIQUE INDEX `idx_course_code_unique` (`course_code` ASC) VISIBLE,
  INDEX `idx_fk_course_department` (`department_id` ASC) VISIBLE,
  INDEX `idx_fk_course_instructor` (`instructor_id` ASC) VISIBLE,
  CONSTRAINT `chk_credits_positive` CHECK (`credits` > 0), -- Ensures credits are positive.
  CONSTRAINT `fk_course_department`
    FOREIGN KEY (`department_id`)
    REFERENCES `Departments` (`department_id`)
    ON DELETE RESTRICT -- Prevent deleting a department if it offers courses.
    ON UPDATE CASCADE,
  CONSTRAINT `fk_course_instructor`
    FOREIGN KEY (`instructor_id`)
    REFERENCES `Instructors` (`instructor_id`)
    ON DELETE SET NULL -- If instructor is deleted or leaves, the course's instructor field becomes NULL.
    ON UPDATE CASCADE
) ENGINE = InnoDB COMMENT = 'Stores information about courses.';

-- -----------------------------------------------------
-- Table `Enrollments`
-- Represents a Many-to-Many relationship between Students and Courses.
-- Stores records of students enrolled in specific courses for a particular semester.
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `Enrollments` (
  `enrollment_id` INT NOT NULL AUTO_INCREMENT,
  `student_id` INT NOT NULL COMMENT 'Foreign key referencing the student.',
  `course_id` INT NOT NULL COMMENT 'Foreign key referencing the course.',
  `enrollment_semester` VARCHAR(50) NOT NULL COMMENT 'Semester of enrollment, e.g., ''Fall 2024'', ''Spring 2025''.',
  `enrollment_date` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Date and time of enrollment.',
  `grade` VARCHAR(5) NULL COMMENT 'Grade obtained by the student in the course, e.g., A, B+, C-, P (Pass), F (Fail). NULL if not yet graded.',
  PRIMARY KEY (`enrollment_id`),
  INDEX `idx_fk_enrollment_student` (`student_id` ASC) VISIBLE,
  INDEX `idx_fk_enrollment_course` (`course_id` ASC) VISIBLE,
  UNIQUE INDEX `idx_unique_student_course_semester` (`student_id` ASC, `course_id` ASC, `enrollment_semester` ASC) VISIBLE COMMENT 'Ensures a student cannot enroll in the same course in the same semester more than once.',
  CONSTRAINT `fk_enrollment_student`
    FOREIGN KEY (`student_id`)
    REFERENCES `Students` (`student_id`)
    ON DELETE CASCADE -- If a student record is deleted, their enrollment records are also deleted.
    ON UPDATE CASCADE,
  CONSTRAINT `fk_enrollment_course`
    FOREIGN KEY (`course_id`)
    REFERENCES `Courses` (`course_id`)
    ON DELETE CASCADE -- If a course is deleted, enrollment records for that course are also deleted.
    ON UPDATE CASCADE
) ENGINE = InnoDB COMMENT = 'Stores student enrollment records for courses (Many-to-Many relationship linking Students and Courses).';
