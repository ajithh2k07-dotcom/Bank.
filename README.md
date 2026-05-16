#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <math.h>

#define MAX 100
#define MIN_BAL 500
#define MAX_PIN_ATTEMPTS 3
#define MINI_STMT_COUNT 5

// ─────────────────────────────────────────────
// Structure
// ─────────────────────────────────────────────
typedef struct {
    int   accNo;
    char  name[50];
    float balance;
    int   pin;
    int   locked;       // 1 = locked due to failed PIN attempts
    char  accType[20];  // "Savings" or "Current"
} Account;

// ─────────────────────────────────────────────
// Global Variables
// ─────────────────────────────────────────────
Account accounts[MAX];
int count = 0;

// ─────────────────────────────────────────────
// Function Prototypes
// ─────────────────────────────────────────────
void  loadAccounts();
void  saveAccounts();
void  addAccount();
void  deposit();
void  withdraw();
void  searchAccount();
void  listAccounts();
void  changePin();
void  fundTransfer();
void  miniStatement();
void  statementByDate();
void  interestCalculator();
int   findAccount(int accNo);
int   verifyPIN(int index);
void  logTransaction(int accNo, char type[], float amt, char note[]);
void  printDivider(char ch, int len);

// ─────────────────────────────────────────────
// Main
// ─────────────────────────────────────────────
int main() {

    int choice;
    loadAccounts();

    while (1) {

        printf("\n");
        printDivider('=', 40);
        printf("       BANK MANAGEMENT SYSTEM\n");
        printDivider('=', 40);
        printf("  1.  Add Account\n");
        printf("  2.  Deposit\n");
        printf("  3.  Withdraw\n");
        printf("  4.  Fund Transfer\n");
        printf("  5.  Search Account\n");
        printf("  6.  List Accounts\n");
        printf("  7.  Change PIN\n");
        printf("  8.  Mini Statement (Last %d)\n", MINI_STMT_COUNT);
        printf("  9.  Statement by Date Range\n");
        printf("  10. Interest Calculator\n");
        printf("  11. Exit\n");
        printDivider('=', 40);
        printf("  Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:  addAccount();         break;
            case 2:  deposit();            break;
            case 3:  withdraw();           break;
            case 4:  fundTransfer();       break;
            case 5:  searchAccount();      break;
            case 6:  listAccounts();       break;
            case 7:  changePin();          break;
            case 8:  miniStatement();      break;
            case 9:  statementByDate();    break;
            case 10: interestCalculator(); break;
            case 11:
                saveAccounts();
                printf("\n  Data saved. Goodbye!\n\n");
                exit(0);
            default:
                printf("\n  Invalid choice! Try again.\n");
        }
    }

    return 0;
}

// ─────────────────────────────────────────────
// Utility: Print divider line
// ─────────────────────────────────────────────
void printDivider(char ch, int len) {
    for (int i = 0; i < len; i++) putchar(ch);
    putchar('\n');
}

// ─────────────────────────────────────────────
// Load Accounts from File
// ─────────────────────────────────────────────
void loadAccounts() {

    FILE *fp = fopen("accounts.txt", "r");
    if (fp == NULL) return;

    while (fscanf(fp, "%d %19s %[^\n] %f %d %d",
                  &accounts[count].accNo,
                  accounts[count].accType,
                  accounts[count].name,
                  &accounts[count].balance,
                  &accounts[count].pin,
                  &accounts[count].locked) == 6) {
        count++;
    }

    fclose(fp);
}

// ─────────────────────────────────────────────
// Save Accounts to File
// ─────────────────────────────────────────────
void saveAccounts() {

    FILE *fp = fopen("accounts.txt", "w");
    if (fp == NULL) return;

    for (int i = 0; i < count; i++) {
        fprintf(fp, "%d %s %s %.2f %d %d\n",
                accounts[i].accNo,
                accounts[i].accType,
                accounts[i].name,
                accounts[i].balance,
                accounts[i].pin,
                accounts[i].locked);
    }

    fclose(fp);
}

