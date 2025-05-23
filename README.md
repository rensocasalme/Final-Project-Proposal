// Casalme, Renso Anthony L.
// Blance, Marc Andrei T

#include <iostream>
#include <vector>
#include <string>
#include <memory>
#include <stdexcept>

using namespace std;

// Abstract Base Class for Room
class RoomBase {
protected:
    int id;
    double costPerNight;
    bool available;

public:
    RoomBase(int roomID, double rate) : id(roomID), costPerNight(rate), available(true) {}
    virtual void showDetails() const = 0;
    virtual double computeCharge(int nights) const = 0;

    bool isRoomAvailable() const { return available; }
    void reserve() { 
        if (!available) throw runtime_error("Room already reserved."); 
        available = false; 
    }
    void release() { available = true; }
    int getID() const { return id; }
};

class Deluxe : public RoomBase {
public:
    Deluxe(int roomID) : RoomBase(roomID, 150.0) {}
    void showDetails() const override {
        cout << "Deluxe Room " << id << ", Rate: $" << costPerNight 
             << ", Available: " << (available ? "Yes" : "No") << endl;
    }
    double computeCharge(int nights) const override {
        return nights * costPerNight;
    }
};

class Suite : public RoomBase {
public:
    Suite(int roomID) : RoomBase(roomID, 300.0) {}
    void showDetails() const override {
        cout << "Suite Room " << id << ", Rate: $" << costPerNight 
             << ", Available: " << (available ? "Yes" : "No") << endl;
    }
    double computeCharge(int nights) const override {
        return (nights * costPerNight) + 100;
    }
};

// Strategy for Billing
class Billing {
public:
    virtual double calculate(const RoomBase* room, int nights) const = 0;
};

class RegularBilling : public Billing {
public:
    double calculate(const RoomBase* room, int nights) const override {
        return room->computeCharge(nights);
    }
};

// User encapsulation
class Account {
private:
    string name;
    string secret;

public:
    Account(string uname, string pwd) : name(uname), secret(pwd) {}
    string getName() const { return name; }
    bool verifyPassword(const string& pwd) const { return secret == pwd; }
};

// Hotel Management System
class HotelApp {
private:
    vector<shared_ptr<RoomBase>> inventory;
    vector<Account> accounts;
    shared_ptr<Billing> billingPolicy;

public:
    HotelApp() {
        for (int i = 1; i <= 3; ++i) inventory.push_back(make_shared<Deluxe>(i));
        for (int i = 4; i <= 5; ++i) inventory.push_back(make_shared<Suite>(i));
        billingPolicy = make_shared<RegularBilling>();
    }

    void registerUser(const string& uname, const string& pwd) {
        accounts.emplace_back(uname, pwd);
        cout << "Signup successful!\n";
    }

    bool authenticate(const string& uname, const string& pwd) {
        for (const auto& user : accounts) {
            if (user.getName() == uname && user.verifyPassword(pwd)) {
                cout << "Login successful!\n";
                return true;
            }
        }
        cout << "Login failed!\n";
        return false;
    }

    void listAvailableRooms() const {
        for (const auto& room : inventory) {
            if (room->isRoomAvailable())
                room->showDetails();
        }
    }

    void reserveRoom(int roomID) {
        for (const auto& room : inventory) {
            if (room->getID() == roomID) {
                room->reserve();
                cout << "Room " << roomID << " reserved successfully.\n";
                return;
            }
        }
        throw runtime_error("Invalid room number.");
    }

    void showBill(int roomID, int nights) {
        for (const auto& room : inventory) {
            if (room->getID() == roomID) {
                cout << "Total Charge: $" << billingPolicy->calculate(room.get(), nights) << endl;
                return;
            }
        }
        throw runtime_error("Room not found.");
    }
};

int main() {
    HotelApp app;
    string uname, pwd;

    cout << "===== Hotel Booking System =====\n";
    cout << "1. Register\n2. Login\nSelect Option: ";
    int option;
    cin >> option;
    cout << "Username: "; cin >> uname;
    cout << "Password: "; cin >> pwd;

    if (option == 1) app.registerUser(uname, pwd);
    if (!app.authenticate(uname, pwd)) return 0;

    while (true) {
        cout << "\n1. View Available Rooms\n2. Reserve Room\n3. Get Bill\n4. Exit\nOption: ";
        cin >> option;
        try {
            if (option == 1) {
                app.listAvailableRooms();
            } else if (option == 2) {
                int num;
                cout << "Enter Room ID: "; cin >> num;
                app.reserveRoom(num);
            } else if (option == 3) {
                int num, nights;
                cout << "Enter Room ID: "; cin >> num;
                cout << "Enter Days: "; cin >> nights;
                app.showBill(num, nights);
            } else if (option == 4) {
                break;
            } else {
                cout << "Invalid input.\n";
            }
        } catch (const exception& ex) {
            cerr << "Error: " << ex.what() << endl;
        }
    }

    return 0;
}
