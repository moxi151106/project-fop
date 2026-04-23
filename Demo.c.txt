#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <ctype.h>


void printHeader(void)
{
    printf("\n========================================================\n");
    printf("                 VIHAM BANK SYSTEM\n");
    printf("                 ESTD : 10/02/2026\n");
    printf("========================================================\n");
}

void printLine()
{
    printf("--------------------------------------------------------\n");
}

void printBox(char *title)
{
    printLine();
    printf("                 %s\n", title);
    printLine();
}

void pauseScreen()
{
    printf("\nPress Enter to continue...");
    while(getchar() != '\n');
    getchar(); 
}

void clearScreen()
{
    system("cls");   
}

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

struct AccountInfo *accounts = NULL;
int totalUsers = 0;
int loggedInUser = -1;

char loginAccNo[13];
char loginPwd[9];
char selectedAccType[10];


void userLogin(int);
void createAccount(void);
void mainMenu(void);
void viewAccount(void);
void checkBalance(void);
void withdrawCash(void);
void depositCash(void);
void showDetails(void);
void transferFunds(void);
void viewTransactions(void);
void updatePassword(void);
void requestLoan(void);



int main(void)
{
    int i, choice;
    FILE *dataFile;
    FILE *tempFile;
	

    accounts = (struct AccountInfo *)calloc(98, sizeof(struct AccountInfo));

    if(accounts == NULL)
    {
        printf("Memory Allocation Failed\n");
        return 1;
    }

    dataFile = fopen("Bank_Data.csv","r");

    if(dataFile == NULL)
    {
        printf("No previous data found. Starting fresh.\n");
    }
    else
    {
        char line[500];

        while(fgets(line, sizeof(line), dataFile))
        {
            int fields = sscanf(line,
            "%8[^,],%8[^,],%12[^,],%12[^,],%19[^,],%9[^,],%9[^,],%d,%d,%d,%10[^,],%12[^,],%19[^,],%d,%d,%d,%d,%6[^,],%d,%d,%d,%d,%d",

            accounts[totalUsers].savingPassword,
            accounts[totalUsers].currentPassword,
            accounts[totalUsers].savingAccNo,
            accounts[totalUsers].currentAccNo,
            accounts[totalUsers].firstName,
            accounts[totalUsers].accountType,
            accounts[totalUsers].currentType,
            &accounts[totalUsers].day,
            &accounts[totalUsers].month,
            &accounts[totalUsers].year,
            accounts[totalUsers].phoneNumber,
            accounts[totalUsers].aadhaarNo,
            accounts[totalUsers].lastName,
            &accounts[totalUsers].savingBalance,
            &accounts[totalUsers].currentBalance,
            &accounts[totalUsers].withdrawAmt,
            &accounts[totalUsers].depositAmt,
            accounts[totalUsers].gender,
            &accounts[totalUsers].failedAttempts,
            &accounts[totalUsers].isBlocked,
            &accounts[totalUsers].loanRequested,
            &accounts[totalUsers].loanApproved,
            &accounts[totalUsers].loanAmount
            );

            if(fields >= 20)
            {
                totalUsers++;
            }

            if(totalUsers >= 98)
            {
                printf("Max user limit reached!\n");
                break;
            }
        }

        fclose(dataFile);

        printf("Loaded Users: %d\n", totalUsers);
    }


    while(1)
    {
        clearScreen();
        printHeader();

        printf("\n                 Swagat Hai!\n");
        printLine();

        printf("  1] Login\n");
        printf("  2] Create Account\n");
        printf("  3] Logout\n");

        printLine();
        printf("Enter choice: ");
        scanf("%d", &choice);
		
		
        if(choice == 3)
            break;

        userLogin(choice);
		
    	if(totalUsers == 0)
		{
			printf("WARNING: No users loaded. File NOT overwritten!\n");
		}
		if(1)
		{
			tempFile = fopen("temp.csv", "w");

			if(tempFile == NULL)
			{
				printf("Error creating temp file!\n");
			}
			else
			{
				for(i = 0; i < totalUsers; i++)
				{
					if(strlen(accounts[i].savingAccNo) == 0)
					strcpy(accounts[i].savingAccNo, "0");

					if(strlen(accounts[i].currentAccNo) == 0)
						strcpy(accounts[i].currentAccNo, "0");

					if(strlen(accounts[i].savingPassword) == 0)
						strcpy(accounts[i].savingPassword, "0");

					if(strlen(accounts[i].currentPassword) == 0)
						strcpy(accounts[i].currentPassword, "0");

					if(strlen(accounts[i].accountType) == 0)
						strcpy(accounts[i].accountType, "None");

					if(strlen(accounts[i].currentType) == 0)
						strcpy(accounts[i].currentType, "None");
					
					
					fprintf(tempFile,
					"%s,%s,%s,%s,%s,%s,%s,%d,%d,%d,%s,%s,%s,%d,%d,%d,%d,%s,%d,%d,%d,%d,%d\n",
					accounts[i].savingPassword,
					accounts[i].currentPassword,
					accounts[i].savingAccNo,
					accounts[i].currentAccNo,
					accounts[i].firstName,
					accounts[i].accountType,
					accounts[i].currentType,
					accounts[i].day,
					accounts[i].month,
					accounts[i].year,
					accounts[i].phoneNumber,
					accounts[i].aadhaarNo,
					accounts[i].lastName,
					accounts[i].savingBalance,
					accounts[i].currentBalance,
					accounts[i].withdrawAmt,
					accounts[i].depositAmt,
					accounts[i].gender,
					accounts[i].failedAttempts,
					accounts[i].isBlocked,
					accounts[i].loanRequested,
					accounts[i].loanApproved,
					accounts[i].loanAmount);
				}

				fclose(tempFile);

				remove("Bank_Data.csv");
				rename("temp.csv", "Bank_Data.csv");

				printf("Data saved successfully!\n");
			}
		}
    }

    free(accounts);
    return 0;
}

