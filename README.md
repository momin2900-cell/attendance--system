// Data storage
let students = [];
let attendanceRecords = [];

// Initialize with sample data
function initializeData() {
    const savedStudents = JSON.parse(localStorage.getItem('students') || '[]');
    const savedAttendance = JSON.parse(localStorage.getItem('attendance') || '[]');

    if (savedStudents.length === 0) {
        students = [
            { id: 1, rollNo: 'CSE2021001', name: 'Michael Johnson', email: 'michael@student.edu', dept: 'CSE' },
            { id: 2, rollNo: 'CSE2021002', name: 'Emma Wilson', email: 'emma@student.edu', dept: 'CSE' },
            { id: 3, rollNo: 'CSE2021003', name: 'Oliver Moore', email: 'oliver@student.edu', dept: 'CSE' },
            { id: 4, rollNo: 'ECE2021001', name: 'Sophia Taylor', email: 'sophia@student.edu', dept: 'ECE' },
            { id: 5, rollNo: 'MECH2021001', name: 'Liam Anderson', email: 'liam@student.edu', dept: 'MECH' }
        ];
        localStorage.setItem('students', JSON.stringify(students));
    } else {
        students = savedStudents;
    }

    attendanceRecords = savedAttendance;
    
    // Set default date to today
    const today = new Date().toISOString().split('T')[0];
    document.getElementById('attendanceDate').value = today;
    document.getElementById('reportFromDate').value = today;
    document.getElementById('reportToDate').value = today;

    updateDashboard();
    displayStudents();
}

// Tab navigation
function showTab(tabName) {
    const tabs = document.querySelectorAll('.tab-content');
    const navTabs = document.querySelectorAll('.nav-tab');
    
    tabs.forEach(tab => tab.classList.remove('active'));
    navTabs.forEach(tab => tab.classList.remove('active'));
    
    document.getElementById(tabName).classList.add('active');
    event.target.classList.add('active');

    if (tabName === 'dashboard') updateDashboard();
    if (tabName === 'students') displayStudents();
}

// Add student
document.getElementById('addStudentForm').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const newStudent = {
        id: Date.now(),
        rollNo: document.getElementById('rollNumber').value,
        name: document.getElementById('studentName').value,
        email: document.getElementById('studentEmail').value,
        dept: document.getElementById('studentDept').value
    };

    students.push(newStudent);
    localStorage.setItem('students', JSON.stringify(students));
    
    showAlert('studentAlert', 'Student added successfully!', 'success');
    this.reset();
    displayStudents();
    updateDashboard();
});

// Display students
function displayStudents() {
    const container = document.getElementById('studentsList');
    container.innerHTML = '';

    students.forEach(student => {
        const card = document.createElement('div');
        card.className = 'student-card';
        card.innerHTML = `
            <div class="student-info">
                <h3>${student.name}</h3>
                <p>Roll No: ${student.rollNo} | Dept: ${student.dept} | Email: ${student.email}</p>
            </div>
            <button class="btn btn-danger" onclick="deleteStudent(${student.id})">Delete</button>
        `;
        container.appendChild(card);
    });
}

// Delete student
function deleteStudent(id) {
    if (confirm('Are you sure you want to delete this student?')) {
        students = students.filter(s => s.id !== id);
        localStorage.setItem('students', JSON.stringify(students));
        displayStudents();
        updateDashboard();
    }
}

// Search students
function searchStudents() {
    const searchTerm = document.getElementById('searchStudent').value.toLowerCase();
    const filtered = students.filter(s => 
        s.name.toLowerCase().includes(searchTerm) || 
        s.rollNo.toLowerCase().includes(searchTerm)
    );

    const container = document.getElementById('studentsList');
    container.innerHTML = '';

    filtered.forEach(student => {
        const card = document.createElement('div');
        card.className = 'student-card';
        card.innerHTML = `
            <div class="student-info">
                <h3>${student.name}</h3>
                <p>Roll No: ${student.rollNo} | Dept: ${student.dept} | Email: ${student.email}</p>
            </div>
            <button class="btn btn-danger" onclick="deleteStudent(${student.id})">Delete</button>
        `;
        container.appendChild(card);
    });
}

