
#include <iostream>
#include <fstream>
#include <vector>
#include <iomanip>
#include <sstream>
#include <algorithm>
#include <ctime>
#include <cstdlib>

using namespace std;

class User {
private:
    string username;
    string password;
    string role;

public:
    User(string u, string p, string r = "user")
        : username(u), password(p), role(r) {}

    string getUsername() const { return username; }
    string getPassword() const { return password; }
    string getRole() const { return role; }

    string toString() const {
        return username + "|" + password + "|" + role;
    }

    static User fromString(string line) {
        size_t pos1 = line.find('|');
        size_t pos2 = line.find('|', pos1 + 1);

        string u = line.substr(0, pos1);
        string p = line.substr(pos1 + 1, pos2 - pos1 - 1);
        string r = line.substr(pos2 + 1);

        return User(u, p, r);
    }
};

class Expense {
private:
    double amount;
    string category;
    string date;
    string description;
    string username;

public:
    Expense(double amt, string cat, string dt, string desc, string user = "")
        : amount(amt), category(cat), date(dt), description(desc), username(user) {}

    double getAmount() const { return amount; }
    string getCategory() const { return category; }
    string getDate() const { return date; }
    string getDescription() const { return description; }
    string getUsername() const { return username; }

    void display() const {
        cout << fixed << setprecision(2);
        cout << setw(10) << amount << " | "
             << setw(15) << category << " | "
             << setw(12) << date << " | "
             << setw(20) << username << " | "
             << description << endl;
    }

    string toString() const {
        stringstream ss;
        ss << fixed << setprecision(2);
        ss << amount << "|" << category << "|" << date << "|" << username << "|" << description;
        return ss.str();
    }

    static Expense fromString(string line) {
        size_t pos1 = line.find('|');
        size_t pos2 = line.find('|', pos1 + 1);
        size_t pos3 = line.find('|', pos2 + 1);
        size_t pos4 = line.find('|', pos3 + 1);

        double amt = stod(line.substr(0, pos1));
        string cat = line.substr(pos1 + 1, pos2 - pos1 - 1);
        string dt = line.substr(pos2 + 1, pos3 - pos2 - 1);
        string user = line.substr(pos3 + 1, pos4 - pos3 - 1);
        string desc = line.substr(pos4 + 1);

        return Expense(amt, cat, dt, desc, user);
    }
};

class ExpenseTracker {
private:
    vector<Expense> expenses;
    vector<User> users;
    const string EXPENSE_FILE = "expenses.txt";
    const string USER_FILE = "users.txt";
    const string SUMMARY_FILE = "summary_report.txt";
    const vector<string> CATEGORIES = {"food", "transportation", "school", "bills", "leisure", "other"};
    string currentUser;
    string userRole;

public:
    ExpenseTracker() {
        loadUsers();
        loadExpenses();
    }

    void clearScreen() {
        #ifdef _WIN32
            system("cls");
        #else
            system("clear");
        #endif
    }

    bool validateDate(string dateStr) {
        if (dateStr.length() != 10) return false;
        if (dateStr[4] != '-' || dateStr[7] != '-') return false;

        for (int i = 0; i < 10; i++) {
            if (i == 4 || i == 7) continue;
            if (!isdigit(dateStr[i])) return false;
        }

        int year = stoi(dateStr.substr(0, 4));
        int month = stoi(dateStr.substr(5, 2));
        int day = stoi(dateStr.substr(8, 2));

        if (month < 1 || month > 12) return false;
        if (day < 1 || day > 31) return false;

        return true;
    }

    bool validateAmount(string amountStr) {
        try {
            double amount = stod(amountStr);
            return amount > 0;
        } catch (...) {
            return false;
        }
    }

    string getTodayDate() {
        time_t now = time(0);
        tm* timeinfo = localtime(&now);
        stringstream ss;
        ss << (1900 + timeinfo->tm_year) << "-"
           << setfill('0') << setw(2) << (1 + timeinfo->tm_mon) << "-"
           << setfill('0') << setw(2) << timeinfo->tm_mday;
        return ss.str();
    }

    string toLower(string str) {
        transform(str.begin(), str.end(), str.begin(), ::tolower);
        return str;
    }

    bool userExists(string username) {
        for (const auto& u : users) {
            if (u.getUsername() == username) {
                return true;
            }
        }
        return false;
    }

    bool authenticateUser(string username, string password) {
        for (const auto& u : users) {
            if (u.getUsername() == username && u.getPassword() == password) {
                userRole = u.getRole();
                return true;
            }
        }
        return false;
    }