void userLogin(int choice)
{
    int j; 
	int validLogin = 0;

    if (choice == 2)
    {
        createAccount();
        return;
    }

    clearScreen();
    printHeader();
    printBox("LOGIN");

    printf("Enter Your Account Number : ");
    scanf("%12s", loginAccNo);

    printf("Enter Password            : ");
    scanf("%8s", loginPwd);

    for (j = 0; j < totalUsers; j++)
	{
		
		if (strcmp(accounts[j].savingAccNo, loginAccNo) == 0)
		{
			if(accounts[j].isBlocked == 1)
			{
				printf("Account Blocked! Contact Admin\n");
				pauseScreen();
				return;
			}

			if(strcmp(accounts[j].savingPassword, loginPwd) == 0)
			{
				loggedInUser = j;
				validLogin = 1;
				accounts[j].failedAttempts = 0;
				break;
			}
			else
			{
				accounts[j].failedAttempts++;

				if(accounts[j].failedAttempts >= 3)
				{
					accounts[j].isBlocked = 1;
					printf("Account Blocked!\n");
					pauseScreen();
				}
				else
				{
					printf("Wrong Password\n");	
					printf("Attempts left: %d\n", 3 - accounts[j].failedAttempts);
					pauseScreen();
				}
				return;
			}
		}

		else if (strcmp(accounts[j].currentAccNo, loginAccNo) == 0)
		{
			if(accounts[j].isBlocked == 1)
			{
				printf("Account Blocked! Contact Admin\n");
				pauseScreen();
				return;
			}

			if(strcmp(accounts[j].currentPassword, loginPwd) == 0)
			{
				loggedInUser = j;
				validLogin = 1;
				accounts[j].failedAttempts = 0;
				break;
			}
			else
			{
				accounts[j].failedAttempts++;

				if(accounts[j].failedAttempts >= 3)
				{
					accounts[j].isBlocked = 1;
					printf("Account Blocked!\n");
					pauseScreen();
				}
				else
				{
					printf("Wrong Password\n");
					printf("Attempts left: %d\n", 3 - accounts[j].failedAttempts);
					pauseScreen();
				}
				return;
			}
		}
	}
    if (validLogin == 0)
    {
        printf("Invalid Login\n");
		pauseScreen();
        return;
    }

    mainMenu();
}


