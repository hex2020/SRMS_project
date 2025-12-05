#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Changed filenames to make it look different
#define RECORD_FILE "records.txt"
#define AUTH_FILE "users.txt" 

struct Candidate {
    int id;
    char fullName[50];
    float score;
};

// Renamed global variables
char sessionUser[20];
char sessionRole[10];

// Renamed functions
int authenticateUser();
void dashboard();
void adminPanel();
void userPanel();
void addNewRecord();
void viewAllRecords();
void findRecord();
void modifyRecord();
void removeRecord();
void showAnalytics();

// Helpers
float getBatchAvg();
char getGrade(float score, float avg);

int main() {
    int attempts = 0;
    
    // Logic for 3 login attempts
    while (attempts < 3) {
        if (authenticateUser()) {
            dashboard();
            return 0; 
        }
        
        attempts++;
        if (attempts < 3) {
            printf("\n>> Wrong credentials! You have %d attempts remaining.\n", 3 - attempts);
        }
    }

    printf("\n=============================================");
    printf("\n ERROR: Maximum login attempts reached.");
    printf("\n System Locked. Exiting...");
    printf("\n=============================================\n");
    return 0;
}

float getBatchAvg() {
    FILE *fp = fopen(RECORD_FILE, "r");
    struct Candidate c;
    float total = 0;
    int n = 0;
    
    if (!fp) return 0.0;

    while (fscanf(fp, "%d %s %f", &c.id, c.fullName, &c.score) == 3) {
        total += c.score;
        n++;
    }
    fclose(fp);
    return (n == 0) ? 0.0 : total / n;
}

char getGrade(float score, float avg) {
    if (score >= avg + 10) return 'O';
    else if (score >= avg) return 'A';
    else if (score >= avg - 10) return 'B';
    else return 'C';
}

int authenticateUser() {
    char uInput[20], pInput[20];
    char fUser[20], fPass[20], fRole[10];

    printf("\n======= SYSTEM LOGIN =======\n");
    printf(" User ID : ");
    scanf("%s", uInput);
    printf(" Password: ");
    scanf("%s", pInput);

    FILE *fp = fopen(AUTH_FILE, "r");
    if (!fp) {
        fp = fopen(AUTH_FILE, "w");
        fprintf(fp, "admin admin123 ADMIN\n");
        fprintf(fp, "staff staff123 STAFF\n");
        fprintf(fp, "guest guest123 GUEST\n");
        fprintf(fp, "student student123 USER\n");
        fclose(fp);
        
        printf("\n[INIT] User database created.\n");
        printf("Restart the app and login with: admin / admin123\n");
        exit(0); // Exit immediately after setup
    }

    while (fscanf(fp, "%s %s %s", fUser, fPass, fRole) == 3) {
        if (strcmp(uInput, fUser) == 0 && strcmp(pInput, fPass) == 0) {
            strcpy(sessionRole, fRole);
            strcpy(sessionUser, fUser);
            fclose(fp);
            return 1; // Success
        }
    }

    fclose(fp);
    return 0; // Fail
}

void dashboard() {
    if (strcmp(sessionRole, "ADMIN") == 0)
        adminPanel();
    else
        userPanel();
}

void adminPanel() {
    int opt;
    do {
        printf("\n\n--- ADMIN DASHBOARD ---\n");
        printf("1. Register New Candidate\n");
        printf("2. View All Records\n");
        printf("3. Search Candidate\n");
        printf("4. Modify Data\n");
        printf("5. Remove Data\n");
        printf("6. Batch Analytics\n");
        printf("7. Sign Out\n");
        printf("Select: ");
        scanf("%d", &opt);

        switch (opt) {
            case 1: addNewRecord(); break;
            case 2: viewAllRecords(); break;
            case 3: findRecord(); break;
            case 4: modifyRecord(); break;
            case 5: removeRecord(); break;
            case 6: showAnalytics(); break;
            case 7: return;
            default: printf("Invalid option.\n");
        }
    } while (1);
}

void userPanel() {
    int opt;
    do {
        printf("\n\n--- USER DASHBOARD ---\n");
        printf("1. View Records\n");
        printf("2. Search Candidate\n");
        printf("3. View Analytics\n");
        printf("4. Sign Out\n");
        printf("Select: ");
        scanf("%d", &opt);

        switch (opt) {
            case 1: viewAllRecords(); break;
            case 2: findRecord(); break;
            case 3: showAnalytics(); break;
            case 4: return;
            default: printf("Invalid option.\n");
        }
    } while(opt != 4);
}