// Load attendance for date
function loadAttendanceForDate() {
    const date = document.getElementById('attendanceDate').value;
    if (!date) {
        showAlert('attendanceAlert', 'Please select a date!', 'danger');
        return;
    }

    const container = document.getElementById('attendanceList');
    container.innerHTML = '';

    students.forEach(student => {
        const existingRecord = attendanceRecords.find(
            r => r.studentId === student.id && r.date === date
        );

        const card = document.createElement('div');
        card.className = 'student-card';
        card.innerHTML = `
            <div class="student-info">
                <h3>${student.name}</h3>
                <p>Roll No: ${student.rollNo} | Dept: ${student.dept}</p>
            </div>
            <div class="attendance-buttons">
                <button class="btn btn-success" onclick="markAttendance(${student.id}, '${date}', 'present')">
                    ${existingRecord?.status === 'present' ? '✓ Present' : 'Present'}
                </button>
                <button class="btn btn-danger" onclick="markAttendance(${student.id}, '${date}', 'absent')">
                    ${existingRecord?.status === 'absent' ? '✓ Absent' : 'Absent'}
                </button>
            </div>
        `;
        container.appendChild(card);
    });
}

// Mark attendance
function markAttendance(studentId, date, status) {
    const existingIndex = attendanceRecords.findIndex(
        r => r.studentId === studentId && r.date === date
    );

    if (existingIndex !== -1) {
        attendanceRecords[existingIndex].status = status;
    } else {
        attendanceRecords.push({
            studentId,
            date,
            status,
            timestamp: new Date().toISOString()
        });
    }

    localStorage.setItem('attendance', JSON.stringify(attendanceRecords));
    loadAttendanceForDate();
    showAlert('attendanceAlert', 'Attendance marked successfully!', 'success');
    updateDashboard();
}

// Mark all present
function markAllPresent() {
    const date = document.getElementById('attendanceDate').value;
    if (!date) {
        showAlert('attendanceAlert', 'Please select a date!', 'danger');
        return;
    }

    students.forEach(student => {
        markAttendance(student.id, date, 'present');
    });
}

// Update dashboard
function updateDashboard() {
    const today = new Date().toISOString().split('T')[0];
    const todayRecords = attendanceRecords.filter(r => r.date === today);
    
    document.getElementById('totalStudents').textContent = students.length;
    document.getElementById('presentToday').textContent = todayRecords.filter(r => r.status === 'present').length;
    document.getElementById('absentToday').textContent = todayRecords.filter(r => r.status === 'absent').length;

    // Calculate average attendance
    const totalRecords = attendanceRecords.length;
    const presentCount = attendanceRecords.filter(r => r.status === 'present').length;
    const avgAttendance = totalRecords > 0 ? Math.round((presentCount / totalRecords) * 100) : 0;
    document.getElementById('avgAttendance').textContent = avgAttendance + '%';

    // Display recent records
    const recentRecords = attendanceRecords.slice(-10).reverse();
    const tbody = document.getElementById('recentAttendanceBody');
    tbody.innerHTML = '';

    recentRecords.forEach(record => {
        const student = students.find(s => s.id === record.studentId);
        if (student) {
            const row = tbody.insertRow();
            row.innerHTML = `
                <td>${student.rollNo}</td>
                <td>${student.name}</td>
                <td>${record.date}</td>
                <td><span class="badge badge-${record.status === 'present' ? 'success' : 'danger'}">${record.status.toUpperCase()}</span></td>
            `;
        }
    });
}

// Generate report
function generateReport() {
    const fromDate = document.getElementById('reportFromDate').value;
    const toDate = document.getElementById('reportToDate').value;

    if (!fromDate || !toDate) {
        alert('Please select both dates!');
        return;
    }

    const tbody = document.getElementById('reportTableBody');
    tbody.innerHTML = '';

    students.forEach(student => {
        const studentRecords = attendanceRecords.filter(r => 
            r.studentId === student.id && 
            r.date >= fromDate && 
            r.date <= toDate
        );

        const totalClasses = studentRecords.length;
        const present = studentRecords.filter(r => r.status === 'present').length;
        const absent = studentRecords.filter(r => r.status === 'absent').length;
        const percentage = totalClasses > 0 ? Math.round((present / totalClasses) * 100) : 0;

        const row = tbody.insertRow();
        row.innerHTML = `
            <td>${student.rollNo}</td>
            <td>${student.name}</td>
            <td>${totalClasses}</td>
            <td>${present}</td>
            <td>${absent}</td>
            <td>${percentage}%</td>
            <td><span class="badge badge-${percentage >= 75 ? 'success' : percentage >= 60 ? 'warning' : 'danger'}">
                ${percentage >= 75 ? 'Good' : percentage >= 60 ? 'Average' : 'Poor'}
            </span></td>
        `;
    });
}

// Show alert
function showAlert(elementId, message, type) {
    const alertDiv = document.getElementById(elementId);
    alertDiv.innerHTML = `<div class="alert alert-${type}">${message}</div>`;
    setTimeout(() => {
        alertDiv.innerHTML = '';
    }, 3000);
}

// Initialize on load
window.onload = initializeData;