void mainMenu(void)
{
    int menuChoice;

    while(1)
    {
        clearScreen();
        printHeader();
        printBox("MAIN MENU");

        printf("  1] MY ACCOUNT\n");
        printf("  2] CREATE NEW ACCOUNT\n");
        printf("  3] LOGOUT\n");

        printLine();
        printf("Select: ");
        scanf("%d", &menuChoice);

        switch(menuChoice)
        {
            case 1: 
				viewAccount(); 
				break;
            case 2: 
				createAccount(); 
				break;
            case 3:
                printf("\nThank You!\n");
                loggedInUser = -1;
                return;
            default:
                printf("Invalid Choice\n");
				pauseScreen();
        }
    }
}

void viewAccount(void)
{
    int option;

    while(1)
    {
        clearScreen();
        printHeader();
        printBox("MY ACCOUNT");

        printf("  1] Balance\n");
        printf("  2] Account Details\n");
        printf("  3] Withdraw Money\n");
        printf("  4] Deposit Money\n");
        printf("  5] Transfer Money\n");
        printf("  6] Transaction History\n");
        printf("  7] Request Loan\n");
		printf("  8] Update Password\n");
        printf("  0] Back\n");

        printLine();
        printf("Select: ");
        scanf("%d", &option);

        switch(option)
        {
            case 1: 
					checkBalance(); 
					break;
            case 2: 
					showDetails(); 
					break;
            case 3: 
					withdrawCash(); 
					break;
            case 4: 
					depositCash(); 
					break;
            case 5: 
					transferFunds();
					break;
            case 6: 
					viewTransactions(); 
					break;
            case 7: 
					requestLoan(); 
					break;
			case 8:
					updatePassword();
					break;
					
            case 0: return;
			
            default: 
				printf("Invalid\n");
				pauseScreen();
        }
    }
}


void showDetails(void)
{
    int selectOption;
    int confirmChange;

    clearScreen();
    printHeader();
    printBox("ACCOUNT DETAILS");

    printf("Name       : %s %s\n",accounts[loggedInUser].firstName, accounts[loggedInUser].lastName);
    printf("Gender     : %s\n", accounts[loggedInUser].gender);
    printf("Mobile     : %s\n", accounts[loggedInUser].phoneNumber);
    printf("DOB        : %d/%d/%d\n", accounts[loggedInUser].day, accounts[loggedInUser].month, accounts[loggedInUser].year);
    printf("Aadhaar    : %s\n", accounts[loggedInUser].aadhaarNo);

    printLine();

    if(strcmp(accounts[loggedInUser].savingAccNo, loginAccNo) == 0)
    {
        printf("Saving Account : %s\n", accounts[loggedInUser].savingAccNo);
        printf("Balance        : Rs. %d\n", accounts[loggedInUser].savingBalance);
    }

    if(strcmp(accounts[loggedInUser].currentAccNo, loginAccNo) == 0)
    {
        printf("Current Account: %s\n", accounts[loggedInUser].currentAccNo);
        printf("Balance        : Rs. %d\n", accounts[loggedInUser].currentBalance);
    }

	
    printLine();

    if(accounts[loggedInUser].loanRequested == 1)
    {
        printf("Loan Requested : YES\n");
        printf("Loan Amount    : %d\n", accounts[loggedInUser].loanAmount);

        if(accounts[loggedInUser].loanApproved == 1)
            printf("Loan Status    : Approved\n");
        else
            printf("Loan Status    : Pending\n");
    }

    printLine();
    pauseScreen();
	return; 
}

