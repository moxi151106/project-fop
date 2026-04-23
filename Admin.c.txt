#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX 100

struct AccountInfo
{
    char savingPassword[9];
    char currentPassword[9];
    char savingAccNo[13];
    char currentAccNo[13];
    char firstName[20];
    char accountType[10];
    char currentType[10];
    int  day, month, year;
    char phoneNumber[11];
    char aadhaarNo[13];
    char lastName[20];
    int  savingBalance;
    int  currentBalance;
    int  withdrawAmt;
    int  depositAmt;
    char gender[7];
    int failedAttempts;
    int isBlocked;
    int loanRequested;
    int loanApproved;
    int loanAmount;
};

void loadData(void);
void saveData(void);
void viewUsers(void);
void unblockUser(void);
void approveLoan(void);


struct AccountInfo users[MAX];
int totalUsers = 0;


int main(void)
{
    char u[10], p[10];
    int ch;

    printf("\n=== ADMIN LOGIN ===\n");

    printf("Username: ");
    scanf("%s", u);

    printf("Password: ");
    scanf("%s", p);

    if(strcmp(u,"admin") != 0 || strcmp(p,"1234") != 0)
    {
        printf("Wrong Credentials\n");
        return 0;
    }

    printf("Login Successful\n");

    loadData();

    while(1)
    {
        printf("\n=== ADMIN PANEL ===\n");
        printf("1 View Users\n");
        printf("2 Unblock User\n");
        printf("3 Approve Loans\n");
        printf("4 Exit\n");

        printf("Enter Choice: ");
        scanf("%d", &ch);

        if(ch == 1)
			viewUsers();
        else if(ch == 2)
			unblockUser();
        else if(ch == 3)
			approveLoan();
        else if(ch == 4)
			break;
        else 
			printf("Invalid Choice\n");
    }

    return 0;
}

void loadData(void)
{
    FILE *fp = fopen("Bank_Data.csv", "r");

    if (!fp)
    {
        printf("File not found!\n");
        return;
    }

    while (fscanf(fp,
    "%8[^,],%8[^,],%12[^,],%12[^,],%19[^,],%9[^,],%9[^,],%d,%d,%d,%10[^,],%12[^,],%19[^,],%d,%d,%d,%d,%6[^,],%d,%d,%d,%d,%d\n",

    users[totalUsers].savingPassword,
    users[totalUsers].currentPassword,
    users[totalUsers].savingAccNo,
    users[totalUsers].currentAccNo,
    users[totalUsers].firstName,
    users[totalUsers].accountType,
    users[totalUsers].currentType,
    &users[totalUsers].day,
    &users[totalUsers].month,
    &users[totalUsers].year,
    users[totalUsers].phoneNumber,
    users[totalUsers].aadhaarNo,
    users[totalUsers].lastName,
    &users[totalUsers].savingBalance,
    &users[totalUsers].currentBalance,
    &users[totalUsers].withdrawAmt,
    &users[totalUsers].depositAmt,
    users[totalUsers].gender,
    &users[totalUsers].failedAttempts,
    &users[totalUsers].isBlocked,
    &users[totalUsers].loanRequested,
    &users[totalUsers].loanApproved,
    &users[totalUsers].loanAmount) != EOF)
    {
        totalUsers++;
    }

    fclose(fp);
}

void saveData()
{
    FILE *fp = fopen("Bank_Data.csv", "w");

    int i;
    for(i = 0; i < totalUsers; i++)
    {
        fprintf(fp,
        "%s,%s,%s,%s,%s,%s,%s,%d,%d,%d,%s,%s,%s,%d,%d,%d,%d,%s,%d,%d,%d,%d,%d\n",

        users[i].savingPassword,
        users[i].currentPassword,
        users[i].savingAccNo,
        users[i].currentAccNo,
        users[i].firstName,
        users[i].accountType,
        users[i].currentType,
        users[i].day,
        users[i].month,
        users[i].year,
        users[i].phoneNumber,
        users[i].aadhaarNo,
        users[i].lastName,
        users[i].savingBalance,
        users[i].currentBalance,
        users[i].withdrawAmt,
        users[i].depositAmt,
        users[i].gender,
        users[i].failedAttempts,
        users[i].isBlocked,
        users[i].loanRequested,
        users[i].loanApproved,
        users[i].loanAmount);
    }

    fclose(fp);
}

