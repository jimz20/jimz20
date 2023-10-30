#include <iostream>
#include <fstream>
#include <string>
#include <sstream>
#include <iomanip>
#include <windows.h>  // For Sleep function
 
// Define a function to encrypt the PIN (This is a simple example, not secure for production use)
std::string encryptPin(const std::string& pin, const std::string& key) {
    std::string encryptedPin;
    for (size_t i = 0; i < pin.length(); i++) {
        encryptedPin += static_cast<char>(pin[i] + key[i % key.length()]);
    }
    return encryptedPin;
}
 
// Define a function to decrypt the PIN (This is a simple example, not secure for production use)
std::string decryptPin(const std::string& encryptedPin, const std::string& key) {
    std::string decryptedPin;
    for (size_t i = 0; i < encryptedPin.length(); i++) {
        decryptedPin += static_cast<char>(encryptedPin[i] - key[i % key.length()]);
    }
    return decryptedPin;
}
 
bool isAllowedWithdrawalAmount(double amount) {
    return (amount == 100 || amount == 500 || amount == 1000 || amount == 2000 || amount == 3000 || amount == 5000);
}
 
bool isCardRegistered(const std::string& cardFileName, const std::string& accountFileName, const std::string& encryptionKey, int& accountPin) {
    std::ifstream cardFile(cardFileName);
    if (!cardFile.is_open()) {
        std::cerr << "ATM card file not found." << std::endl;
        return false;
    }
 
    std::string storedAccountNumber;
    std::string storedPin;
 
    // Read the account number and PIN from the file
    if (!(cardFile >> storedAccountNumber >> storedPin)) {
        std::cerr << "Error reading account number and PIN from the ATM card file." << std::endl;
        return false;
    }
 
    cardFile.close();
 
    std::ifstream accountFile(accountFileName);
    if (!accountFile.is_open()) {
        std::cerr << "Account file not found." << std::endl;
        return false;
    }
 
    // Encrypt the stored PIN for comparison
    std::string encryptedStoredPin = encryptPin(storedPin, encryptionKey);
 
    std::string line;
    while (std::getline(accountFile, line)) {
        if (line == encryptedStoredPin) {
            accountPin = std::stoi(decryptPin(line, encryptionKey));
            return true;
        }
    }
    accountFile.close();
 
    return false;
}
 
class Account {
private:
    double balance;
    int pin;
 
public:
    Account(double initialBalance, int initialPin) : balance(initialBalance), pin(initialPin) {}
 
    double getBalance() {
        return balance;
    }
 
    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
 
    bool deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            return true;
        }
        return false;
    }
 
    bool fundTransfer(Account& recipient, double amount) {
        if (amount > 0 && amount <= balance) {
            std::cout << "Enter your PIN for confirmation: ";
            int confirmationPin;
            std::cin >> confirmationPin;
 
            if (confirmationPin == pin) {
                if (recipient.getBalance() >= amount) {
                    balance -= amount;
                    recipient.deposit(amount);
                    return true;  // Transfer successful
                } else {
                    std::cout << "Recipient account has insufficient balance. Fund transfer canceled." << std::endl;
                }
            } else {
                std::cout << "PIN confirmation failed. Fund transfer canceled." << std::endl;
            }
        }
        return false;  // Transfer failed
    }
 
    bool changePIN(int newPin, const std::string& cardFileName, const std::string& encryptionKey) {
        if (newPin >= 1000 && newPin <= 9999) {
            pin = newPin;
 
            // Update the PIN in the card file
            std::ofstream cardFile(cardFileName);
            cardFile << encryptPin(std::to_string(pin), encryptionKey);
            cardFile.close();
 
            return true;
        }
        return false;
    }
 
    int getPIN() {
        return pin;
    }
};
 
// Function to find recipient account by account number
Account findRecipientAccount(const std::string& accountNumber, const std::string& accountFileName) {
    std::ifstream accountFile(accountFileName);  // Assuming this file contains account information
    if (!accountFile.is_open()) {
        std::cerr << "Account file not found." << std::endl;
        return Account(0.0, 0);  // Return an empty account
    }
 
    std::string line;
    while (std::getline(accountFile, line)) {
        // Parse the line to get account details
        std::istringstream iss(line);
        std::string accountNum;
        double balance;
        int pin;
        if (iss >> accountNum >> balance >> pin) {
            if (accountNum == accountNumber) {
                return Account(balance, pin);
            }
        }
    }
 
    accountFile.close();
 
    return Account(0.0, 0);  // Return an empty account if not found
}
 