void updatePassword(void)
{
    int attemptsLeft = 3;
    int j;
    int inputDay;
	int inputMonth;
	int	inputYear;

    clearScreen();
    printHeader();
    printBox("UPDATE PASSWORD");

    for(j=1; j<=attemptsLeft; j++)
    {
        printf("Enter Your Date Of Birth (YYYY/DD/MM): ");
        scanf("%d/%d/%d",&inputYear ,&inputDay, &inputMonth);

        if(inputDay==accounts[loggedInUser].day &&
           inputMonth==accounts[loggedInUser].month &&
           inputYear==accounts[loggedInUser].year)
        {
            printf("Enter New Password: ");
            
            if(strcmp(loginAccNo, accounts[loggedInUser].savingAccNo) == 0)
                scanf("%8s", accounts[loggedInUser].savingPassword);
            else
                scanf("%8s", accounts[loggedInUser].currentPassword);

            printf("\nPassword has Updated Successfully!!!\n");
			pauseScreen();
            return;
        }
        else
        {
            printf("Wrong Date Of Birth!!, Attempts Remaining - %d\n", attemptsLeft-j);
        }
    }

    printf("Too Many Failed Attempts. Try Later.\n");
	pauseScreen();
	
}



void withdrawCash(void)
{
    int selectedAcc;
    int amount;
    FILE *transFile;
	

    clearScreen();
    printHeader();
    printBox("WITHDRAW MONEY");

	if(accounts[loggedInUser].savingAccNo[0] != '\0' && accounts[loggedInUser].currentAccNo[0] != '\0')
		{
			printf("Select Account: \n1] Saving\n2] Current\n ");  
			printf("Enter Choice:");
			scanf("%d", &selectedAcc);
			printf("Withdrawal Amount: Rs. ");
			scanf("%d", &amount);
			if(amount<=0)
			{
				printf("Invalid\n");
				return;
			}
		}
		else if(accounts[loggedInUser].savingAccNo[0] != '\0')
		{
			selectedAcc = 1;
			printf("Withdrawal Amount: Rs. ");
			scanf("%d", &amount);
			if(amount<=0)
			{
				printf("Invalid\n");
				return;
			}
		}
		else if(accounts[loggedInUser].currentAccNo[0] != '\0')
		{
			selectedAcc = 2;
			printf("Withdrawal Amount: Rs. ");
			scanf("%d", &amount);
			if(amount<=0)
			{
				printf("Invalid\n");
				return;
			}
		}	
		else
		{	printf("No account exists to Withdraw.\n");
			return;
		}
		
		if(selectedAcc == 1) 
		{
			if(amount > accounts[loggedInUser].savingBalance)
			{	printf("Insufficient Saving Balance\n");
				pauseScreen();
			}
			else
			{
				accounts[loggedInUser].savingBalance -= amount;
				printf("Withdrawn Rs. %d from Saving\n", amount);
				pauseScreen();
				transFile = fopen("Transactions.txt", "a");

				if(transFile != NULL)
				{
					fprintf(transFile, "Acc: %s | Withdraw | Amount: %d\n",accounts[loggedInUser].savingAccNo, amount);
					fclose(transFile);
				}
				else
				{
					printf("Transaction file error!\n");
				}
			}
		}
		else if(selectedAcc == 2) 
		{
			if(amount > accounts[loggedInUser].currentBalance)
			{		printf("Insufficient Current Balance\n");
					pauseScreen();
			}
				
			else
			{
				accounts[loggedInUser].currentBalance -= amount;
				printf("Withdrawn Rs. %d from Current\n", amount);
				pauseScreen();
				transFile = fopen("Transactions.txt", "a");

				if(transFile != NULL)
				{
					fprintf(transFile, "Acc: %s | Withdraw | Amount: %d\n",accounts[loggedInUser].currentAccNo, amount);
					fclose(transFile);
				}
				else
				{
					printf("Transaction file error!\n");
				}
			}
		}
		else
			printf("Invalid account selection\n");
		
		
			
	}


