# Database-Project
CREATE TABLE Department (
    Department_ID INT PRIMARY KEY,
    Department_Name VARCHAR(100)
);

INSERT INTO Department( Department_ID, Department_Name )
VALUES
(1, 'Cardiology'),
(2, 'Neurology');

CREATE TABLE Doctor (
    Doctor_ID INT PRIMARY KEY,
    Doctor_Name VARCHAR(100),
    Department_ID INT FOREIGN KEY REFERENCES Department(Department_ID)
);

INSERT INTO Doctor(  Doctor_ID, Doctor_Name, Department_ID )
VALUES
(1, 'Dr. Sarah Khan', 1),
(2, 'Dr. Ahmed Raza', 2);

CREATE TABLE Patient (
    Patient_ID INT PRIMARY KEY,
    Patient_Name VARCHAR(100)
);

INSERT INTO Patient( Patient_ID, Patient_Name )
VALUES
(1, 'Ali Rehman'),
(2, 'Sana Tariq');

CREATE TABLE Room (
    Room_ID INT PRIMARY KEY,
    Room_Type VARCHAR(50),
    ChargesPerDay DECIMAL(10, 2)
);

INSERT INTO Room( Room_ID, Room_Type, ChargesPerDay ) 
VALUES
(101, 'General', 2000.00),
(102, 'Private', 5000.00);

CREATE TABLE Patient_Room (
    Patient_ID INT FOREIGN KEY REFERENCES Patient(Patient_ID),
    Room_ID INT FOREIGN KEY REFERENCES Room(Room_ID),
    PRIMARY KEY (Patient_ID, Room_ID)
);

INSERT INTO Patient_Room ( Patient_ID, Room_ID )
VALUES
(1, 101),
(2, 102);

CREATE TABLE Medication (
    Medication_ID INT PRIMARY KEY,
    Patient_ID INT FOREIGN KEY REFERENCES Patient(Patient_ID),
    Medication_Name VARCHAR(100),
    Dosage VARCHAR(50)
);

INSERT INTO Medication( Medication_ID, Patient_ID, Medication_Name, Dosage )
VALUES
(1, 1, 'Paracetamol', '500mg'),
(2, 2, 'Ibuprofen', '200mg');

CREATE TABLE Appointment (
    Appointment_ID INT PRIMARY KEY,
    Patient_ID INT FOREIGN KEY REFERENCES Patient(Patient_ID),
    Doctor_ID INT FOREIGN KEY REFERENCES Doctor(Doctor_ID),
    Appointment_Date DATE,
    Appointment_Time TIME,
    Status VARCHAR(50)
);

INSERT INTO Appointment( Appointment_ID, Patient_ID, Doctor_ID, Appointment_Date, Appointment_Time, Status )
VALUES
(1, 1, 1, '2025-06-10', '10:30', 'Completed'),
(2, 2, 2, '2025-06-12', '11:00', 'Scheduled');

CREATE TABLE Prescription (
    Prescription_ID INT PRIMARY KEY,
    Appointment_ID INT FOREIGN KEY REFERENCES Appointment(Appointment_ID),
    Medication_Name VARCHAR(100),
    Dosage VARCHAR(50),
    Duration VARCHAR(50)
);

INSERT INTO Prescription( Prescription_ID, Appointment_ID, Medication_Name, Dosage, Duration )
VALUES
(1, 1, 'Paracetamol', '500mg', '5 days'),
(2, 2, 'Ibuprofen', '200mg', '3 days');

CREATE TABLE Admission (
    Admission_ID INT PRIMARY KEY,
    Patient_ID INT FOREIGN KEY REFERENCES Patient(Patient_ID),
    Room_ID INT FOREIGN KEY REFERENCES Room(Room_ID),
    Admission_Date DATE,
    Discharge_Date DATE
);

INSERT INTO Admission( Admission_ID, Patient_ID, Room_ID, Admission_Date, Discharge_Date ) 
VALUES
(1, 1, 101, '2025-06-01', '2025-06-05'),
(2, 2, 102, '2025-06-03', '2025-06-07');

CREATE TABLE Bill (
    Bill_ID INT PRIMARY KEY,
    Admission_ID INT FOREIGN KEY REFERENCES Admission(Admission_ID),
    Total_Amount DECIMAL(10, 2),
    Payment_Status VARCHAR(50)
);

INSERT INTO Bill( Bill_ID, Admission_ID, Total_Amount, Payment_Status ) 
VALUES
(1, 1, 10000.00, 'Paid'),
(2, 2, 20000.00, 'Unpaid');

--Retrieve all patients
SELECT * FROM Patient;

--Appointments that are completed
SELECT Appointment_ID, Patient_ID, Appointment_Date
FROM Appointment
WHERE Status = 'Completed';

--Get patient names with their room type
SELECT p.Patient_Name, r.Room_Type
FROM Patient p
JOIN Patient_Room pr ON p.Patient_ID = pr.Patient_ID
JOIN Room r ON pr.Room_ID = r.Room_ID;

--List all prescriptions with patient and doctor names
SELECT p.Patient_Name, d.Doctor_Name, pr.Medication_Name, pr.Dosage, pr.Duration
FROM Prescription pr
JOIN Appointment a ON pr.Appointment_ID = a.Appointment_ID
JOIN Patient p ON a.Patient_ID = p.Patient_ID
JOIN Doctor d ON a.Doctor_ID = d.Doctor_ID;

--Count of appointments for each doctor
SELECT d.Doctor_Name, COUNT(*) AS Total_Appointments
FROM Appointment a
JOIN Doctor d ON a.Doctor_ID = d.Doctor_ID
GROUP BY d.Doctor_Name;

--Doctors with more than 1 appointment
SELECT d.Doctor_Name, COUNT(*) AS Appointment_Count
FROM Appointment a
JOIN Doctor d ON a.Doctor_ID = d.Doctor_ID
GROUP BY d.Doctor_Name
HAVING COUNT(*) > 1;

--Patients who have unpaid bills
SELECT Patient_Name
FROM Patient
WHERE Patient_ID IN (
    SELECT a.Patient_ID
    FROM Admission a
    JOIN Bill b ON a.Admission_ID = b.Admission_ID
    WHERE b.Payment_Status = 'Unpaid'
);

--Total amount billed per patient
SELECT p.Patient_Name, SUM(b.Total_Amount) AS Total_Billed
FROM Patient p
JOIN Admission a ON p.Patient_ID = a.Patient_ID
JOIN Bill b ON a.Admission_ID = b.Admission_ID
GROUP BY p.Patient_Name;

--Appointments scheduled for a specific date
SELECT p.Patient_Name, d.Doctor_Name, a.Appointment_Date, a.Appointment_Time
FROM Appointment a
JOIN Patient p ON a.Patient_ID = p.Patient_ID
JOIN Doctor d ON a.Doctor_ID = d.Doctor_ID
WHERE a.Appointment_Date = '2025-06-10';

--Rooms never assigned to any patient
SELECT Room_ID, Room_Type
FROM Room
WHERE Room_ID NOT IN (
    SELECT DISTINCT Room_ID FROM Patient_Room
);


