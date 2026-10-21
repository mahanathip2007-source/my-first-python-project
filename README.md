# my-first-python-project
my first python project 
import sqlite3
from datetime import datetime

# Connect to database
conn = sqlite3.connect("hospital.db")
cursor = conn.cursor()

# Create patient table
cursor.execute("""
CREATE TABLE IF NOT EXISTS patients (
    patient_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER NOT NULL,
    gender TEXT NOT NULL,
    phone TEXT,
    disease TEXT
)
""")

# Create appointment table
cursor.execute("""
CREATE TABLE IF NOT EXISTS appointments (
    appointment_id INTEGER PRIMARY KEY AUTOINCREMENT,
    patient_id INTEGER,
    doctor TEXT NOT NULL,
    appointment_date TEXT NOT NULL,
    FOREIGN KEY(patient_id) REFERENCES patients(patient_id)
)
""")

conn.commit()


def register_patient():
    print("\n--- Patient Registration ---")

    name = input("Enter patient name: ").strip()

    if not name:
        print("Name cannot be empty!")
        return

    try:
        age = int(input("Enter age: "))
        if age < 0 or age > 130:
            print("Please enter a valid age.")
            return
    except ValueError:
        print("Age must be a number.")
        return

    gender = input("Enter gender: ").strip()
    phone = input("Enter phone number: ").strip()
    disease = input("Enter reported health problem: ").strip()

    cursor.execute("""
        INSERT INTO patients (name, age, gender, phone, disease)
        VALUES (?, ?, ?, ?, ?)
    """, (name, age, gender, phone, disease))

    conn.commit()

    print("Patient registered successfully!")
    print("Patient ID:", cursor.lastrowid)


def view_patients():
    print("\n--- Patient List ---")

    cursor.execute("SELECT * FROM patients")
    patients = cursor.fetchall()

    if not patients:
        print("No patient records found.")
        return

    for patient in patients:
        print("-" * 40)
        print("Patient ID:", patient[0])
        print("Name:", patient[1])
        print("Age:", patient[2])
        print("Gender:", patient[3])
        print("Phone:", patient[4])
        print("Reported Problem:", patient[5])


def search_patient():
    print("\n--- Search Patient ---")

    try:
        patient_id = int(input("Enter patient ID: "))
    except ValueError:
        print("Invalid patient ID.")
        return

    cursor.execute(
        "SELECT * FROM patients WHERE patient_id = ?",
        (patient_id,)
    )

    patient = cursor.fetchone()

    if patient:
        print("\nPatient Found")
        print("ID:", patient[0])
        print("Name:", patient[1])
        print("Age:", patient[2])
        print("Gender:", patient[3])
        print("Phone:", patient[4])
        print("Reported Problem:", patient[5])
    else:
        print("Patient not found.")


def update_patient():
    print("\n--- Update Patient ---")

    try:
        patient_id = int(input("Enter patient ID: "))
    except ValueError:
        print("Invalid patient ID.")
        return

    cursor.execute(
        "SELECT * FROM patients WHERE patient_id = ?",
        (patient_id,)
    )

    patient = cursor.fetchone()

    if not patient:
        print("Patient not found.")
        return

    print("Press Enter to keep an existing value.")

    name = input(f"Name [{patient[1]}]: ").strip() or patient[1]

    try:
        age_input = input(f"Age [{patient[2]}]: ").strip()
        age = int(age_input) if age_input else patient[2]

        if age < 0 or age > 130:
            print("Invalid age.")
            return
    except ValueError:
        print("Age must be a number.")
        return

    gender = input(
        f"Gender [{patient[3]}]: "
    ).strip() or patient[3]

    phone = input(
        f"Phone [{patient[4] or ''}]: "
    ).strip() or patient[4]

    disease = input(
        f"Reported Problem [{patient[5] or ''}]: "
    ).strip() or patient[5]

    cursor.execute("""
        UPDATE patients
        SET name = ?, age = ?, gender = ?, phone = ?, disease = ?
        WHERE patient_id = ?
    """, (name, age, gender, phone, disease, patient_id))

    conn.commit()
    print("Patient details updated successfully!")


def delete_patient():
    print("\n--- Delete Patient ---")

    try:
        patient_id = int(input("Enter patient ID: "))
    except ValueError:
        print("Invalid patient ID.")
        return

    cursor.execute(
        "SELECT name FROM patients WHERE patient_id = ?",
        (patient_id,)
    )

    patient = cursor.fetchone()

    if not patient:
        print("Patient not found.")
        return

    print("Patient:", patient[0])
    confirm = input("Delete this record? (yes/no): ").strip().lower()

    if confirm == "yes":
        cursor.execute(
            "DELETE FROM appointments WHERE patient_id = ?",
            (patient_id,)
        )
        cursor.execute(
            "DELETE FROM patients WHERE patient_id = ?",
            (patient_id,)
        )
        conn.commit()
        print("Patient record deleted.")
    else:
        print("Deletion cancelled.")


