# Academic-Performance-Evaluation-System
 This object-oriented program processes student marks across multiple subjects, handles structural weighting (e.g., matching a syllabus design where assignments are worth 40% and exams are worth 60%), assigns letter grades, flags academic risk factors, and displays detailed summaries.

from datetime import datetime
import json
from typing import Dict, List, Any, Tuple

# =====================================================================
# 1. CORE DATA STRUCTURE: STUDENT PERFORMANCE PROFILE
# =====================================================================

class StudentAcademicRecord:
    """Manages raw grade tracking, weighting metrics, and grade conversions."""
    def __init__(self, student_id: str, name: str):
        self.student_id = student_id
        self.name = name
        # Format: {"Subject_Name": {"assignments": [...], "exams": [...]}}
        self.coursework_data: Dict[str, Dict[str, List[float]]] = {}

    def add_marks(self, subject: str, component_type: str, mark: float):
        """Safely logs raw marks under explicit categorical components."""
        if component_type not in ["assignments", "exams"]:
            raise ValueError("Component type must be strictly 'assignments' or 'exams'.")
        
        if subject not in self.coursework_data:
            self.coursework_data[subject] = {"assignments": [], "exams": []}
            
        self.coursework_data[subject][component_type].append(mark)

    @staticmethod
    def calculate_letter_grade(final_score: float) -> str:
        """Applies academic rubric standards to classify performance metrics."""
        if final_score >= 90.0: return "A"
        elif final_score >= 80.0: return "B"
        elif final_score >= 70.0: return "C"
        elif final_score >= 60.0: return "D"
        else: return "F"

    def evaluate_subject_performance(self, subject: str, assignment_weight: float = 0.40) -> Dict[str, Any]:
        """Computes weighted totals. Default split: 40% Assignments, 60% Exams."""
        scores = self.coursework_data.get(subject)
        if not scores:
            return {"status": "No Data Registered"}

        # Calculate isolated component means
        avg_assign = sum(scores["assignments"]) / len(scores["assignments"]) if scores["assignments"] else 0.0
        avg_exam = sum(scores["exams"]) / len(scores["exams"]) if scores["exams"] else 0.0

        # Calculate final weighted score
        exam_weight = 1.0 - assignment_weight
        final_weighted_mark = (avg_assign * assignment_weight) + (avg_exam * exam_weight)
        
        return {
            "assignment_average": round(avg_assign, 2),
            "exam_average": round(avg_exam, 2),
            "final_score": round(final_weighted_mark, 2),
            "letter_grade": self.calculate_letter_grade(final_weighted_mark)
        }


# =====================================================================
# 2. ORCHESTRATION LAYER: FACULTY MARKS CONSOLE
# =====================================================================

class FacultyMarksConsole:
    """Provides processing, analytical screening, and batch generation tools."""
    def __init__(self, course_batch_name: str):
        self.course_batch_name = course_batch_name
        self.student_directory: Dict[str, StudentAcademicRecord] = {}

    def register_student(self, student: StudentAcademicRecord):
        self.student_directory[student.student_id] = student

    def generate_academic_report(self) -> Dict[str, Any]:
        """Aggregates all student marks into a structured dashboard report."""
        report: Dict[str, Any] = {
            "report_generation_date": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "academic_batch": self.course_batch_name,
            "student_summaries": [],
            "risk_assessment_alerts": []
        }

        for s_id, record in self.student_directory.items():
            student_profile = {
                "id": s_id,
                "name": record.name,
                "subjects": {}
            }
            
            total_gpa_score = 0.0
            subjects_counted = 0

            for subject in record.coursework_data.keys():
                metrics = record.evaluate_subject_performance(subject)
                student_profile["subjects"][subject] = metrics
                
                total_gpa_score += metrics["final_score"]
                subjects_counted += 1

            # Cumulative average across all subjects
            cumulative_avg = round(total_gpa_score / subjects_counted, 2) if subjects_counted > 0 else 0.0
            student_profile["cumulative_average"] = cumulative_avg
            student_profile["overall_standing"] = StudentAcademicRecord.calculate_letter_grade(cumulative_avg)
            
            report["student_summaries"].append(student_profile)

            # Proactive Risk Screening: Flag students tracking below minimum criteria (F grade boundary)
            if cumulative_avg < 60.0:
                report["risk_assessment_alerts"].append({
                    "student_id": s_id,
                    "name": record.name,
                    "current_average": cumulative_avg,
                    "intervention_required": "High Risk - Schedule academic counseling review."
                })

        return report


# =====================================================================
# 3. RUNTIME EXECUTION & DATA SIMULATION
# =====================================================================

if __name__ == "__main__":
    # Initialize Faculty Console for Semester 1 Computer Science
    faculty_hub = FacultyMarksConsole(course_batch_name="CS_2026_Semester_1")

    # --- Student 1: High Academic Achievement Profile ---
    student_1 = StudentAcademicRecord(student_id="STU_001", name="Aarav Sharma")
    # Log Data for Data Structures
    student_1.add_marks("Data Structures", "assignments", 92.0)
    student_1.add_marks("Data Structures", "assignments", 88.5)
    student_1.add_marks("Data Structures", "exams", 94.0)
    # Log Data for Discrete Mathematics
    student_1.add_marks("Discrete Math", "assignments", 85.0)
    student_1.add_marks("Discrete Math", "exams", 89.0)

    # --- Student 2: Underperforming Profile (Triggers Alert) ---
    student_2 = StudentAcademicRecord(student_id="STU_002", name="Priya Patel")
    # Log Data for Data Structures
    student_2.add_marks("Data Structures", "assignments", 62.0)
    student_2.add_marks("Data Structures", "exams", 54.5)
    # Log Data for Discrete Mathematics
    student_2.add_marks("Discrete Math", "assignments", 50.0)
    student_2.add_marks("Discrete Math", "exams", 58.0)

    # Register both profiles into the Faculty ecosystem
    faculty_hub.register_student(student_1)
    faculty_hub.register_student(student_2)

    # Run the computational processing pipeline
    batch_report_data = faculty_hub.generate_academic_report()
    
    print("==============================================================")
    print("🎓 COMPUTED ACADEMIC BATCH REPORT OUTPUT 🎓")
    print("==============================================================\n")
    print(json.dumps(batch_report_data, indent=4))