    void registerUser() {
        clearScreen();
        cout << "\n========================================\n";
        cout << "           USER REGISTRATION\n";
        cout << "========================================\n\n";

        string username;
        while (true) {
            cout << "Enter username: ";
            getline(cin, username);
            if (userExists(username)) {
                cout << "Username already exists. Try another one.\n";
                continue;
            }
            if (username.empty()) {
                cout << "Username cannot be empty.\n";
                continue;
            }
            break;
        }

        string password;
        while (true) {
            cout << "Enter password: ";
            getline(cin, password);
            if (password.empty()) {
                cout << "Password cannot be empty.\n";
                continue;
            }
            break;
        }

        users.push_back(User(username, password, "user"));
        saveUsers();
        cout << "\nUser registered successfully!\n";
        cout << "Press Enter to continue...";
        cin.ignore();
    }

    void login() {
        clearScreen();
        cout << "\n========================================\n";
        cout << "  PERSONAL EXPENSES TRACKER SYSTEM\n";
        cout << "========================================\n\n";

        cout << "--- LOGIN ---\n";
        cout << "1. Login as existing user\n";
        cout << "2. Register new user\n";
        cout << "3. Admin login\n";
        cout << "\nSelect option (1-3): ";

        string choice;
        getline(cin, choice);

        if (choice == "1") {
            cout << "\nEnter username: ";
            getline(cin, currentUser);
            cout << "Enter password: ";
            string password;
            getline(cin, password);

            if (authenticateUser(currentUser, password)) {
                cout << "\nLogin successful!\n";
                cin.ignore();
            } else {
                cout << "\nInvalid username or password.\n";
                cin.ignore();
                login();
            }
        } else if (choice == "2") {
            registerUser();
            login();
        } else if (choice == "3") {
            cout << "\nEnter admin username: ";
            getline(cin, currentUser);
            cout << "Enter admin password: ";
            string password;
            getline(cin, password);

            if (currentUser == "admin" && password == "admin123") {
                userRole = "admin";
                cout << "\nAdmin login successful!\n";
                cin.ignore();
            } else {
                cout << "\nInvalid admin credentials.\n";
                cin.ignore();
                login();
            }
        } else {
            cout << "\nInvalid choice.\n";
            cin.ignore();
            login();
        }
    }

    void addExpense() {
        clearScreen();
        cout << "\n--- ADD EXPENSE ---\n";

        double amount;
        while (true) {
            cout << "Enter amount: ";
            string input;
            getline(cin, input);
            if (validateAmount(input)) {
                amount = stod(input);
                break;
            }
            cout << "Invalid amount. Please enter a positive number.\n";
        }

        cout << "Categories: food, transportation, school, bills, leisure, other\n";
        string category;
        while (true) {
            cout << "Enter category: ";
            getline(cin, category);
            category = toLower(category);
            if (find(CATEGORIES.begin(), CATEGORIES.end(), category) != CATEGORIES.end()) {
                break;
            }
            cout << "Invalid category. Please choose from the list above.\n";
        }

        string date;
        while (true) {
            cout << "Enter date (YYYY-MM-DD) or press Enter for today: ";
            getline(cin, date);
            if (date.empty()) {
                date = getTodayDate();
                break;
            }
            if (validateDate(date)) {
                break;
            }
            cout << "Invalid date format. Use YYYY-MM-DD (e.g., 2025-12-07).\n";
        }

        string description;
        cout << "Enter description: ";
        getline(cin, description);
        if (description.empty()) {
            description = "No description";
        }

        expenses.push_back(Expense(amount, category, date, description, currentUser));
        cout << "\nExpense added successfully!\n";
        saveExpenses();
        cout << "Press Enter to continue...";
        cin.ignore();
    }

    void viewAllExpenses() {
        clearScreen();
        vector<Expense> userExpenses;

        if (userRole == "user") {
            for (const auto& e : expenses) {
                if (e.getUsername() == currentUser) {
                    userExpenses.push_back(e);
                }
            }
        } else {
            userExpenses = expenses;
        }

        if (userExpenses.empty()) {
            cout << "\n--- VIEW ALL EXPENSES ---\n";
            cout << "No expenses recorded yet.\n";
            cout << "Press Enter to continue...";
            cin.ignore();
            return;
        }

        cout << "\n--- ALL EXPENSES ---\n";
        cout << setw(3) << "#" << " | "
             << setw(10) << "Amount" << " | "
             << setw(15) << "Category" << " | "
             << setw(12) << "Date" << " | "
             << setw(20) << "Username" << " | "
             << "Description\n";
        cout << string(100, '-') << "\n";

        double total = 0;
        for (size_t i = 0; i < userExpenses.size(); i++) {
            cout << setw(3) << (i + 1) << " | ";
            userExpenses[i].display();
            total += userExpenses[i].getAmount();
        }

        cout << string(100, '-') << "\n";
        cout << "Total: " << fixed << setprecision(2) << total << "\n\n";
        cout << "Press Enter to continue...";
        cin.ignore();
    }

