# oop-final
this is basicaaly restaurant management system
#include <iostream>
#include <vector>
#include <string>
#include <stdexcept>
#include <fstream>
#include <limits>
#include <unordered_map>

using namespace std;

// Customer Class
class Customer {
private:
    string name;
    int age;
    string cellNo;
    string address;

public:
    Customer(const string& name, int age, const string& cellNo, const string& address)
        : name(name), age(age), cellNo(cellNo), address(address) {}

    // Getters
    string getName() const { return name; }
    int getAge() const { return age; }
    string getCellNo() const { return cellNo; }
    string getAddress() const { return address; }

    // Setters
    void setAddress(const string& addr) { address = addr; }
};

// MenuItem Class
class MenuItem {
private:
    string name;
    double price;

public:
    MenuItem(const string& name, double price)
        : name(name), price(price) {}

    string getName() const { return name; }
    double getPrice() const { return price; }
};

// Menu Class
class Menu {
private:
    vector<MenuItem> mainItems;
    vector<MenuItem> drinks;
    vector<MenuItem> starters;
    vector<MenuItem> iceCreams;

public:
    void addMainItem(const MenuItem& item) {
        mainItems.push_back(item);
    }

    void addDrink(const MenuItem& item) {
        drinks.push_back(item);
    }

    void addStarter(const MenuItem& item) {
        starters.push_back(item);
    }

    void addIceCream(const MenuItem& item) {
        iceCreams.push_back(item);
    }

    void displayMenu() const {
        cout << "Main Items:\n";
        for (size_t i = 0; i < mainItems.size(); ++i) {
            cout << mainItems[i].getName() << ": " << mainItems[i].getPrice() << " PKR\n";
        }
        cout << "\nDrinks:\n";
        for (size_t i = 0; i < drinks.size(); ++i) {
            cout << drinks[i].getName() << ": " << drinks[i].getPrice() << " PKR\n";
        }
        cout << "\nStarters:\n";
        for (size_t i = 0; i < starters.size(); ++i) {
            cout << starters[i].getName() << ": " << starters[i].getPrice() << " PKR\n";
        }
        cout << "\nIce Creams:\n";
        for (size_t i = 0; i < iceCreams.size(); ++i) {
            cout << iceCreams[i].getName() << ": " << iceCreams[i].getPrice() << " PKR\n";
        }
    }

    MenuItem findItemByName(const string& name) const {
        for (size_t i = 0; i < mainItems.size(); ++i) {
            if (mainItems[i].getName() == name) {
                return mainItems[i];
            }
        }
        for (size_t i = 0; i < drinks.size(); ++i) {
            if (drinks[i].getName() == name) {
                return drinks[i];
            }
        }
        for (size_t i = 0; i < starters.size(); ++i) {
            if (starters[i].getName() == name) {
                return starters[i];
            }
        }
        for (size_t i = 0; i < iceCreams.size(); ++i) {
            if (iceCreams[i].getName() == name) {
                return iceCreams[i];
            }
        }
        throw runtime_error("Item not found");
    }
};

// Abstract Order Class
class Order {
protected:
    Customer customer;
    typedef pair<MenuItem, int> OrderItem; // Pair to store MenuItem and quantity
    vector<OrderItem> items;

public:
    Order(const Customer& customer) : customer(customer) {}
    virtual ~Order() {}

    void addItem(const MenuItem& item, int quantity) {
        items.push_back(OrderItem(item, quantity));
    }

    virtual void confirmOrder() = 0; // Pure virtual function

    double calculateTotal() const {
        double total = 0;
        for (size_t i = 0; i < items.size(); ++i) {
            total += items[i].first.getPrice() * items[i].second;
        }
        return total;
    }
    void displayOrder() const {
        cout << "Order Details:\n";
        for (size_t i = 0; i < items.size(); ++i) {
            cout << items[i].first.getName() << " x " << items[i].second << " = "
                << items[i].first.getPrice() * items[i].second << " PKR\n";
        }
        cout << "Total Bill: " << calculateTotal() << " PKR\n";
    }
};

// PickupOrder Class
class PickupOrder : public Order {
public:
    PickupOrder(const Customer& customer) : Order(customer) {}

    void confirmOrder() override {
        cout << "Order confirmed for pickup.\n";
    }
};

// DeliveryOrder Class
class DeliveryOrder : public Order {
private:
    string deliveryAddress;

public:
    DeliveryOrder(const Customer& customer, const string& deliveryAddress)
        : Order(customer), deliveryAddress(deliveryAddress) {}

    void confirmOrder() override {
        cout << "Order confirmed for delivery to: " << deliveryAddress << "\n";
    }
};

// User Management Class
class UserManager {
private:
    unordered_map<string, string> userDatabase;

    void loadUserData() {
        ifstream inFile("userData.txt");
        if (!inFile.is_open()) return;

        string username, password;
        while (inFile >> username >> password) {
            userDatabase[username] = password;
        }
        inFile.close();
    }