void depositCash(void)
{
		int selectedAcc;
		int amount;
		FILE *transFile;
		

		clearScreen();
		printHeader();
		printBox("DEPOSIT MONEY");

		if(accounts[loggedInUser].savingAccNo[0] != '\0' && accounts[loggedInUser].currentAccNo[0] != '\0')
			{
				printf("Select Account: 1] Saving\n2] Current\n ");  
				scanf("%d", &selectedAcc);
				printf("Deposit Amount: Rs. ");
				scanf("%d", &amount);
				if(amount<=0)
				{
					printf("Invalid\n");
					return;
				}
			}
			else if(accounts[loggedInUser].savingAccNo[0] != '\0')
			{
				selectedAcc = 1;
				printf("Deposit Amount: Rs. ");
				scanf("%d", &amount); 
				if(amount<=0)
				{
					printf("Invalid\n");
					return;
				}
			}
			else if(accounts[loggedInUser].currentAccNo[0] != '\0')
			{
				selectedAcc = 2;
				printf("Deposit Amount: Rs. ");
				scanf("%d", &amount); 
				if(amount<=0)
				{
					printf("Invalid\n");
					return;
				}
			}	
			else
			{	printf("No account exists to deposit.\n");
				return;
			}
			
			if(selectedAcc == 1)
			{	accounts[loggedInUser].savingBalance += amount;
				printf("Deposited Rs. %d to Saving\n", amount);
				pauseScreen();
				transFile = fopen("Transactions.txt", "a");

				if(transFile != NULL)
				{
					fprintf(transFile, "Acc: %s | Deposit | Amount: %d\n",accounts[loggedInUser].savingAccNo, amount);
					fclose(transFile);
				}
				else
				{
					printf("Transaction file error!\n");
				}
			}
			else if(selectedAcc==2)
			{
				accounts[loggedInUser].currentBalance += amount;
				printf("Deposited Rs. %d to Current\n", amount);
				pauseScreen();
				transFile = fopen("Transactions.txt", "a");
				
				if(transFile != NULL)
				{
					fprintf(transFile, "Acc: %s | Deposit | Amount: %d\n",accounts[loggedInUser].currentAccNo, amount);
					fclose(transFile);
				}
				else
				{
					printf("Transaction file error!\n");
				}
					
			}
			else 
				printf("Invalid account selection\n");
			
			

}

void viewTransactions(void)
{
    FILE *transFile;
    char line[200];
	int hasTransactions = 0;
	char searchKey[30];


    clearScreen();
    printHeader();
    printBox("TRANSACTION HISTORY");

    transFile = fopen("Transactions.txt", "r");

    if(transFile == NULL)
    {
        printf("No Transaction History Found.\n");
        return;
    }
	
	printf("\n--- YOUR TRANSACTION HISTORY ---\n\n");
	sprintf(searchKey, "Acc: %s", loginAccNo);

    while(fgets(line, sizeof(line), transFile) != NULL)
    {
		
        if(strstr(line, searchKey) != NULL)
        {
            printf("%s", line);
            hasTransactions = 1;
        }
    }

    if(hasTransactions == 0)
        printf("No Transactions Yet.\n");

    fclose(transFile);

    pauseScreen();
}

void requestLoan(void)
{
    clearScreen();
    printHeader();
    printBox("LOAN REQUEST");

    if(accounts[loggedInUser].loanRequested == 1)
    {
        printf("Loan already requested\n");
		pauseScreen();
        return;
    }

    printf("Enter Loan Amount: ");
    scanf("%d", &accounts[loggedInUser].loanAmount);

    if(accounts[loggedInUser].loanAmount <= 0)
    {
        printf("Invalid Amount\n");
		pauseScreen();
        return;
    }

    accounts[loggedInUser].loanRequested = 1;
    accounts[loggedInUser].loanApproved = 0;

    printf("Loan request sent\n");
	pauseScreen();
}


