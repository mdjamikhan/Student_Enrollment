package AgainMain;

import java.util.*;
import java.util.stream.Collectors;

// ===== Interfaces =====
interface IStudent {
    int getStudentId();
    String getName();
    int getSemester();
}

interface ICourse {
    String getCourseCode();
    String getTitle();
    int getMaxCapacity();
    int getCredits();
}

// ===== Enrollment System =====
class EnrollmentSystem<TStudent extends IStudent, TCourse extends ICourse> {

    private final Map<TCourse, Set<TStudent>> enrollments = new HashMap<>();

    public boolean enrollStudent(TStudent student, TCourse course) {
        Objects.requireNonNull(student, "student must not be null");
        Objects.requireNonNull(course, "course must not be null");

        Set<TStudent> students = enrollments.computeIfAbsent(course, k -> new HashSet<>());

        if (students.size() >= course.getMaxCapacity()) {
            System.out.println("❌ Enrollment failed: Course is full");
            return false;
        }

        if (students.contains(student)) {
            System.out.println("❌ Enrollment failed: Student already enrolled");
            return false;
        }

        if (course instanceof LabCourse lab) {
            if (student.getSemester() < lab.getRequiredSemester()) {
                System.out.println("❌ Enrollment failed: Prerequisite semester not satisfied");
                return false;
            }
        }

        students.add(student);
        System.out.println("✅ Enrolled " + student.getName() + " in " + course.getTitle());
        return true;
    }

    public Set<TStudent> getEnrolledStudents(TCourse course) {
        return Collections.unmodifiableSet(enrollments.getOrDefault(course, Set.of()));
    }

    public Set<TCourse> getStudentCourses(TStudent student) {
        return enrollments.entrySet().stream()
                .filter(e -> e.getValue().contains(student))
                .map(Map.Entry::getKey)
                .collect(Collectors.toSet());
    }

    public int calculateStudentWorkload(TStudent student) {
        return getStudentCourses(student).stream()
                .mapToInt(ICourse::getCredits)
                .sum();
    }

    public boolean isEnrolled(TStudent student, TCourse course) {
        return enrollments.getOrDefault(course, Set.of()).contains(student);
    }
}

// ===== Models =====
class EngineeringStudent implements IStudent {
    private final int studentId;
    private final String name;
    private final int semester;
    private final String specialization;

    public EngineeringStudent(int studentId, String name, int semester, String specialization) {
        this.studentId = studentId;
        this.name = Objects.requireNonNull(name);
        this.semester = semester;
        this.specialization = Objects.requireNonNull(specialization);
    }

    public int getStudentId() { return studentId; }
    public String getName() { return name; }
    public int getSemester() { return semester; }
    public String getSpecialization() { return specialization; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof EngineeringStudent that)) return false;
        return studentId == that.studentId;
    }

    @Override
    public int hashCode() {
        return Objects.hash(studentId);
    }

    @Override
    public String toString() {
        return name + " (Sem " + semester + ", " + specialization + ")";
    }
}

class LabCourse implements ICourse {
    private final String courseCode;
    private final String title;
    private final int maxCapacity;
    private final int credits;
    private final String labEquipment;
    private final int requiredSemester;

    public LabCourse(String courseCode, String title, int maxCapacity, int credits, String labEquipment, int requiredSemester) {
        this.courseCode = Objects.requireNonNull(courseCode);
        this.title = Objects.requireNonNull(title);
        this.maxCapacity = maxCapacity;
        this.credits = credits;
        this.labEquipment = Objects.requireNonNull(labEquipment);
        this.requiredSemester = requiredSemester;
    }

    public String getCourseCode() { return courseCode; }
    public String getTitle() { return title; }
    public int getMaxCapacity() { return maxCapacity; }
    public int getCredits() { return credits; }
    public String getLabEquipment() { return labEquipment; }
    public int getRequiredSemester() { return requiredSemester; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof LabCourse that)) return false;
        return courseCode.equals(that.courseCode);
    }

    @Override
    public int hashCode() {
        return Objects.hash(courseCode);
    }

    @Override
    public String toString() {
        return title + " (" + courseCode + ")";
    }
}

// ===== GradeBook =====
class GradeBook<TStudent extends IStudent, TCourse extends ICourse> {

    private final Map<TStudent, Map<TCourse, Double>> grades = new HashMap<>();
    private final EnrollmentSystem<TStudent, TCourse> enrollmentSystem;

    public GradeBook(EnrollmentSystem<TStudent, TCourse> enrollmentSystem) {
        this.enrollmentSystem = enrollmentSystem;
    }

    public void addGrade(TStudent student, TCourse course, double grade) {
        if (grade < 0 || grade > 100)
            throw new IllegalArgumentException("Grade must be between 0 and 100");

        if (!enrollmentSystem.isEnrolled(student, course))
            throw new IllegalArgumentException("Student is not enrolled in this course");

        grades.computeIfAbsent(student, s -> new HashMap<>()).put(course, grade);
    }

    public Optional<Double> calculateGPA(TStudent student) {
        Map<TCourse, Double> studentGrades = grades.get(student);
        if (studentGrades == null || studentGrades.isEmpty()) return Optional.empty();

        double totalWeighted = 0;
        int totalCredits = 0;

        for (Map.Entry<TCourse, Double> e : studentGrades.entrySet()) {
            totalWeighted += e.getValue() * e.getKey().getCredits();
            totalCredits += e.getKey().getCredits();
        }

        return totalCredits == 0 ? Optional.empty() : Optional.of(totalWeighted / totalCredits);
    }

    public Optional<Map.Entry<TStudent, Double>> getTopStudent(TCourse course) {
        return grades.entrySet().stream()
                .filter(e -> e.getValue().containsKey(course))
                .map(e -> Map.entry(e.getKey(), e.getValue().get(course)))
                .max(Map.Entry.comparingByValue());
    }
}

// ===== Main =====
public class Main {
    public static void main(String[] args) {

        EnrollmentSystem<EngineeringStudent, LabCourse> system = new EnrollmentSystem<>();

        EngineeringStudent s1 = new EngineeringStudent(1, "Amit", 2, "CS");
        EngineeringStudent s2 = new EngineeringStudent(2, "Riya", 4, "IT");
        EngineeringStudent s3 = new EngineeringStudent(3, "Karan", 1, "CS");

        LabCourse c1 = new LabCourse("CS101", "Data Structures Lab", 2, 4, "PC Lab", 2);
        LabCourse c2 = new LabCourse("CS201", "OS Lab", 1, 3, "Linux Lab", 4);

        // Enrollments
        system.enrollStudent(s1, c1);
        system.enrollStudent(s2, c1);
        system.enrollStudent(s3, c1); // fails (semester)

        system.enrollStudent(s2, c2);
        system.enrollStudent(s1, c2); // fails

        // Gradebook
        GradeBook<EngineeringStudent, LabCourse> gradeBook = new GradeBook<>(system);
        gradeBook.addGrade(s1, c1, 85);
        gradeBook.addGrade(s2, c1, 92);
        gradeBook.addGrade(s2, c2, 88);

        // GPA
        System.out.println("GPA of Amit: " + gradeBook.calculateGPA(s1).orElse(null));
        System.out.println("GPA of Riya: " + gradeBook.calculateGPA(s2).orElse(null));

        // Top student
        gradeBook.getTopStudent(c1).ifPresent(top ->
                System.out.println("Top student in " + c1.getTitle() + ": " +
                        top.getKey().getName() + " with " + top.getValue())
        );
    }
}