// ─────────────────────────────────────────────
// Add Account
// ─────────────────────────────────────────────
void addAccount() {

    if (count >= MAX) {
        printf("\n  Account limit reached!\n");
        return;
    }

    printf("\n");
    printDivider('-', 40);
    printf("          ADD ACCOUNT\n");
    printDivider('-', 40);

    printf("  Account Number  : ");
    scanf("%d", &accounts[count].accNo);

    if (findAccount(accounts[count].accNo) != -1) {
        printf("\n  Account already exists!\n");
        return;
    }

    printf("  Account Type\n");
    printf("    1. Savings\n");
    printf("    2. Current\n");
    printf("  Choice: ");
    int typeChoice;
    scanf("%d", &typeChoice);
    if (typeChoice == 1)
        strcpy(accounts[count].accType, "Savings");
    else if (typeChoice == 2)
        strcpy(accounts[count].accType, "Current");
    else {
        printf("\n  Invalid account type!\n");
        return;
    }

    printf("  Holder Name     : ");
    scanf(" %[^\n]", accounts[count].name);

    printf("  Initial Balance : ");
    scanf("%f", &accounts[count].balance);

    if (accounts[count].balance < MIN_BAL) {
        printf("\n  Minimum balance must be %d!\n", MIN_BAL);
        return;
    }

    printf("  Set 4-Digit PIN : ");
    scanf("%d", &accounts[count].pin);

    if (accounts[count].pin < 1000 || accounts[count].pin > 9999) {
        printf("\n  PIN must be exactly 4 digits!\n");
        return;
    }

    accounts[count].locked = 0;
    count++;
    saveAccounts();

    printf("\n  Account created successfully!\n");
}

// ─────────────────────────────────────────────
// Deposit
// ─────────────────────────────────────────────
void deposit() {

    int accNo;
    float amt;

    printf("\n");
    printDivider('-', 40);
    printf("            DEPOSIT\n");
    printDivider('-', 40);

    printf("  Account Number: ");
    scanf("%d", &accNo);

    int idx = findAccount(accNo);
    if (idx == -1) { printf("\n  Account not found!\n"); return; }
    if (!verifyPIN(idx)) return;

    printf("  Deposit Amount: ");
    scanf("%f", &amt);
    if (amt <= 0) { printf("\n  Invalid amount!\n"); return; }

    float old = accounts[idx].balance;
    accounts[idx].balance += amt;

    printf("\n  %-15s : %d\n",   "Account",    accNo);
    printf("  %-15s : %.2f\n",  "Old Balance", old);
    printf("  %-15s : +%.2f\n", "Deposited",   amt);
    printf("  %-15s : %.2f\n",  "New Balance", accounts[idx].balance);

    logTransaction(accNo, "Deposit  ", amt, "Self deposit");
    saveAccounts();
}

// ─────────────────────────────────────────────
// Withdraw
// ─────────────────────────────────────────────
void withdraw() {

    int accNo;
    float amt;

    printf("\n");
    printDivider('-', 40);
    printf("            WITHDRAW\n");
    printDivider('-', 40);

    printf("  Account Number: ");
    scanf("%d", &accNo);

    int idx = findAccount(accNo);
    if (idx == -1) { printf("\n  Account not found!\n"); return; }
    if (!verifyPIN(idx)) return;

    printf("  Withdraw Amount: ");
    scanf("%f", &amt);
    if (amt <= 0) { printf("\n  Invalid amount!\n"); return; }

    if (accounts[idx].balance - amt < MIN_BAL) {
        printf("\n  Insufficient funds! Min balance of %d must remain.\n", MIN_BAL);
        return;
    }

    float old = accounts[idx].balance;
    accounts[idx].balance -= amt;

    printf("\n  %-15s : %d\n",   "Account",    accNo);
    printf("  %-15s : %.2f\n",  "Old Balance", old);
    printf("  %-15s : -%.2f\n", "Withdrawn",   amt);
    printf("  %-15s : %.2f\n",  "New Balance", accounts[idx].balance);

    logTransaction(accNo, "Withdraw ", amt, "Self withdrawal");
    saveAccounts();
}

