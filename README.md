# proj2_C
/*
 * Contact Management System in C
 * Concepts used: structures, arrays/strings, functions, file handling
 * Features: add, display, search, edit, delete contacts
 * Data is saved in a binary file (contacts.dat) so it persists between runs.
 */

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <ctype.h>

#define FILE_NAME "contacts.dat"
#define TEMP_FILE "temp.dat"

/* ---------- Structure ---------- */
typedef struct {
    int  id;
    char name[50];
    char phone[15];
    char email[50];
} Contact;

/* ---------- Function declarations ---------- */
void showMenu(void);
void addContact(void);
void displayContacts(void);
void searchContact(void);
void editContact(void);
void deleteContact(void);

void readLine(char *buffer, int size);
int  readInt(void);
int  getNextId(void);
int  isValidPhone(const char *phone);
void toLowerStr(const char *src, char *dest);
void printContact(const Contact *c);

/* ---------- Main ---------- */
int main() {
    int choice;

    do {
        showMenu();
        printf("Enter your choice (1-6): ");
        choice = readInt();

        switch (choice) {
            case 1: addContact();      break;
            case 2: displayContacts(); break;
            case 3: searchContact();   break;
            case 4: editContact();     break;
            case 5: deleteContact();   break;
            case 6: printf("Goodbye!\n"); break;
            default: printf("Invalid choice! Please enter 1-6.\n");
        }
    } while (choice != 6);

    return 0;
}

/* ---------- Menu ---------- */
void showMenu(void) {
    printf("\n====== CONTACT MANAGEMENT SYSTEM ======\n");
    printf("1. Add Contact\n");
    printf("2. Display All Contacts\n");
    printf("3. Search Contact\n");
    printf("4. Edit Contact\n");
    printf("5. Delete Contact\n");
    printf("6. Exit\n");
    printf("=======================================\n");
}

/* ---------- Helper functions ---------- */

/* Reads a line safely (allows spaces) and removes the newline */
void readLine(char *buffer, int size) {
    if (fgets(buffer, size, stdin) != NULL) {
        buffer[strcspn(buffer, "\n")] = '\0';
    } else {
        buffer[0] = '\0';
    }
}

/* Reads an integer safely; returns -1 if input is not a number */
int readInt(void) {
    char line[20];
    int value;
    readLine(line, sizeof(line));
    if (sscanf(line, "%d", &value) != 1)
        return -1;
    return value;
}

/* Returns the next unique ID (max existing ID + 1) */
int getNextId(void) {
    FILE *fp = fopen(FILE_NAME, "rb");
    Contact c;
    int maxId = 0;

    if (fp == NULL)
        return 1;

    while (fread(&c, sizeof(Contact), 1, fp) == 1) {
        if (c.id > maxId)
            maxId = c.id;
    }
    fclose(fp);
    return maxId + 1;
}

/* Phone must be exactly 10 digits */
int isValidPhone(const char *phone) {
    int i;
    if (strlen(phone) != 10)
        return 0;
    for (i = 0; i < 10; i++) {
        if (!isdigit((unsigned char)phone[i]))
            return 0;
    }
    return 1;
}

/* Converts a string to lowercase (for case-insensitive search) */
void toLowerStr(const char *src, char *dest) {
    int i;
    for (i = 0; src[i] != '\0'; i++)
        dest[i] = (char)tolower((unsigned char)src[i]);
    dest[i] = '\0';
}

void printContact(const Contact *c) {
    printf("%-5d %-25s %-15s %-30s\n", c->id, c->name, c->phone, c->email);
}

/* ---------- 1. Add contact ---------- */
void addContact(void) {
    Contact c;
    FILE *fp;

    c.id = getNextId();

    printf("Enter name: ");
    readLine(c.name, sizeof(c.name));
    if (strlen(c.name) == 0) {
        printf("Name cannot be empty. Contact not added.\n");
        return;
    }

    printf("Enter phone (10 digits): ");
    readLine(c.phone, sizeof(c.phone));
    if (!isValidPhone(c.phone)) {
        printf("Invalid phone number. Contact not added.\n");
        return;
    }

    printf("Enter email: ");
    readLine(c.email, sizeof(c.email));

    fp = fopen(FILE_NAME, "ab");
    if (fp == NULL) {
        printf("Error: could not open file!\n");
        return;
    }
    fwrite(&c, sizeof(Contact), 1, fp);
    fclose(fp);

    printf("Contact added successfully! (ID: %d)\n", c.id);
}