void viewUsers()
{
    int i, option;
    char acc[20];
    int found = 0;

    printf("\n--- VIEW USERS ---\n");
    printf("1. View All Users\n");
    printf("2. Search by Account Number\n");
    printf("Enter choice: ");
    scanf("%d", &option);

    switch(option)
    {
        case 1:
            for(i = 0; i < totalUsers; i++)
            {
                printf("Name: %s %s | Saving: %s | Current: %s | Blocked: %d\n",
                    users[i].firstName,
                    users[i].lastName,
                    users[i].savingAccNo,
                    users[i].currentAccNo,
                    users[i].isBlocked);
            }
            break;

        case 2:
            printf("Enter Account No: ");
            scanf("%s", acc);

            for(i = 0; i < totalUsers; i++)
            {
                if(strcmp(users[i].savingAccNo, acc) == 0 ||
                   strcmp(users[i].currentAccNo, acc) == 0)
                {
                    printf("Name: %s %s | Saving: %s | Current: %s | Blocked: %d\n",
                        users[i].firstName,
                        users[i].lastName,
                        users[i].savingAccNo,
                        users[i].currentAccNo,
                        users[i].isBlocked);
                    found = 1;
                    break;
                }
            }

            if(!found)
                printf("Account not found.\n");
            break;

        default:
            printf("Invalid choice.\n");
    }
}

void unblockUser()
{
    char acc[20];
    int i, found = 0;

    printf("\n--- BLOCKED USERS ---\n");
    for(i = 0; i < totalUsers; i++)
    {
        if(users[i].isBlocked == 1)
        {
            printf("Name: %s %s | Saving: %s | Current: %s | Blocked: %d\n",
                users[i].firstName,
                users[i].lastName,
                users[i].savingAccNo,
                users[i].currentAccNo,
                users[i].isBlocked);
            found = 1;
        }
    }

    if(!found)
    {
        printf("No blocked accounts.\n");
        return;
    }

    printf("\nEnter Account No to unblock: ");
    scanf("%s", acc);

    for(i = 0; i < totalUsers; i++)
    {
        if(strcmp(users[i].savingAccNo, acc) == 0 ||
           strcmp(users[i].currentAccNo, acc) == 0)
        {
            if(users[i].isBlocked == 1)
            {
                users[i].isBlocked = 0;
                users[i].failedAttempts = 0;

                printf("User Unblocked: %s %s\n",
                    users[i].firstName,
                    users[i].lastName);

                saveData();
            }
            else
            {
                printf("This account is not blocked.\n");
            }
            return;
        }
    }

    printf("Account not found.\n");
}


void approveLoan()
{
    int i;
    char acc[20];
    int found = 0;
	FILE *transFile = NULL;

    printf("\n--- PENDING LOAN REQUESTS ---\n");
    for(i = 0; i < totalUsers; i++)
    {
        if(users[i].loanRequested == 1 && users[i].loanApproved == 0)
        {
            printf("Name: %s %s | Saving: %s | Current: %s | Loan Amount: %d\n",
                users[i].firstName,
                users[i].lastName,
                users[i].savingAccNo,
                users[i].currentAccNo,
                users[i].loanAmount);
            found = 1;
        }
    }

    if(!found)
    {
        printf("No pending loan requests.\n");
        return;
    }

    printf("\nEnter Account No to approve loan: ");
    scanf("%s", acc);

    for(i = 0; i < totalUsers; i++)
    {
        if((strcmp(users[i].savingAccNo, acc) == 0 ||
            strcmp(users[i].currentAccNo, acc) == 0) &&
            users[i].loanRequested == 1 && users[i].loanApproved == 0)
        {
            users[i].loanApproved = 1;
			users[i].loanRequested = 0;

            if(strcmp(users[i].savingAccNo, acc) == 0)
                users[i].savingBalance += users[i].loanAmount;
            else
                users[i].currentBalance += users[i].loanAmount;

            printf("Loan Approved for %s %s | Amount: %d\n",
                users[i].firstName,
                users[i].lastName,
                users[i].loanAmount);

            // Append transaction entry for user software
            transFile = fopen("Transactions.txt", "a");
            if(transFile)
            {
                fprintf(transFile,
                    "Acc: %s | Amount Credited: %d | By VIHAM BANK\n",
                    acc,
                    users[i].loanAmount);
                fclose(transFile);
            }

            saveData();
            return;
        }
    }

    printf("Account not found or loan already approved.\n");
}