// ─────────────────────────────────────────────
// Fund Transfer
// ─────────────────────────────────────────────
void fundTransfer() {

    int fromAcc, toAcc;
    float amt;

    printf("\n");
    printDivider('-', 40);
    printf("          FUND TRANSFER\n");
    printDivider('-', 40);

    printf("  Your Account Number      : ");
    scanf("%d", &fromAcc);

    int fromIdx = findAccount(fromAcc);
    if (fromIdx == -1) { printf("\n  Sender account not found!\n"); return; }
    if (!verifyPIN(fromIdx)) return;

    printf("  Recipient Account Number : ");
    scanf("%d", &toAcc);

    if (fromAcc == toAcc) { printf("\n  Cannot transfer to the same account!\n"); return; }

    int toIdx = findAccount(toAcc);
    if (toIdx == -1) { printf("\n  Recipient account not found!\n"); return; }

    printf("  Transfer Amount          : ");
    scanf("%f", &amt);
    if (amt <= 0) { printf("\n  Invalid amount!\n"); return; }

    if (accounts[fromIdx].balance - amt < MIN_BAL) {
        printf("\n  Insufficient funds! Min balance of %d must remain.\n", MIN_BAL);
        return;
    }

    accounts[fromIdx].balance -= amt;
    accounts[toIdx].balance   += amt;

    printf("\n  From     : %d (%s)\n", accounts[fromIdx].accNo, accounts[fromIdx].name);
    printf("  To       : %d (%s)\n",  accounts[toIdx].accNo,   accounts[toIdx].name);
    printf("  Amount   : %.2f\n",     amt);
    printf("  Your Bal : %.2f\n",     accounts[fromIdx].balance);

    char note[100];
    snprintf(note, sizeof(note), "Transferred to Acc#%d", toAcc);
    logTransaction(fromAcc, "Transfer ", amt, note);
    snprintf(note, sizeof(note), "Received from Acc#%d", fromAcc);
    logTransaction(toAcc, "Received ", amt, note);

    saveAccounts();
}

// ─────────────────────────────────────────────
// Search Account
// ─────────────────────────────────────────────
void searchAccount() {

    int accNo;
    printf("\n  Enter Account Number: ");
    scanf("%d", &accNo);

    int idx = findAccount(accNo);
    if (idx == -1) { printf("\n  Account not found!\n"); return; }

    printf("\n");
    printDivider('-', 40);
    printf("          ACCOUNT DETAILS\n");
    printDivider('-', 40);
    printf("  %-15s : %d\n",   "Account No", accounts[idx].accNo);
    printf("  %-15s : %s\n",   "Name",        accounts[idx].name);
    printf("  %-15s : %s\n",   "Type",        accounts[idx].accType);
    printf("  %-15s : %.2f\n", "Balance",     accounts[idx].balance);
    printf("  %-15s : %s\n",   "Status",      accounts[idx].locked ? "LOCKED" : "Active");
    printDivider('-', 40);
}

// ─────────────────────────────────────────────
// List All Accounts
// ─────────────────────────────────────────────
void listAccounts() {

    if (count == 0) { printf("\n  No accounts available!\n"); return; }

    printf("\n");
    printDivider('=', 65);
    printf("  %-8s %-20s %-10s %-12s %s\n",
           "Acc No", "Name", "Type", "Balance", "Status");
    printDivider('-', 65);

    float total = 0;
    for (int i = 0; i < count; i++) {
        printf("  %-8d %-20s %-10s %-12.2f %s\n",
               accounts[i].accNo,
               accounts[i].name,
               accounts[i].accType,
               accounts[i].balance,
               accounts[i].locked ? "LOCKED" : "Active");
        total += accounts[i].balance;
    }

    printDivider('-', 65);
    printf("  Total Accounts : %d\n", count);
    printf("  Total Balance  : %.2f\n", total);
    printDivider('=', 65);
}