/* ---------- 2. Display all contacts ---------- */
void displayContacts(void) {
    FILE *fp = fopen(FILE_NAME, "rb");
    Contact c;
    int count = 0;

    if (fp == NULL) {
        printf("No contacts found.\n");
        return;
    }

    printf("\n%-5s %-25s %-15s %-30s\n", "ID", "Name", "Phone", "Email");
    printf("------------------------------------------------------------------------\n");

    while (fread(&c, sizeof(Contact), 1, fp) == 1) {
        printContact(&c);
        count++;
    }
    fclose(fp);

    if (count == 0)
        printf("No contacts found.\n");
    else
        printf("\nTotal contacts: %d\n", count);
}

/* ---------- 3. Search contact (by name or phone) ---------- */
void searchContact(void) {
    FILE *fp = fopen(FILE_NAME, "rb");
    Contact c;
    char keyword[50], keyLower[50], nameLower[50];
    int found = 0;

    if (fp == NULL) {
        printf("No contacts found.\n");
        return;
    }

    printf("Enter name or phone to search: ");
    readLine(keyword, sizeof(keyword));
    toLowerStr(keyword, keyLower);

    printf("\n%-5s %-25s %-15s %-30s\n", "ID", "Name", "Phone", "Email");
    printf("------------------------------------------------------------------------\n");

    while (fread(&c, sizeof(Contact), 1, fp) == 1) {
        toLowerStr(c.name, nameLower);
        if (strstr(nameLower, keyLower) != NULL || strstr(c.phone, keyword) != NULL) {
            printContact(&c);
            found++;
        }
    }
    fclose(fp);

    if (!found)
        printf("No matching contact found.\n");
}

/* ---------- 4. Edit contact ---------- */
void editContact(void) {
    FILE *fp = fopen(FILE_NAME, "rb+");
    Contact c;
    int id, found = 0;
    char input[50];

    if (fp == NULL) {
        printf("No contacts found.\n");
        return;
    }

    printf("Enter ID of the contact to edit: ");
    id = readInt();

    while (fread(&c, sizeof(Contact), 1, fp) == 1) {
        if (c.id == id) {
            found = 1;
            printf("Current details:\n");
            printContact(&c);
            printf("(Press Enter to keep the existing value)\n");

            printf("New name: ");
            readLine(input, sizeof(input));
            if (strlen(input) > 0)
                strcpy(c.name, input);

            printf("New phone (10 digits): ");
            readLine(input, sizeof(input));
            if (strlen(input) > 0) {
                if (isValidPhone(input))
                    strcpy(c.phone, input);
                else
                    printf("Invalid phone. Keeping old number.\n");
            }

            printf("New email: ");
            readLine(input, sizeof(input));
            if (strlen(input) > 0)
                strcpy(c.email, input);

            /* Move back one record and overwrite it */
            fseek(fp, -(long)sizeof(Contact), SEEK_CUR);
            fwrite(&c, sizeof(Contact), 1, fp);
            printf("Contact updated successfully!\n");
            break;
        }
    }
    fclose(fp);

    if (!found)
        printf("Contact with ID %d not found.\n", id);
}

/* ---------- 5. Delete contact ---------- */
void deleteContact(void) {
    FILE *fp = fopen(FILE_NAME, "rb");
    FILE *temp;
    Contact c;
    int id, found = 0;
    char confirm[10];

    if (fp == NULL) {
        printf("No contacts found.\n");
        return;
    }

    printf("Enter ID of the contact to delete: ");
    id = readInt();

    temp = fopen(TEMP_FILE, "wb");
    if (temp == NULL) {
        printf("Error: could not create temporary file!\n");
        fclose(fp);
        return;
    }

    while (fread(&c, sizeof(Contact), 1, fp) == 1) {
        if (c.id == id) {
            found = 1;
            printf("Contact found:\n");
            printContact(&c);
            printf("Are you sure you want to delete? (y/n): ");
            readLine(confirm, sizeof(confirm));
            if (tolower((unsigned char)confirm[0]) != 'y') {
                fwrite(&c, sizeof(Contact), 1, temp);   /* keep it */
                found = 2;                               /* cancelled */
            }
        } else {
            fwrite(&c, sizeof(Contact), 1, temp);
        }
    }

    fclose(fp);
    fclose(temp);

    remove(FILE_NAME);
    rename(TEMP_FILE, FILE_NAME);

    if (found == 0)
        printf("Contact with ID %d not found.\n", id);
    else if (found == 1)
        printf("Contact deleted successfully!\n");
    else
        printf("Deletion cancelled.\n");
}