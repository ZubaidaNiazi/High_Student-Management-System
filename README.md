
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Student {
    int id;
    char name[50];
    int age;
    char department[30];
};

FILE *fp;

// Function to add a new student
void addStudent() {
    struct Student s;
    fp = fopen("students.dat", "ab");
    if (fp == NULL) {
        printf("File cannot be opened.\n");
        return;
    }

    printf("Enter Student ID: ");
    scanf("%d", &s.id);
    getchar(); // consume newline
    printf("Enter Name: ");
    fgets(s.name, sizeof(s.name), stdin);
    s.name[strcspn(s.name, "\n")] = 0; // remove newline
    printf("Enter Age: ");
    scanf("%d", &s.age);
    getchar();
    printf("Enter Department: ");
    fgets(s.department, sizeof(s.department), stdin);
    s.department[strcspn(s.department, "\n")] = 0;

    fwrite(&s, sizeof(struct Student), 1, fp);
    fclose(fp);
    printf("Student added successfully!\n");
}

// Function to display all students
void displayStudents() {
    struct Student s;
    fp = fopen("students.dat", "rb");
    if (fp == NULL) {
        printf("No records found.\n");
        return;
    }

    printf("\nStudent Records:\n");
    printf("ID\tName\tAge\tDepartment\n");
    printf("---------------------------------------\n");

    while (fread(&s, sizeof(struct Student), 1, fp)) {
        printf("%d\t%s\t%d\t%s\n", s.id, s.name, s.age, s.department);
    }

    fclose(fp);
}

// Function to search a student by ID
void searchStudent() {
    int searchId;
    struct Student s;
    int found = 0;

    printf("Enter Student ID to search: ");
    scanf("%d", &searchId);

    fp = fopen("students.dat", "rb");
    if (fp == NULL) {
        printf("No records found.\n");
        return;
    }

    while (fread(&s, sizeof(struct Student), 1, fp)) {
        if (s.id == searchId) {
            printf("\nStudent Found:\n");
            printf("ID: %d\nName: %s\nAge: %d\nDepartment: %s\n", s.id, s.name, s.age, s.department);
            found = 1;
            break;
        }
    }

    if (!found) {
        printf("Student with ID %d not found.\n", searchId);
    }

    fclose(fp);
}

// Function to update a student record
void updateStudent() {
    int updateId;
    struct Student s;
    int found = 0;

    printf("Enter Student ID to update: ");
    scanf("%d", &updateId);
    getchar();

    fp = fopen("students.dat", "rb+");
    if (fp == NULL) {
        printf("No records found.\n");
        return;
    }

    while (fread(&s, sizeof(struct Student), 1, fp)) {
        if (s.id == updateId) {
            printf("Enter new Name: ");
            fgets(s.name, sizeof(s.name), stdin);
            s.name[strcspn(s.name, "\n")] = 0;
            printf("Enter new Age: ");
            scanf("%d", &s.age);
            getchar();
            printf("Enter new Department: ");
            fgets(s.department, sizeof(s.department), stdin);
            s.department[strcspn(s.department, "\n")] = 0;

            fseek(fp, -sizeof(struct Student), SEEK_CUR);
            fwrite(&s, sizeof(struct Student), 1, fp);
            found = 1;
            printf("Student record updated successfully!\n");
            break;
        }
    }

    if (!found) {
        printf("Student with ID %d not found.\n", updateId);
    }

    fclose(fp);
}

// Function to delete a student record
void deleteStudent() {
    int deleteId;
    struct Student s;
    int found = 0;

    printf("Enter Student ID to delete: ");
    scanf("%d", &deleteId);

    FILE *fpTemp = fopen("temp.dat", "wb");
    fp = fopen("students.dat", "rb");
    if (fp == NULL) {
        printf("No records found.\n");
        fclose(fpTemp);
        return;
    }

    while (fread(&s, sizeof(struct Student), 1, fp)) {
        if (s.id != deleteId) {
            fwrite(&s, sizeof(struct Student), 1, fpTemp);
        } else {
            found = 1;
        }
    }

    fclose(fp);
    fclose(fpTemp);

    remove("students.dat");
    rename("temp.dat", "students.dat");

    if (found)
        printf("Student record deleted successfully!\n");
    else
        printf("Student with ID %d not found.\n", deleteId);
}

int main() {
    int choice;
    while (1) {
        printf("\n===== Student Management System =====\n");
        printf("1. Add Student\n");
        printf("2. Display Students\n");
        printf("3. Search Student\n");
        printf("4. Update Student\n");
        printf("5. Delete Student\n");
        printf("6. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);
        getchar(); // consume newline

        switch (choice) {
            case 1:
                addStudent();
                break;
            case 2:
                displayStudents();
                break;
            case 3:
                searchStudent();
                break;
            case 4:
                updateStudent();
                break;
            case 5:
                deleteStudent();
                break;
            case 6:
                printf("Exiting program...\n");
                exit(0);
            default:
                printf("Invalid choice! Please try again.\n");
        }
    }
    return 0;
}