// ─────────────────────────────────────────────
// Change PIN
// ─────────────────────────────────────────────
void changePin() {

    int accNo;
    printf("\n");
    printDivider('-', 40);
    printf("           CHANGE PIN\n");
    printDivider('-', 40);

    printf("  Account Number: ");
    scanf("%d", &accNo);

    int idx = findAccount(accNo);
    if (idx == -1) { printf("\n  Account not found!\n"); return; }

    if (accounts[idx].locked) {
        printf("\n  Account is LOCKED. Contact the bank to unlock.\n");
        return;
    }

    if (!verifyPIN(idx)) return;

    int newPin, confirmPin;
    printf("  New 4-Digit PIN : ");
    scanf("%d", &newPin);

    if (newPin < 1000 || newPin > 9999) {
        printf("\n  PIN must be exactly 4 digits!\n");
        return;
    }

    printf("  Confirm New PIN : ");
    scanf("%d", &confirmPin);

    if (newPin != confirmPin) {
        printf("\n  PINs do not match! PIN not changed.\n");
        return;
    }

    if (newPin == accounts[idx].pin) {
        printf("\n  New PIN cannot be the same as the old PIN!\n");
        return;
    }

    accounts[idx].pin = newPin;
    saveAccounts();
    printf("\n  PIN changed successfully!\n");
    logTransaction(accNo, "PIN Chng ", 0.0, "PIN changed by user");
}

// ─────────────────────────────────────────────
// Mini Statement — Last 5 Transactions
// ─────────────────────────────────────────────
void miniStatement() {

    int accNo;
    printf("\n");
    printDivider('-', 40);
    printf("         MINI STATEMENT\n");
    printDivider('-', 40);

    printf("  Account Number: ");
    scanf("%d", &accNo);

    int idx = findAccount(accNo);
    if (idx == -1) { printf("\n  Account not found!\n"); return; }
    if (!verifyPIN(idx)) return;

    FILE *fp = fopen("transactions.txt", "r");
    if (fp == NULL) { printf("\n  No transactions available!\n"); return; }

    char lines[1000][300];
    int  lineCount = 0;
    char buffer[300];
    char prefix[30];
    snprintf(prefix, sizeof(prefix), "Acc#%d", accNo);

    while (fgets(buffer, sizeof(buffer), fp) && lineCount < 1000) {
        if (strstr(buffer, prefix))
            strncpy(lines[lineCount++], buffer, 299);
    }
    fclose(fp);

    if (lineCount == 0) {
        printf("\n  No transactions found for Account %d!\n", accNo);
        return;
    }

    int start = (lineCount > MINI_STMT_COUNT) ? (lineCount - MINI_STMT_COUNT) : 0;

    printf("\n  Account : %d | %s | %s\n",
           accounts[idx].accNo, accounts[idx].name, accounts[idx].accType);
    printf("  Balance : %.2f\n", accounts[idx].balance);
    printf("\n  Last %d Transaction(s):\n", MINI_STMT_COUNT);
    printDivider('-', 65);

    for (int i = start; i < lineCount; i++)
        printf("  %s", lines[i]);

    printDivider('=', 65);
}