    void searchExpenses() {
        clearScreen();
        cout << "\n--- SEARCH EXPENSES ---\n";
        cout << "Search by (1) Category or (2) Date? Enter 1 or 2: ";
        string choice;
        getline(cin, choice);

        vector<Expense> searchPool;
        if (userRole == "user") {
            for (const auto& e : expenses) {
                if (e.getUsername() == currentUser) {
                    searchPool.push_back(e);
                }
            }
        } else {
            searchPool = expenses;
        }

        if (choice == "1") {
            cout << "Enter category to search: ";
            string category;
            getline(cin, category);
            category = toLower(category);

            vector<Expense> results;
            for (const auto& e : searchPool) {
                if (e.getCategory() == category) {
                    results.push_back(e);
                }
            }

            if (results.empty()) {
                cout << "No expenses found for category: " << category << "\n";
                cout << "Press Enter to continue...";
                cin.ignore();
                return;
            }

            cout << "\nExpenses in " << category << " category:\n";
            cout << setw(3) << "#" << " | "
                 << setw(10) << "Amount" << " | "
                 << setw(15) << "Category" << " | "
                 << setw(12) << "Date" << " | "
                 << "Description\n";
            cout << string(80, '-') << "\n";

            double subtotal = 0;
            for (size_t i = 0; i < results.size(); i++) {
                cout << setw(3) << (i + 1) << " | "
                     << fixed << setprecision(2) << setw(9) << results[i].getAmount() << " | "
                     << setw(15) << results[i].getCategory() << " | "
                     << setw(12) << results[i].getDate() << " | "
                     << results[i].getDescription() << "\n";
                subtotal += results[i].getAmount();
            }

            cout << string(80, '-') << "\n";
            cout << "Subtotal: " << fixed << setprecision(2) << subtotal << "\n\n";

        } else if (choice == "2") {
            cout << "Enter date to search (YYYY-MM-DD): ";
            string date;
            getline(cin, date);

            if (!validateDate(date)) {
                cout << "Invalid date format. Use YYYY-MM-DD.\n";
                cout << "Press Enter to continue...";
                cin.ignore();
                return;
            }

            vector<Expense> results;
            for (const auto& e : searchPool) {
                if (e.getDate() == date) {
                    results.push_back(e);
                }
            }

            if (results.empty()) {
                cout << "No expenses found for date: " << date << "\n";
                cout << "Press Enter to continue...";
                cin.ignore();
                return;
            }

            cout << "\nExpenses on " << date << ":\n";
            cout << setw(3) << "#" << " | "
                 << setw(10) << "Amount" << " | "
                 << setw(15) << "Category" << " | "
                 << "Description\n";
            cout << string(80, '-') << "\n";

            double subtotal = 0;
            for (size_t i = 0; i < results.size(); i++) {
                cout << setw(3) << (i + 1) << " | "
                     << fixed << setprecision(2) << setw(9) << results[i].getAmount() << " | "
                     << setw(15) << results[i].getCategory() << " | "
                     << results[i].getDescription() << "\n";
                subtotal += results[i].getAmount();
            }

            cout << string(80, '-') << "\n";
            cout << "Subtotal: " << fixed << setprecision(2) << subtotal << "\n\n";

        } else {
            cout << "Invalid choice.\n";
        }
        cout << "Press Enter to continue...";
        cin.ignore();
    }

