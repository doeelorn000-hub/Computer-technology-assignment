#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <iomanip>
#include <limits>

using namespace std;

// ==========================================
// WEEK 1: Student Class & Core Management
// ==========================================
class Student {
private:
    string id;
    string name;

public:
    Student(string id, string name) : id(id), name(name) {}

    string getId() const { return id; }
    string getName() const { return name; }

    void display() const {
        cout << "ID: " << left << setw(10) << id << " | Name: " << name << endl;
    }

    // Helper for file saving (Week 4)
    string serialize() const {
        return id + "," + name;
    }
};

// ==========================================
// WEEK 2: AttendanceSession & Program Flow
// ==========================================
class AttendanceSession {
private:
    string date;
    string subject;
    vector<string> presentStudentIds;

public:
    AttendanceSession(string d, string s) : date(d), subject(s) {}

    void markPresent(string studentId) {
        presentStudentIds.push_back(studentId);
    }

    string getDate() const { return date; }
    string getSubject() const { return subject; }
    const vector<string>& getAttendanceList() const { return presentStudentIds; }
};

// ==========================================
// WEEK 3 & 4: Logic, Reports, & File I/O
// ==========================================
class AttendanceManager {
private:
    vector<Student> students;
    vector<AttendanceSession> sessions;
    const string fileName = "students_db.csv";

public:
    // --- Week 1 Functions ---
    void addStudent() {
        string id, name;
        cout << "Enter Student ID: "; cin >> id;
        cin.ignore(); // Clear buffer
        cout << "Enter Student Name: "; getline(cin, name);
        students.push_back(Student(id, name));
        cout << "Student added successfully!\n";
    }

    void viewStudents() {
        if (students.empty()) {
            cout << "No students registered.\n";
            return;
        }
        cout << "\n--- Student Registry ---\n";
        for (const auto& s : students) s.display();
    }

    // --- Week 2 & 3 Functions ---
    void takeAttendance() {
        if (students.empty()) {
            cout << "No students available to mark attendance.\n";
            return;
        }

        string date, subject;
        cout << "Enter Date (YYYY-MM-DD): "; cin >> date;
        cout << "Enter Subject: "; cin >> subject;

        AttendanceSession newSession(date, subject);

        for (const auto& s : students) {
            char status;
            cout << "Is " << s.getName() << " (ID: " << s.getId() << ") present? (y/n): ";
            cin >> status;
            if (status == 'y' || status == 'Y') {
                newSession.markPresent(s.getId());
            }
        }
        sessions.push_back(newSession);
        cout << "Attendance session recorded.\n";
    }

    void generateReport() {
        if (sessions.empty()) {
            cout << "No attendance records found.\n";
            return;
        }

        cout << "\n--- Attendance Reports ---\n";
        for (const auto& session : sessions) {
            cout << "\nDate: " << session.getDate() << " | Subject: " << session.getSubject() << endl;
            cout << "Present IDs: ";
            for (const auto& id : session.getAttendanceList()) {
                cout << id << " ";
            }
            cout << "\nTotal Present: " << session.getAttendanceList().size() << endl;
        }
    }

    // --- Week 4 Functions: File Persistence ---
    void saveToFile() {
        ofstream outFile(fileName);
        if (outFile.is_open()) {
            for (const auto& s : students) {
                outFile << s.serialize() << endl;
            }
            outFile.close();
            cout << "Data saved to " << fileName << endl;
        }
    }

    void loadFromFile() {
        ifstream inFile(fileName);
        if (!inFile) return; // No file yet

        string line;
        students.clear();
        while (getline(inFile, line)) {
            size_t comma = line.find(',');
            if (comma != string::npos) {
                string id = line.substr(0, comma);
                string name = line.substr(comma + 1);
                students.push_back(Student(id, name));
            }
        }
        inFile.close();
    }
};

// --- Main Menu (Week 2/3 Refinement) ---
int main() {
    AttendanceManager manager;
    manager.loadFromFile(); // Week 4 feature

    int choice;
    do {
        cout << "\n=== STUDENT ATTENDANCE SYSTEM ===\n";
        cout << "1. Add Student (Week 1)\n";
        cout << "2. View All Students (Week 1)\n";
        cout << "3. Take Attendance (Week 2/3)\n";
        cout << "4. View Reports (Week 3)\n";
        cout << "5. Save and Exit (Week 4)\n";
        cout << "Enter choice: ";
        
        // Input Validation (Week 3)
        if (!(cin >> choice)) {
            cout << "Invalid input. Please enter a number.\n";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            continue;
        }

        switch (choice) {
            case 1: manager.addStudent(); break;
            case 2: manager.viewStudents(); break;
            case 3: manager.takeAttendance(); break;
            case 4: manager.generateReport(); break;
            case 5: manager.saveToFile(); cout << "Goodbye!\n"; break;
            default: cout << "Invalid option.\n";
        }
    } while (choice != 5);

    return 0;
}