// ─────────────────────────────────────────────
// Statement by Date Range
// ─────────────────────────────────────────────
void statementByDate() {

    int accNo;
    char fromDate[15], toDate[15];

    printf("\n");
    printDivider('-', 40);
    printf("      STATEMENT BY DATE RANGE\n");
    printDivider('-', 40);

    printf("  Account Number         : ");
    scanf("%d", &accNo);

    int idx = findAccount(accNo);
    if (idx == -1) { printf("\n  Account not found!\n"); return; }
    if (!verifyPIN(idx)) return;

    printf("  From Date (DD-MM-YYYY) : ");
    scanf("%s", fromDate);
    printf("  To Date   (DD-MM-YYYY) : ");
    scanf("%s", toDate);

    struct tm tmFrom = {0}, tmTo = {0};
    if (sscanf(fromDate, "%d-%d-%d",
               &tmFrom.tm_mday, &tmFrom.tm_mon, &tmFrom.tm_year) != 3 ||
        sscanf(toDate,   "%d-%d-%d",
               &tmTo.tm_mday,   &tmTo.tm_mon,   &tmTo.tm_year) != 3) {
        printf("\n  Invalid date format! Use DD-MM-YYYY.\n");
        return;
    }

    tmFrom.tm_mon -= 1;  tmFrom.tm_year -= 1900;
    tmTo.tm_mon   -= 1;  tmTo.tm_year   -= 1900;
    time_t tFrom = mktime(&tmFrom);
    time_t tTo   = mktime(&tmTo) + 86399;   // include the full end day

    if (tFrom > tTo) {
        printf("\n  'From' date must be before 'To' date!\n");
        return;
    }

    FILE *fp = fopen("transactions.txt", "r");
    if (fp == NULL) { printf("\n  No transactions available!\n"); return; }

    char buffer[300];
    char prefix[30];
    snprintf(prefix, sizeof(prefix), "Acc#%d", accNo);
    int found = 0;

    printf("\n  Account : %d | %s\n", accounts[idx].accNo, accounts[idx].name);
    printf("  Period  : %s  to  %s\n", fromDate, toDate);
    printDivider('-', 65);

    while (fgets(buffer, sizeof(buffer), fp)) {

        if (!strstr(buffer, prefix)) continue;

        // Log line format: [Www Mmm DD HH:MM:SS YYYY] ...
        char *bStart = strchr(buffer, '[');
        char *bEnd   = strchr(buffer, ']');
        if (!bStart || !bEnd || bEnd <= bStart) continue;

        char dateStr[50] = {0};
        strncpy(dateStr, bStart + 1, (size_t)(bEnd - bStart - 1));

        struct tm txTm = {0};
        char monStr[10] = {0};
        if (sscanf(dateStr, "%*s %9s %d %d:%d:%d %d",
                   monStr,
                   &txTm.tm_mday,
                   &txTm.tm_hour,
                   &txTm.tm_min,
                   &txTm.tm_sec,
                   &txTm.tm_year) != 6) continue;

        const char *months[] = {"Jan","Feb","Mar","Apr","May","Jun",
                                 "Jul","Aug","Sep","Oct","Nov","Dec"};
        for (int m = 0; m < 12; m++) {
            if (strcmp(monStr, months[m]) == 0) { txTm.tm_mon = m; break; }
        }
        txTm.tm_year -= 1900;
        time_t txTime = mktime(&txTm);

        if (txTime >= tFrom && txTime <= tTo) {
            printf("  %s", buffer);
            found++;
        }
    }

    fclose(fp);
    printDivider('=', 65);

    if (!found)
        printf("  No transactions found in this date range.\n");
    else
        printf("  Total Transactions Found: %d\n", found);
}