def book_appointment():
    print("\n--- Book Appointment ---")

    try:
        patient_id = int(input("Enter patient ID: "))
    except ValueError:
        print("Invalid patient ID.")
        return

    cursor.execute(
        "SELECT name FROM patients WHERE patient_id = ?",
        (patient_id,)
    )

    patient = cursor.fetchone()

    if not patient:
        print("Patient not found. Register the patient first.")
        return

    doctor = input("Enter doctor name: ").strip()

    if not doctor:
        print("Doctor name cannot be empty.")
        return

    date_text = input(
        "Enter appointment date (YYYY-MM-DD): "
    ).strip()

    try:
        appointment_date = datetime.strptime(
            date_text, "%Y-%m-%d"
        ).date()

        if appointment_date < datetime.today().date():
            print("Appointment date cannot be in the past.")
            return
    except ValueError:
        print("Invalid date format.")
        return

    cursor.execute("""
        INSERT INTO appointments
        (patient_id, doctor, appointment_date)
        VALUES (?, ?, ?)
    """, (patient_id, doctor, date_text))

    conn.commit()

    print("Appointment booked successfully!")
    print("Patient:", patient[0])


def view_appointments():
    print("\n--- Appointment List ---")

    cursor.execute("""
        SELECT a.appointment_id, p.name, a.doctor,
               a.appointment_date
        FROM appointments a
        JOIN patients p ON a.patient_id = p.patient_id
        ORDER BY a.appointment_date
    """)

    appointments = cursor.fetchall()

    if not appointments:
        print("No appointments found.")
        return

    for appointment in appointments:
        print("-" * 40)
        print("Appointment ID:", appointment[0])
        print("Patient:", appointment[1])
        print("Doctor:", appointment[2])
        print("Date:", appointment[3])


def generate_bill():
    print("\n--- Patient Bill ---")

    try:
        patient_id = int(input("Enter patient ID: "))
    except ValueError:
        print("Invalid patient ID.")
        return

    cursor.execute(
        "SELECT name FROM patients WHERE patient_id = ?",
        (patient_id,)
    )

    patient = cursor.fetchone()

    if not patient:
        print("Patient not found.")
        return

    try:
        consultation_fee = float(
            input("Enter consultation fee (Rs.): ")
        )
        test_fee = float(input("Enter test fee (Rs.): "))
        medicine_fee = float(input("Enter medicine fee (Rs.): "))

        fees = [consultation_fee, test_fee, medicine_fee]

        if any(fee < 0 for fee in fees):
            print("Fees cannot be negative.")
            return

    except ValueError:
        print("Please enter valid amounts.")
        return

    total = sum(fees)

    print("\n========== HOSPITAL BILL ==========")
    print("Patient ID:", patient_id)
    print("Patient Name:", patient[0])
    print(f"Consultation Fee: Rs. {consultation_fee:.2f}")
    print(f"Test Fee:         Rs. {test_fee:.2f}")
    print(f"Medicine Fee:     Rs. {medicine_fee:.2f}")
    print("-----------------------------------")
    print(f"TOTAL:            Rs. {total:.2f}")
    print("===================================")
    print("Demonstration bill only.")


def main():
    while True:
        print("\n")
        print("=" * 40)
        print("      HOSPITAL MANAGEMENT SYSTEM")
        print("=" * 40)

        print("1. Register Patient")
        print("2. View Patients")
        print("3. Search Patient")
        print("4. Update Patient")
        print("5. Delete Patient")
        print("6. Book Appointment")
        print("7. View Appointments")
        print("8. Generate Bill")
        print("9. Exit")

        choice = input("\nEnter your choice (1-9): ").strip()

        try:
            if choice == "1":
                register_patient()

            elif choice == "2":
                view_patients()

            elif choice == "3":
                search_patient()

            elif choice == "4":
                update_patient()

            elif choice == "5":
                delete_patient()

            elif choice == "6":
                book_appointment()

            elif choice == "7":
                view_appointments()

            elif choice == "8":
                generate_bill()

            elif choice == "9":
                print("Thank you for using the system!")
                break

            else:
                print("Invalid choice. Enter a number from 1 to 9.")

        except sqlite3.Error as error:
            conn.rollback()
            print("A database error occurred:", error)

    conn.close()


if __name__ == "__main__":
    main()