void createAccount(void)
{
    int userIndex;
    int validationFlag;
	char dob[20];
	int day, month, year, pos;
	int i;
	char c;


    if (loggedInUser == -1)
        userIndex = totalUsers;
    else
        userIndex = loggedInUser;

    clearScreen();
    printHeader();
    printBox("CREATE ACCOUNT");


	 if (loggedInUser == -1)
    {
		printf("Enter Your Details:\n");
        printf("-> First Name: ");
        scanf("%19s", accounts[userIndex].firstName);

        printf("-> Last Name: ");
        scanf("%19s", accounts[userIndex].lastName);
		while(1)
		{
			validationFlag = 0;
			
			printf("-> Date of Birth (DD/MM/YYYY): ");
			scanf("%19s", dob);
			if (sscanf(dob, "%2d/%2d/%4d%n", &day, &month, &year, &pos) != 3 || dob[pos] != '\0') 
			{
				printf("Invalid format\n");
				validationFlag = 1;
			}
			else if (day <= 0 || day > 31 ||
				   month <= 0 || month > 12 ||
				   year >= 2009 ||year<=0 ) 
			{
				printf("Invalid DOB\n");
				validationFlag = 1;
			}

			if (validationFlag == 1) {
				printf("Enter DOB Again\n");
				continue;
			}

			accounts[userIndex].day = day;
			accounts[userIndex].month = month;
			accounts[userIndex].year = year;
			break;

		
		}
		
		while(1)
		{
			 validationFlag = 0;

			printf("-> Mobile No.(+91): ");
			scanf("%s", accounts[userIndex].phoneNumber);

			if (strlen(accounts[userIndex].phoneNumber) != 10 || accounts[userIndex].phoneNumber[0] == '-')
			{
				printf("Invalid phone\n");
				validationFlag = 1;
			}
			else 			
			{
				
				for (i = 0; i < 10; i++) 
				{
					if (!isdigit(accounts[userIndex].phoneNumber[i]))
					{
						printf("Invalid phone\n");
						validationFlag = 1;
						break;
					}
				}
			}

			if (validationFlag == 1) 
			{
				printf("Enter Mobile Number Again\n");
				continue;
			}

			break;

		}
		
		while(1)
		{
			validationFlag = 0;
			
			printf("-> Aadhaar Number: ");
			scanf("%s", accounts[userIndex].aadhaarNo);
			if(strlen(accounts[userIndex].aadhaarNo)!= 12 || accounts[userIndex].aadhaarNo[0] == '-')
			{
				printf("Invalid\n");
				validationFlag=1;
				
			}
			else 			
			{
				
				for (i = 0; i < 12; i++) 
				{
					if (!isdigit(accounts[userIndex].aadhaarNo[i]))
					{
						printf("Invalid AadhaarNuber\n");
						validationFlag = 1;
						break;
					}
				}
			}
			
			if(validationFlag==1)
			{
				printf("Enter Aadhar Number Again\n");
				continue;
			}
			
			break;
		}
		
		while(1)
		{
			validationFlag = 0;
			
			printf("-> Gender(Male/Female/Other): ");
			scanf("%s", accounts[userIndex].gender);
			if(strcmp(accounts[userIndex].gender, "Male") != 0 && strcmp(accounts[userIndex].gender, "Female") != 0 && strcmp(accounts[userIndex].gender, "Other") != 0 &&
				strcmp(accounts[userIndex].gender, "male") != 0 && strcmp(accounts[userIndex].gender, "female") != 0 && strcmp(accounts[userIndex].gender, "other") != 0)
			{
				printf("Invalid\n");
				validationFlag=1;
					
			}
				
			if(validationFlag==1)
			{
				printf("Enter Gender Again\n");
				continue;
			}
			
			break;
			
		}	
		
		
    }
	
	while(1)
		
	{
		validationFlag = 0;
		
		printf("-> Account Type (Saving/Current): ");
		scanf("%9s", selectedAccType);
		while(getchar() != '\n');
		
		if(strcmp(selectedAccType, "Saving") != 0 &&
       strcmp(selectedAccType, "saving") != 0 &&
       strcmp(selectedAccType, "Current") != 0 &&
       strcmp(selectedAccType, "current") != 0)
		{		printf("Invalid\n");
				validationFlag=1;
		}
		
		if(validationFlag==1)
		{
			printf("Enter AccountType Again\n");
			continue;
		}
			
		break;
		
		
	}
	

    if (strcmp(selectedAccType, "Saving") == 0 || strcmp(selectedAccType, "saving") == 0)
    {
        if (accounts[userIndex].savingAccNo[0] != '\0')
        {
            printf("Saving account already exists\n");
            return;
        }
		
		
        sprintf(accounts[userIndex].savingAccNo, "126225%02d%02d%02d", accounts[userIndex].day, accounts[userIndex].year % 100, userIndex + 1);
		
		while(1)
		{
			validationFlag=0;
			
			printf("-> Opening Balance: Rs. ");
			if (scanf("%d", &accounts[userIndex].savingBalance) != 1) 
			{
				printf("Invalid Amount\n");
				validationFlag = 1;
				while(getchar() != '\n');
			}
			else
			{
				c = getchar();
				if (c != '\n') 
				{
					validationFlag = 1;
					while (c != '\n' && c != EOF) c = getchar();
				}
			}
			if(accounts[userIndex].savingBalance<0)
			{
				printf("Invalid Negative Amount\n");
				validationFlag = 1;
			}
			
			if(validationFlag==1)
			{
				printf("Enter Amount Again\n");
				continue;
			}
			
			break;
		}

        
		
		printf("\n");

        strcpy(accounts[userIndex].accountType, "Saving");
        printf("Your Saving Account Number is %s\n", accounts[userIndex].savingAccNo);
		
		
		printf("\n");
		
		printf("-> Create Password: ");
        scanf("%8s", accounts[userIndex].savingPassword);
		pauseScreen();
    }
    else if (strcmp(selectedAccType, "Current") == 0 || strcmp(selectedAccType, "current") == 0)
    {
        if (accounts[userIndex].currentAccNo[0] != '\0')
        {
            printf("Current account already exists\n");
            return;
        }

        sprintf(accounts[userIndex].currentAccNo, "126225%02d%02d%02d",accounts[userIndex].year % 100, accounts[userIndex].day, userIndex + 1);

		while(1)
		{
			validationFlag=0;
			
			printf("-> Opening Balance: Rs. ");
			
			if(scanf("%d", &accounts[userIndex].currentBalance) != 1)
			{
				printf("Invalid Amount\n");
				validationFlag = 1;
				while(getchar() != '\n');
			}
			else
			{
				c = getchar();
				if (c != '\n') 
				{
					validationFlag = 1;
					while (c != '\n' && c != EOF) c = getchar();
				}
			}
				
			if(accounts[userIndex].currentBalance<0)
			{
				printf("Invalid Amount\n");
				validationFlag = 1;
			}
			
			if(validationFlag==1)
			{
				printf("Enter Amount Again\n");
				continue;
			}
			
			break;
		}
		
		printf("\n");

        strcpy(accounts[userIndex].currentType, "Current");
        printf("Your Current Account Number is %s\n", accounts[userIndex].currentAccNo);
		
		printf("\n");
		
		printf("-> Create Password: ");
        scanf("%8s", accounts[userIndex].currentPassword);
		
		pauseScreen();
		while(getchar() != '\n');
		
    }
	else
	{
		printf("Invalid\n");
		return;
	}

    if (loggedInUser == -1)
	{
		totalUsers++;   
		accounts[userIndex].failedAttempts = 0;
		accounts[userIndex].isBlocked = 0;
		accounts[userIndex].loanRequested = 0;
		accounts[userIndex].loanApproved = 0;
		accounts[userIndex].loanAmount = 0;
	}
}