// ─────────────────────────────────────────────
// Interest Calculator
// ─────────────────────────────────────────────
void interestCalculator() {

    printf("\n");
    printDivider('-', 40);
    printf("        INTEREST CALCULATOR\n");
    printDivider('-', 40);

    int choice;
    printf("  Interest Type:\n");
    printf("    1. Simple Interest\n");
    printf("    2. Compound Interest\n");
    printf("  Choice: ");
    scanf("%d", &choice);

    if (choice != 1 && choice != 2) { printf("\n  Invalid choice!\n"); return; }

    float principal, rate, time_yr;

    printf("  Principal Amount (Rs)  : ");
    scanf("%f", &principal);
    if (principal <= 0) { printf("\n  Invalid principal!\n"); return; }

    printf("  Annual Interest Rate %% : ");
    scanf("%f", &rate);
    if (rate <= 0 || rate > 100) { printf("\n  Invalid rate!\n"); return; }

    printf("  Time Period (Years)    : ");
    scanf("%f", &time_yr);
    if (time_yr <= 0) { printf("\n  Invalid time period!\n"); return; }

    printDivider('-', 40);

    if (choice == 1) {

        float interest = (principal * rate * time_yr) / 100.0f;
        float total    = principal + interest;

        printf("  Type         : Simple Interest\n");
        printf("  Principal    : Rs %.2f\n", principal);
        printf("  Rate         : %.2f%% per annum\n", rate);
        printf("  Time         : %.1f year(s)\n", time_yr);
        printDivider('-', 40);
        printf("  Interest     : Rs %.2f\n", interest);
        printf("  Total Amount : Rs %.2f\n", total);

    } else {

        int freqChoice;
        printf("  Compounding Frequency:\n");
        printf("    1. Annually  (n=1)\n");
        printf("    2. Quarterly (n=4)\n");
        printf("    3. Monthly   (n=12)\n");
        printf("    4. Daily     (n=365)\n");
        printf("  Choice: ");
        scanf("%d", &freqChoice);

        int n;
        const char *freqName;
        switch (freqChoice) {
            case 1: n = 1;   freqName = "Annual";    break;
            case 2: n = 4;   freqName = "Quarterly"; break;
            case 3: n = 12;  freqName = "Monthly";   break;
            case 4: n = 365; freqName = "Daily";     break;
            default:
                printf("\n  Invalid frequency!\n");
                return;
        }

        float total    = principal * (float)pow(1.0 + (rate / 100.0) / n, n * time_yr);
        float interest = total - principal;

        printf("  Type         : Compound Interest (%s)\n", freqName);
        printf("  Principal    : Rs %.2f\n", principal);
        printf("  Rate         : %.2f%% per annum\n", rate);
        printf("  Time         : %.1f year(s)\n", time_yr);
        printf("  Compounds    : %d time(s)/year\n", n);
        printDivider('-', 40);
        printf("  Interest     : Rs %.2f\n", interest);
        printf("  Total Amount : Rs %.2f\n", total);

        // Year-by-year breakdown
        printf("\n  Year-by-Year Breakdown:\n");
        printDivider('-', 40);
        printf("  %-6s %-18s %s\n", "Year", "Cumul. Interest", "Balance");
        printDivider('-', 40);
        for (int y = 1; y <= (int)time_yr; y++) {
            float bal  = principal * (float)pow(1.0 + (rate / 100.0) / n, n * y);
            float intr = bal - principal;
            printf("  %-6d Rs %-14.2f Rs %.2f\n", y, intr, bal);
        }
    }

    printDivider('=', 40);
}

// ─────────────────────────────────────────────
// Find Account
// ─────────────────────────────────────────────
int findAccount(int accNo) {
    for (int i = 0; i < count; i++)
        if (accounts[i].accNo == accNo) return i;
    return -1;
}

// ─────────────────────────────────────────────
// Verify PIN  (3-attempt auto-lock)
// ─────────────────────────────────────────────
int verifyPIN(int index) {

    if (accounts[index].locked) {
        printf("\n  Account LOCKED! Contact the bank to unlock.\n");
        return 0;
    }

    for (int attempts = 0; attempts < MAX_PIN_ATTEMPTS; attempts++) {

        int pin;
        printf("  Enter PIN (%d attempt(s) left): ", MAX_PIN_ATTEMPTS - attempts);
        scanf("%d", &pin);

        if (accounts[index].pin == pin) return 1;

        if (attempts < MAX_PIN_ATTEMPTS - 1)
            printf("\n  Wrong PIN! %d attempt(s) remaining.\n\n", MAX_PIN_ATTEMPTS - attempts - 1);
    }

    accounts[index].locked = 1;
    saveAccounts();
    printf("\n  Account LOCKED after %d failed attempts!\n", MAX_PIN_ATTEMPTS);
    printf("  Please contact the bank to unlock.\n");
    return 0;
}

// ─────────────────────────────────────────────
// Log Transaction
// ─────────────────────────────────────────────
void logTransaction(int accNo, char type[], float amt, char note[]) {

    FILE *fp = fopen("transactions.txt", "a");
    if (fp == NULL) return;

    time_t t;
    time(&t);
    char timeStr[40];
    strncpy(timeStr, ctime(&t), sizeof(timeStr) - 1);
    timeStr[strcspn(timeStr, "\n")] = '\0';

    fprintf(fp, "[%s] Acc#%d | %-9s | Amt: %9.2f | %s\n", timeStr, accNo, type, amt, note);

    fclose(fp);
}