    void saveUserData() const {
        ofstream outFile("userData.txt");
        for (const auto& user : userDatabase) {
            outFile << user.first << " " << user.second << "\n";
        }
        outFile.close();
    }

public:
    UserManager() {
        loadUserData();
    }

    ~UserManager() {
        saveUserData();
    }

    bool registerUser(const string& username, const string& password) {
        if (userDatabase.find(username) != userDatabase.end()) {
            cout << "Username already exists. Please choose a different username.\n";
            return false;
        }
        userDatabase[username] = password;
        saveUserData();
        cout << "User registered successfully.\n";
        return true;
    }

    bool authenticateUser(const string& username, const string& password) {
        if (userDatabase.find(username) == userDatabase.end()) {
            cout << "Username not found.\n";
            return false;
        }
        if (userDatabase[username] != password) {
            cout << "Incorrect password.\n";
            return false;
        }
        cout << "User authenticated successfully.\n";
        return true;
    }
};

// Cheezious Class
class Cheezious {
private:
    Menu menu;
    int orderCount;
    UserManager userManager;

public:
    Cheezious() : orderCount(0) {
        // Initialize the menu with items
        menu.addMainItem(MenuItem("Burger", 300));
        menu.addMainItem(MenuItem("Pizza", 800));
        menu.addMainItem(MenuItem("Fries", 150));
        menu.addMainItem(MenuItem("Sandwich", 200));
        menu.addMainItem(MenuItem("Pasta", 500));

        // Drinks
        menu.addDrink(MenuItem("Coke", 50));
        menu.addDrink(MenuItem("Pepsi", 50));
        menu.addDrink(MenuItem("Lemonade", 70));

        // Starters
        menu.addStarter(MenuItem("Rolls", 120));
        menu.addStarter(MenuItem("Bread", 150));
        menu.addStarter(MenuItem("CreamStick", 700));

        // Ice Creams
        menu.addIceCream(MenuItem("Vanilla", 100));
        menu.addIceCream(MenuItem("Chocolate", 120));
        menu.addIceCream(MenuItem("Strawberry", 110));
    }

    void userRegistration() {
        string username, password;
        cout << "Register a new account\n";
        cout << "Enter username: ";
        cin >> username;
        cout << "Enter password: ";
        cin >> password;
        userManager.registerUser(username, password);
    }

    bool userLogin() {
        string username, password;
        cout << "Login to your account\n";
        cout << "Enter username: ";
        cin >> username;
        cout << "Enter password: ";
        cin >> password;
        return userManager.authenticateUser(username, password);
    }

    void takeOrder() {
        char firstTime;
        cout << "Is this your first time using our service? (Y/N): ";
        cin >> firstTime;
        if (firstTime == 'Y' || firstTime == 'y') {
            userRegistration();
        }

        while (true) {
            if (!userLogin()) {
                char choice;
                cout << "Do you want to retry login or register? (L for login, R for register): ";
                cin >> choice;
                if (choice == 'R' || choice == 'r') {
                    userRegistration();
                } else {
                    continue;
                }
            }

            string name, cellNo, address;
            int age;
            char orderType;

            cout << "Welcome to Cheezious!\n";

            cout << "Enter your name: ";
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            getline(cin, name);
            cout << "Enter your age: ";
            while (!(cin >> age)) {
                cout << "Invalid input. Please enter a valid age: ";
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
            }
            cout << "Enter your cell number: ";
            cin >> cellNo;

            Customer customer(name, age, cellNo, "");

            cout << "Do you want delivery or pickup? (D/P): ";
            cin >> orderType;

            Order* order;
            if (orderType == 'D' || orderType == 'd') {
                cout << "Enter your address: ";
                cin.ignore(); // Clear newline character from the input buffer
                getline(cin, address);
                customer.setAddress(address);
                order = new DeliveryOrder(customer, address);
            } else {
                order = new PickupOrder(customer);
            }

            menu.displayMenu();
            cout << "Enter item names and quantities to add to your order (type 'done' to finish):\n";

            string itemName;
            int quantity;
            while (true) {
                cout << "Item: ";
                cin >> itemName;
                if (itemName == "done") break;
                cout << "Quantity: ";
                while (!(cin >> quantity) || quantity <= 0) {
                    cout << "Invalid input. Please enter a valid quantity: ";
                    cin.clear();
                    cin.ignore(numeric_limits<streamsize>::max(), '\n');
                }
                try {
                    MenuItem item = menu.findItemByName(itemName);
                    order->addItem(item, quantity);
                } catch (const runtime_error& e) {
                    cout << "Item not found. Please enter a valid item name.\n";
                }
            }

            order->confirmOrder();
            order->displayOrder();
            cout << "Your order number is: " << ++orderCount << "\n";
            cout << "Please wait for 15-20 minutes.\n";

            delete order;

            char anotherOrder;
            cout << "Would you like to place another order? (Y/N): ";
            cin >> anotherOrder;
            if (anotherOrder == 'N' || anotherOrder == 'n') {
                cout << "Thank you for visiting Cheezious! Have a great day!\n";
                break;
            }
        }
    }
};

int main() {
    Cheezious cheezious;
    cheezious.takeOrder();
    return 0;
}