void transferFunds(void)
{
    int j;
    int confirmTransfer;
    int transferAmount;
    char targetAccNo[13];
    int fromAccType;
	FILE *transFile;

    clearScreen();
    printHeader();
    printBox("TRANSFER FUNDS");

    printf("Enter Target Account Number: ");
    scanf("%12s", targetAccNo);

	for (j = 0; j < totalUsers; j++)
    {
		
        if (strcmp(accounts[j].savingAccNo, targetAccNo) == 0 || strcmp(accounts[j].currentAccNo, targetAccNo) == 0)
        {
			if(strcmp(loginAccNo, targetAccNo) == 0)
			{
				printf("You Cannot Transfer To Your Own Account\n");
				return;
			}
            printf("\nIs This Account Correct:\n");
            printf("Name: %s %s\n", accounts[j].firstName, accounts[j].lastName);
            printf("Gender: %s\n", accounts[j].gender);
            printf("Mobile: %s\n\n", accounts[j].phoneNumber);
			
			
			printf("Confirm Transfer?\n1] Yes \n2] No \n");
			printf("I Select:\n");
			scanf("%d", &confirmTransfer);

			if(confirmTransfer != 1)
			{
				printf("Transaction Cancelled\n");
				pauseScreen();
				return;
			}
			
			if(strcmp(accounts[j].savingAccNo, targetAccNo) == 0 )
			{
				printf("Saving Account: %s\n", accounts[j].savingAccNo);
			}

			if(strcmp(accounts[j].currentAccNo, targetAccNo) == 0)
			{
				printf("Current Account: %s\n", accounts[j].currentAccNo);
			}
			
			
			if(accounts[loggedInUser].savingAccNo[0] != '\0' && accounts[loggedInUser].currentAccNo[0] != '\0')
			{
				printf("\nTransfer From:\n1] Saving\n2] Current\nSelect: ");
				scanf("%d", &fromAccType);
			}
			else if(accounts[loggedInUser].savingAccNo[0] != '\0')
			{
				fromAccType = 1;
			}
			else 
				fromAccType = 2;
			
			printf("Enter The Amount You Want To Send:");
			scanf("%d", &transferAmount);
			
			if (transferAmount <= 0)
            {
                printf("Invalid Amount\n");
				pauseScreen();
                return;
            }
			
			if(fromAccType == 1)
			{
				if(transferAmount > accounts[loggedInUser].savingBalance)
				{
					printf("Insufficient Saving Balance\n");
					pauseScreen();
					return;
				}
				else 
				{
						accounts[loggedInUser].savingBalance -= transferAmount;
						if(strcmp(accounts[j].savingAccNo, targetAccNo) == 0)
							accounts[j].savingBalance += transferAmount;
						else
							accounts[j].currentBalance += transferAmount;
						transFile = fopen("Transactions.txt", "a");
						if(transFile != NULL)
						{
							fprintf(transFile,"From: %s -> To: %s | Transfer | Amount: %d\n",loginAccNo, targetAccNo, transferAmount);
							fclose(transFile);
						}
						else
						{
							printf("Transaction file error!\n");
						}
						printf("Transaction Successful!!\n");
						pauseScreen();
						return;
					
				}
			}
			else if(fromAccType == 2)
			{
				if(transferAmount > accounts[loggedInUser].currentBalance)
				{
					printf("Insufficient Current Balance\n");
					pauseScreen();
					return;
				}
				else 
				{
						accounts[loggedInUser].currentBalance -= transferAmount;
						if(strcmp(accounts[j].savingAccNo, targetAccNo) == 0)
							accounts[j].savingBalance += transferAmount;
						else
							accounts[j].currentBalance += transferAmount;
						transFile = fopen("Transactions.txt", "a");
						if(transFile != NULL)
						{
							fprintf(transFile,"From: %s -> To: %s | Transfer | Amount: %d\n",loginAccNo, targetAccNo, transferAmount);
							fclose(transFile);
						}
						else
						{
							printf("Transaction file error!\n");
						}
						printf("Transaction Successful!!\n");
						pauseScreen();
						return;
				}
			}
			else
			{
				printf("Invalid Choice\n");
				pauseScreen();
				return;
			}
            
        }
    }

    printf("Account Not Found\n");
	pauseScreen();
	
}
void checkBalance(void)
{
    int actionChoice;

    while(1)
    {
        clearScreen();
        printHeader();
        printBox("CHECK BALANCE");

        if(accounts[loggedInUser].savingAccNo[0] != '\0')
            printf("Saving Balance  : Rs. %d\n", accounts[loggedInUser].savingBalance);

        if(accounts[loggedInUser].currentAccNo[0] != '\0')
            printf("Current Balance : Rs. %d\n", accounts[loggedInUser].currentBalance);

        printLine();

        printf("Do You want To:\n");
        printf("  1] Withdraw Money\n");
        printf("  2] Deposit Money\n");
        printf("  0] Back\n");

        printLine();
        printf("Select: ");
        scanf("%d", &actionChoice);

        switch(actionChoice)
        {
            case 1: withdrawCash(); break;
            case 2: depositCash(); break;
            case 0: return;
            default: 
				printf("Invalid\n");
				pauseScreen();
        }
    }
}