int main() {
    std::string cardFileName = "D:\\pin.code";  // The name of the text file in the ATM card
    std::string accountFileName = "atml.txt";  // The name of the text file for registered accounts
    std::string encryptionKey = "MySecretKey"; // Replace with your encryption key
    std::string userPin;
    bool cardInserted = false;
    bool isRegistered = false;
    int accountPin;
 
    // Check if the ATM card is registered
    isRegistered = isCardRegistered(cardFileName, accountFileName, encryptionKey, accountPin);
 
    Account userAccount(5000.0, accountPin);
 
    while (true) {
        system("cls");  // Clear the screen
 
        if (cardInserted) {
            std::cout << "ATM Card Inserted." << std::endl;
        } else {
            std::cout << "Please insert card." << std::endl;
        }
 
        if (!cardInserted) {
            // Simulate card insertion by checking if the card file exists
            std::ifstream cardFile(cardFileName);
            cardInserted = cardFile.good();
            cardFile.close();
        }
 
        if (cardInserted) {
            std::cout << "Choose an option:" << std::endl;
            std::cout << "1. Balance Inquiry" << std::endl;
            std::cout << "2. Withdraw" << std::endl;
            std::cout << "3. Deposit" << std::endl;
            std::cout << "4. Fund Transfer" << std::endl;
            std::cout << "5. Change PIN Code" << std::endl;
            std::cout << "0. Exit" << std::endl;
            std::cout << "Enter your choice: ";
            int choice;
            std::cin >> choice;
 
            if (choice == 1) {
                // i. Balance Inquiry
                double balance = userAccount.getBalance();
                std::cout << "Your balance is $" << balance << std::endl;
                Sleep(2000);  // Pause for 2 seconds
            } else if (choice == 2) {
                // ii. Withdraw
                std::cout << "Enter the amount to withdraw: $";
                double withdrawalAmount;
                std::cin >> withdrawalAmount;
                if (isAllowedWithdrawalAmount(withdrawalAmount) && userAccount.withdraw(withdrawalAmount)) {
                    std::cout << "Successfully withdrew $" << withdrawalAmount << std::endl;
                } else {
                    std::cout << "Invalid amount for withdrawal or insufficient balance." << std::endl;
                }
                Sleep(2000);  // Pause for 2 seconds
            } else if (choice == 3) {
                // iii. Deposit
                std::cout << "Enter the amount to deposit: $";
                double depositAmount;
                std::cin >> depositAmount;
                if (depositAmount > 0 && userAccount.deposit(depositAmount)) {
                    std::cout << "Deposited $" << depositAmount << " into the account." << std::endl;
                } else {
                    std::cout << "Invalid amount for deposit." << std::endl;
                }
                Sleep(2000);  // Pause for 2 seconds
            } else if (choice == 4) {
                // iv. Fund Transfer
                int enteredPin;
                std::cout << "Enter your PIN: ";
                std::cin >> enteredPin;
 
                if (enteredPin == userAccount.getPIN()) {
                    std::cout << "Enter recipient's account number: ";
                    std::string recipientAccountNumber;
                    std::cin >> recipientAccountNumber;
 
                    // Find the recipient's account
                    Account recipientAccount = findRecipientAccount(recipientAccountNumber, accountFileName);
 
                    if (recipientAccount.getPIN() != 0) {
                        std::cout << "Enter your PIN for confirmation: ";
                        int confirmationPin;
                        std::cin >> confirmationPin;
 
                        if (confirmationPin == userAccount.getPIN()) {
                            // Prompt the user for the amount to transfer
                            double transferAmount;
                            std::cout << "Enter the amount to transfer: $";
                            std::cin >> transferAmount;
 
                            if (transferAmount > 0 && transferAmount <= userAccount.getBalance()) {
                                if (userAccount.fundTransfer(recipientAccount, transferAmount)) {
                                    std::cout << "Transferred $" << transferAmount << " to another account." << std::endl;
                                } else {
                                    std::cout << "Invalid amount for fund transfer or insufficient balance." << std::endl;
                                }
                            } else {
                                std::cout << "Invalid amount for fund transfer or insufficient balance." << std::endl;
                            }
                        } else {
                            std::cout << "PIN confirmation failed. Fund transfer canceled." << std::endl;
                        }
                    } else {
                        std::cout << "Recipient account not found or has no balance. Fund transfer canceled." << std::endl;
                    }
                }
                Sleep(2000);  // Pause for 2 seconds
            } else if (choice == 5) {
                // v. Change PIN Code
                if (cardInserted) {
                    int newPin;
                    std::cout << "Enter your new PIN code: ";
                    std::cin >> newPin;
                    if (userAccount.changePIN(newPin, cardFileName, encryptionKey)) {
                        std::cout << "PIN code changed successfully." << std::endl;
                    } else {
                        std::cout << "Invalid PIN. PIN must be 4 digits." << std::endl;
                    }
                    Sleep(2000);  // Pause for 2 seconds
                } else {
                    std::cout << "Card not inserted. Please insert your card first." << std::endl;
                    Sleep(2000);  // Pause for 2 seconds
                }
            } else if (choice == 0) {
                std::cout << "Exiting the program." << std::endl;
                break;
            } else {
                std::cout << "Invalid choice. Please try again." << std::endl;
                Sleep(2000);  // Pause for 2 seconds
            }
        } else {
            Sleep(2000);  // Pause for 2 seconds
        }
    }
 
    return 0;
}