void addNewRecord() {
    FILE *fp = fopen(RECORD_FILE, "a");
    if (!fp) { printf("File error.\n"); return; }

    struct Candidate c;
    printf("Enter ID No: ");
    scanf("%d", &c.id);
    printf("Enter Name: ");
    scanf("%s", c.fullName);
    printf("Enter Score: ");
    scanf("%f", &c.score);

    fprintf(fp, "%d %s %.2f\n", c.id, c.fullName, c.score);
    fclose(fp);
    printf(">> Data Saved.\n");
}

void viewAllRecords() {
    FILE *fp = fopen(RECORD_FILE, "r");
    if (!fp) { printf("No records found.\n"); return; }

    struct Candidate c;
    float avg = getBatchAvg();

    printf("\n%-8s %-15s %-8s %-8s %-5s\n", "ID", "NAME", "SCORE", "PERC %", "GRADE");
    printf("--------------------------------------------------\n");

    while (fscanf(fp, "%d %s %f", &c.id, c.fullName, &c.score) == 3) {
        printf("%-8d %-15s %-8.2f %-8.2f %-5c\n", 
               c.id, c.fullName, c.score, c.score, getGrade(c.score, avg));
    }
    printf("--------------------------------------------------\n");
    printf("* Grade calculated on Batch Avg: %.2f\n", avg);
    fclose(fp);
}

void findRecord() {
    int searchId, found = 0;
    struct Candidate c;
    float avg = getBatchAvg();

    printf("Enter ID to search: ");
    scanf("%d", &searchId);

    FILE *fp = fopen(RECORD_FILE, "r");
    if (!fp) { printf("No database.\n"); return; }

    while (fscanf(fp, "%d %s %f", &c.id, c.fullName, &c.score) == 3) {
        if (c.id == searchId) {
            printf("\n--- DETAILS FOUND ---\n");
            printf("ID       : %d\n", c.id);
            printf("Name     : %s\n", c.fullName);
            printf("Score    : %.2f\n", c.score);
            printf("Rel Grade: %c\n", getGrade(c.score, avg));
            printf("---------------------\n");
            found = 1;
            break;
        }
    }
    if (!found) printf("ID %d not found.\n", searchId);
    fclose(fp);
}

void showAnalytics() {
    FILE *fp = fopen(RECORD_FILE, "r");
    if (!fp) { printf("No data.\n"); return; }

    struct Candidate c;
    float sum = 0, max = -1, min = 101;
    int n = 0, passed = 0;

    while (fscanf(fp, "%d %s %f", &c.id, c.fullName, &c.score) == 3) {
        sum += c.score;
        n++;
        if (c.score > max) max = c.score;
        if (c.score < min) min = c.score;
        if (c.score >= 40) passed++;
    }

    if (n > 0) {
        printf("\n--- BATCH ANALYTICS ---\n");
        printf("Total Candidates : %d\n", n);
        printf("Batch Average    : %.2f\n", sum/n);
        printf("Highest Score    : %.2f\n", max);
        printf("Lowest Score     : %.2f\n", min);
        printf("Pass Percentage  : %.2f%%\n", ((float)passed/n)*100);
        printf("-----------------------\n");
    } else {
        printf("Empty database.\n");
    }
    fclose(fp);
}

void modifyRecord() {
    int targetId, found = 0;
    struct Candidate c;

    printf("Enter ID to update: ");
    scanf("%d", &targetId);

    FILE *fp = fopen(RECORD_FILE, "r");
    FILE *temp = fopen("temp_data.txt", "w");

    if (!fp) return;

    while (fscanf(fp, "%d %s %f", &c.id, c.fullName, &c.score) == 3) {
        if (c.id == targetId) {
            found = 1;
            printf("Updating %s... Enter new Name: ", c.fullName);
            scanf("%s", c.fullName);
            printf("Enter new Score: ");
            scanf("%f", &c.score);
        }
        fprintf(temp, "%d %s %.2f\n", c.id, c.fullName, c.score);
    }
    fclose(fp);
    fclose(temp);
    remove(RECORD_FILE);
    rename("temp_data.txt", RECORD_FILE);

    if (found) printf("Update Success.\n");
    else printf("ID not found.\n");
}

void removeRecord() {
    int targetId, found = 0;
    struct Candidate c;

    printf("Enter ID to delete: ");
    scanf("%d", &targetId);

    FILE *fp = fopen(RECORD_FILE, "r");
    FILE *temp = fopen("temp_data.txt", "w");

    if (!fp) return;

    while (fscanf(fp, "%d %s %f", &c.id, c.fullName, &c.score) == 3) {
        if (c.id != targetId) {
            fprintf(temp, "%d %s %.2f\n", c.id, c.fullName, c.score);
        } else {
            found = 1;
        }
    }
    fclose(fp);
    fclose(temp);
    remove(RECORD_FILE);
    rename("temp_data.txt", RECORD_FILE);

    if (found) printf("Record Deleted.\n");
    else printf("ID not found.\n");
}
