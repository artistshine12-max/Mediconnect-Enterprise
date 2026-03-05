#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <limits>

using namespace std;

/* ================= UTILITY ================= */

string generateID(string prefix)
{
    static int counter = 1000;
    return prefix + to_string(counter++);
}

bool exists(string file)
{
    ifstream f(file);
    return f.good();
}

/* ================= SAMPLE DATA ================= */

void loadSampleData()
{
    if (!exists("doctors.txt"))
    {
        ofstream f("doctors.txt");
        f << "D101|Dr.Smith|Cardiologist|pass\n";
        f << "D102|Dr.Adams|Dermatologist|pass\n";
    }

    if (!exists("medicines.txt"))
    {
        ofstream f("medicines.txt");
        f << "M101|Paracetamol|50\n";
        f << "M102|Ibuprofen|80\n";
    }
}

/* ================= DISPLAY ================= */

void showDoctors()
{
    ifstream f("doctors.txt");
    string line;

    cout << "\nDoctors:\n";

    while (getline(f, line))
    {
        stringstream ss(line);
        string id, name, spec;

        getline(ss, id, '|');
        getline(ss, name, '|');
        getline(ss, spec, '|');

        cout << id << " | " << name << " | " << spec << endl;
    }
}

void showMedicines()
{
    ifstream f("medicines.txt");
    string line;

    cout << "\nMedicines:\n";

    while (getline(f, line))
    {
        stringstream ss(line);
        string id, name, price;

        getline(ss, id, '|');
        getline(ss, name, '|');
        getline(ss, price, '|');

        cout << id << " | " << name << " | Rs." << price << endl;
    }
}

/* ================= BILL ================= */

void createBill(string pid, string service, int amount)
{
    string id = generateID("B");

    ofstream f("bills.txt", ios::app);

    f << id << "|" << pid << "|" << service
      << "|" << amount << "|Unpaid\n";

    cout << "Bill Generated: " << id << endl;
}

void viewBills(string pid)
{
    ifstream f("bills.txt");
    string line;

    cout << "\nBills\n";

    while (getline(f, line))
    {
        stringstream ss(line);
        string bid, patient, service, amount, status;

        getline(ss, bid, '|');
        getline(ss, patient, '|');
        getline(ss, service, '|');
        getline(ss, amount, '|');
        getline(ss, status, '|');

        if (patient == pid)
        {
            cout << bid << " | "
                 << service << " | Rs."
                 << amount << " | "
                 << status << endl;
        }
    }
}

void payBill(string pid)
{
    string billID;

    cout << "Bill ID: ";
    cin >> billID;

    ifstream f("bills.txt");
    ofstream temp("temp.txt");

    string line;

    while (getline(f, line))
    {
        stringstream ss(line);
        string bid, patient, service, amount, status;

        getline(ss, bid, '|');
        getline(ss, patient, '|');
        getline(ss, service, '|');
        getline(ss, amount, '|');
        getline(ss, status, '|');

        if (bid == billID && patient == pid)
            status = "Paid";

        temp << bid << "|" << patient << "|"
             << service << "|" << amount
             << "|" << status << "\n";
    }

    f.close();
    temp.close();

    remove("bills.txt");
    rename("temp.txt", "bills.txt");

    cout << "Payment Successful\n";
}

/* ================= PATIENT ================= */

class Patient
{
public:
    static void registerPatient()
    {
        string id = generateID("P");
        string name, contact, pass;

        cin.ignore();

        cout << "Name: ";
        getline(cin, name);

        cout << "Contact: ";
        getline(cin, contact);

        cout << "Password: ";
        getline(cin, pass);

        ofstream f("patients.txt", ios::app);
        f << id << "|" << name << "|"
          << contact << "|" << pass << "\n";

        cout << "Registered! ID: " << id << endl;
    }

    static string login()
    {
        string id, pass;

        cout << "Patient ID: ";
        cin >> id;

        cout << "Password: ";
        cin >> pass;

        ifstream f("patients.txt");
        string line;

        while (getline(f, line))
        {
            stringstream ss(line);
            string pid, name, contact, pw;

            getline(ss, pid, '|');
            getline(ss, name, '|');
            getline(ss, contact, '|');
            getline(ss, pw, '|');

            if (pid == id && pw == pass)
                return id;
        }

        cout << "Invalid Login\n";
        return "";
    }
};

/* ================= APPOINTMENT ================= */

bool checkConflict(string did, string date, string time)
{
    ifstream f("appointments.txt");
    string line;

    while (getline(f, line))
    {
        stringstream ss(line);

        string aid, pid, doc, d, t;

        getline(ss, aid, '|');
        getline(ss, pid, '|');
        getline(ss, doc, '|');
        getline(ss, d, '|');
        getline(ss, t, '|');

        if (doc == did && d == date && t == time)
            return true;
    }

    return false;
}