    void calculateTotals() {
        clearScreen();

        vector<Expense> userExpenses;
        if (userRole == "user") {
            for (const auto& e : expenses) {
                if (e.getUsername() == currentUser) {
                    userExpenses.push_back(e);
                }
            }
        } else {
            userExpenses = expenses;
        }

        if (userExpenses.empty()) {
            cout << "\n--- CALCULATE TOTALS ---\n";
            cout << "No expenses recorded yet.\n";
            cout << "Press Enter to continue...";
            cin.ignore();
            return;
        }

        cout << "\n--- EXPENSE TOTALS ---\n";

        double total = 0;
        for (const auto& e : userExpenses) {
            total += e.getAmount();
        }

        cout << "Overall Total: " << fixed << setprecision(2) << total << "\n\n";
        cout << "Breakdown by Category:\n";
        cout << string(40, '-') << "\n";

        for (const auto& cat : CATEGORIES) {
            double categoryTotal = 0;
            for (const auto& e : userExpenses) {
                if (e.getCategory() == cat) {
                    categoryTotal += e.getAmount();
                }
            }

            if (categoryTotal > 0) {
                double percentage = (categoryTotal / total) * 100;
                cout << setw(15) << cat << " "
                     << fixed << setprecision(2) << setw(10) << categoryTotal
                     << " (" << fixed << setprecision(1) << percentage << "%)\n";
            }
        }
        cout << string(40, '-') << "\n\n";
        cout << "Press Enter to continue...";
        cin.ignore();
    }

    void generateSummaryReport() {
        clearScreen();
        cout << "\n--- GENERATING SUMMARY REPORT ---\n";

        ofstream file(SUMMARY_FILE);
        if (!file.is_open()) {
            cout << "Error creating summary report file.\n";
            cout << "Press Enter to continue...";
            cin.ignore();
            return;
        }

        vector<Expense> userExpenses;
        if (userRole == "user") {
            for (const auto& e : expenses) {
                if (e.getUsername() == currentUser) {
                    userExpenses.push_back(e);
                }
            }
        } else {
            userExpenses = expenses;
        }

        file << string(70, '=') << "\n";
        file << "              EXPENSE SUMMARY REPORT\n";
        file << string(70, '=') << "\n";
        file << "Generated on: " << getTodayDate() << "\n";
        if (userRole == "user") {
            file << "User: " << currentUser << "\n";
        } else {
            file << "Report Type: Administrator Full Report\n";
        }
        file << string(70, '=') << "\n\n";

        if (userExpenses.empty()) {
            file << "No expenses recorded.\n";
            file.close();
            cout << "Empty summary report generated.\n";
            cout << "Report saved to: " << SUMMARY_FILE << "\n";
            cout << "Press Enter to continue...";
            cin.ignore();
            return;
        }

        file << "DETAILED EXPENSES:\n";
        file << string(70, '-') << "\n";
        file << setw(3) << "#" << " | "
             << setw(10) << "Amount" << " | "
             << setw(15) << "Category" << " | "
             << setw(12) << "Date" << " | "
             << setw(20) << "Username" << " | "
             << "Description\n";
        file << string(120, '-') << "\n";

        double total = 0;
        for (size_t i = 0; i < userExpenses.size(); i++) {
            file << fixed << setprecision(2);
            file << setw(3) << (i + 1) << " | "
                 << setw(10) << userExpenses[i].getAmount() << " | "
                 << setw(15) << userExpenses[i].getCategory() << " | "
                 << setw(12) << userExpenses[i].getDate() << " | "
                 << setw(20) << userExpenses[i].getUsername() << " | "
                 << userExpenses[i].getDescription() << "\n";
            total += userExpenses[i].getAmount();
        }

        file << string(120, '-') << "\n";
        file << "Total Expenses: " << fixed << setprecision(2) << total << "\n\n";

        file << "CATEGORY BREAKDOWN:\n";
        file << string(70, '-') << "\n";

        for (const auto& cat : CATEGORIES) {
            double categoryTotal = 0;
            for (const auto& e : userExpenses) {
                if (e.getCategory() == cat) {
                    categoryTotal += e.getAmount();
                }
            }

            if (categoryTotal > 0) {
                double percentage = (categoryTotal / total) * 100;
                file << setw(15) << cat << ": " << fixed << setprecision(2)
                     << setw(10) << categoryTotal << " (" << fixed << setprecision(1) << percentage << "%)\n";
            }
        }

        file << "\n" << string(70, '=') << "\n";
        file << "End of Report\n";
        file << string(70, '=') << "\n";

        file.close();
        cout << "Summary report generated successfully!\n";
        cout << "Report saved to: " << SUMMARY_FILE << "\n";
        cout << "Press Enter to continue...";
        cin.ignore();
    }

    void saveUsers() {
        ofstream file(USER_FILE);
        if (!file.is_open()) {
            cout << "Error saving users.\n";
            return;
        }

        for (const auto& u : users) {
            file << u.toString() << "\n";
        }

        file.close();
    }

    void loadUsers() {
        ifstream file(USER_FILE);
        if (!file.is_open()) {
            return;
        }

        string line;
        while (getline(file, line)) {
            if (!line.empty()) {
                users.push_back(User::fromString(line));
            }
        }

        file.close();
    }

    void saveExpenses() {
        ofstream file(EXPENSE_FILE);
        if (!file.is_open()) {
            cout << "Error saving expenses.\n";
            return;
        }

        for (const auto& e : expenses) {
            file << e.toString() << "\n";
        }

        file.close();
    }

    void loadExpenses() {
        ifstream file(EXPENSE_FILE);
        if (!file.is_open()) {
            return;
        }

        string line;
        int count = 0;
        while (getline(file, line)) {
            if (!line.empty()) {
                expenses.push_back(Expense::fromString(line));
                count++;
            }
        }

        file.close();
        if (count > 0) {
            cout << "Loaded " << count << " expense(s) from file.\n";
            cin.ignore();
        }
    }


    void deleteExpense() {
        clearScreen();
        cout << "\n--- DELETE EXPENSE ---\n";

        vector<Expense> userExpenses;
        if (userRole == "user") {
            for (const auto& e : expenses) {
                if (e.getUsername() == currentUser) {
                    userExpenses.push_back(e);
                }
            }
        } else {
            userExpenses = expenses;
        }

        if (userExpenses.empty()) {
            cout << "No expenses available to delete.\n";
            cout << "Press Enter to continue...";
            cin.ignore();
            return;
        }

        cout << setw(3) << "#" << " | "
             << setw(10) << "Amount" << " | "
             << setw(15) << "Category" << " | "
             << setw(12) << "Date" << " | "
             << "Description\n";
        cout << string(80, '-') << "\n";

        for (size_t i = 0; i < userExpenses.size(); i++) {
            cout << setw(3) << (i + 1) << " | ";
            userExpenses[i].display();
        }

        cout << "\nEnter the number of the expense to delete (0 to cancel): ";
        string input;
        getline(cin, input);
        int choice = stoi(input);

        if (choice <= 0 || choice > (int)userExpenses.size()) {
            cout << "Deletion canceled.\n";
            cout << "Press Enter to continue...";
            cin.ignore();
            return;
        }

        Expense toDelete = userExpenses[choice - 1];
        auto it = find_if(expenses.begin(), expenses.end(),
                          [&](const Expense& e) {
                              return e.getAmount() == toDelete.getAmount() &&
                                     e.getCategory() == toDelete.getCategory() &&
                                     e.getDate() == toDelete.getDate() &&
                                     e.getDescription() == toDelete.getDescription() &&
                                     e.getUsername() == toDelete.getUsername();
                          });

        if (it != expenses.end()) {
            expenses.erase(it);
            saveExpenses();
            cout << "Expense deleted successfully!\n";
        } else {
            cout << "Error: Expense not found.\n";
        }

        cout << "Press Enter to continue...";
        cin.ignore();
    }


    void run() {
        login();

        while (true) {
            clearScreen();
            cout << "\n========================================\n";
            cout << "  PERSONAL EXPENSES TRACKER SYSTEM\n";
            cout << "User: " << currentUser << " [" << (userRole == "admin" ? "ADMIN" : "USER") << "]\n";
            cout << "========================================\n";

            cout << "\n--- MAIN MENU ---\n";
            cout << "1. Add Expense\n";
            cout << "2. View All Expenses\n";
            cout << "3. Search Expenses\n";
            cout << "4. Calculate Totals\n";
            cout << "5. Generate Summary Report\n";
            cout << "6. Delete Expense\n";
            cout << "7. Logout\n";
            cout << "\nEnter your choice (1-7): ";
            string choice;
            getline(cin, choice);

        if (choice == "1") {
            addExpense();
        } else if (choice == "2") {
            viewAllExpenses();
        } else if (choice == "3") {
            searchExpenses();
        } else if (choice == "4") {
            calculateTotals();
        } else if (choice == "5") {
            generateSummaryReport();
        } else if (choice == "6") {
            deleteExpense();
        } else if (choice == "7") {
            cout << "\nThank you for using Expense Tracker. Goodbye!\n";
            break;
        } else {
            cout << "Invalid choice. Please enter a number between 1 and 7.\n";
            cin.ignore();
            }
        }
    }
};

int main() {
    ExpenseTracker tracker;
    tracker.run();
    return 0;
}