void bookAppointment(string pid)
{
    showDoctors();

    string did, date, time;

    cout << "Doctor ID: ";
    cin >> did;

    cout << "Date: ";
    cin >> date;

    cout << "Time: ";
    cin >> time;

    if (checkConflict(did, date, time))
    {
        cout << "Doctor already booked!\n";
        return;
    }

    string id = generateID("A");

    ofstream f("appointments.txt", ios::app);

    f << id << "|" << pid << "|" << did
      << "|" << date << "|" << time
      << "|Booked\n";

    createBill(pid, "Consultation", 500);

    cout << "Appointment Booked\n";
}

/* ================= DOCTOR DASHBOARD ================= */

void doctorAppointments(string did)
{
    ifstream f("appointments.txt");
    string line;

    cout << "\nDoctor Appointments\n";

    while (getline(f, line))
    {
        stringstream ss(line);

        string aid, pid, doc, date, time, status;

        getline(ss, aid, '|');
        getline(ss, pid, '|');
        getline(ss, doc, '|');
        getline(ss, date, '|');
        getline(ss, time, '|');
        getline(ss, status, '|');

        if (doc == did)
        {
            cout << aid << " | "
                 << pid << " | "
                 << date << " "
                 << time << " | "
                 << status << endl;
        }
    }
}

/* ================= STAFF ================= */

void viewOrders()
{
    ifstream f("orders.txt");
    string line;

    cout << "\nMedicine Orders\n";

    while (getline(f, line))
        cout << line << endl;
}

void viewTransport()
{
    ifstream f("transport.txt");
    string line;

    cout << "\nAmbulance Bookings\n";

    while (getline(f, line))
        cout << line << endl;
}

void viewHomeVisits()
{
    ifstream f("homevisits.txt");
    string line;

    cout << "\nHome Visit Requests\n";

    while (getline(f, line))
        cout << line << endl;
}

/* ================= MENUS ================= */

void patientMenu(string pid)
{
    int ch;

    do
    {
        cout << "\nPATIENT MENU\n";
        cout << "1 Book Appointment\n";
        cout << "2 View Bills\n";
        cout << "3 Pay Bill\n";
        cout << "0 Logout\n";

        cin >> ch;

        if (ch == 1)
            bookAppointment(pid);

        if (ch == 2)
            viewBills(pid);

        if (ch == 3)
            payBill(pid);

    } while (ch != 0);
}

void doctorMenu(string did)
{
    int ch;

    do
    {
        cout << "\nDOCTOR DASHBOARD\n";
        cout << "1 View Appointments\n";
        cout << "0 Logout\n";

        cin >> ch;

        if (ch == 1)
            doctorAppointments(did);

    } while (ch != 0);
}

void staffMenu()
{
    int ch;

    do
    {
        cout << "\nSTAFF DASHBOARD\n";
        cout << "1 Medicine Orders\n";
        cout << "2 Ambulance Bookings\n";
        cout << "3 Home Visits\n";
        cout << "0 Logout\n";

        cin >> ch;

        if (ch == 1) viewOrders();
        if (ch == 2) viewTransport();
        if (ch == 3) viewHomeVisits();

    } while (ch != 0);
}

bool adminLogin()
{
    string u, p;

    cout << "Admin Username: ";
    cin >> u;

    cout << "Password: ";
    cin >> p;

    return u == "admin" && p == "admin123";
}

/* ================= MAIN ================= */

int main()
{
    cout << "===== MEDICONNECT++ =====\n";

    loadSampleData();

    int choice;

    do
    {
        cout << "\nMain Menu\n";
        cout << "1 Patient Register\n";
        cout << "2 Patient Login\n";
        cout << "3 Doctor Login\n";
        cout << "4 Staff Dashboard\n";
        cout << "5 Admin Login\n";
        cout << "0 Exit\n";

        cin >> choice;

        switch (choice)
        {
        case 1:
            Patient::registerPatient();
            break;

        case 2:
        {
            string pid = Patient::login();
            if (!pid.empty())
                patientMenu(pid);
            break;
        }

        case 3:
        {
            string did;
            cout << "Doctor ID: ";
            cin >> did;
            doctorMenu(did);
            break;
        }

        case 4:
            staffMenu();
            break;

        case 5:
            if (adminLogin())
                showDoctors();
            else
                cout << "Invalid Admin\n";
            break;
        }

    } while (choice != 0);

    return 0;
}
