#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <termios.h>

#define MAX_USERS 10
#define CREDENTIAL_LENGTH 30

typedef struct {
    char username[CREDENTIAL_LENGTH];
    char password[CREDENTIAL_LENGTH];
} User;

User users[MAX_USERS];
int user_count = 0;

void register_user();
int login_user(); // returns the user index
void fix_fgets_input(char*);
void input_credentials(char* username, char* password, int mask);

int main() {
    int option;
    int user_index;
    while (1) {
        printf("\n=== Welcome to User Management ===");
        printf("\n1. Register");
        printf("\n2. Login");
        printf("\n3. Exit");
        printf("\nSelect an option: ");
        scanf("%d", &option);
        getchar(); // Consume extra enter

        switch (option) {
        case 1:
            register_user();
            break;
        case 2:
            user_index = login_user();
            if (user_index >= 0) {
                printf("\nLogin successful! Welcome, %s!\n", users[user_index].username);
            } else {
                printf("\nLogin failed! Incorrect username or password.\n");
            }
            break;
        case 3:
            printf("\nExiting Program.\n");
            return 0;
        default:
            printf("\nInvalid option. Please try again.\n");
            break;
        }
    }
    return 0;
}

void register_user() {
    if (user_count == MAX_USERS) {
        printf("\nMaximum %d users supported! No more registrations allowed.\n", MAX_USERS);
        return;
    }

    char username[CREDENTIAL_LENGTH];
    char password[CREDENTIAL_LENGTH];
    char confirm_password[CREDENTIAL_LENGTH];

    printf("\nRegister a new user");
    input_credentials(username, password, 1);

    // Check for duplicate username
    for (int i = 0; i < user_count; i++) {
        if (strcmp(username, users[i].username) == 0) {
            printf("\nUsername already exists! Try again.\n");
            return;
        }
    }

    // Confirm password
    printf("\nConfirm password: ");
    input_credentials(username, confirm_password, 1); // reuse username buffer for input
    if (strcmp(password, confirm_password) != 0) {
        printf("\nPasswords do not match! Registration failed.\n");
        return;
    }

    strcpy(users[user_count].username, username);
    strcpy(users[user_count].password, password);
    user_count++;
    printf("\nRegistration successful!\n");
}

int login_user() {
    char username[CREDENTIAL_LENGTH];
    char password[CREDENTIAL_LENGTH];

    input_credentials(username, password, 1);

    for (int i = 0; i < user_count; i++) {
        if (strcmp(username, users[i].username) == 0 &&
            strcmp(password, users[i].password) == 0) {
            return i;
        }
    }
    return -1;
}

void input_credentials(char* username, char* password, int mask) {
    printf("\nEnter username: ");
    fgets(username, CREDENTIAL_LENGTH, stdin);
    fix_fgets_input(username);

    printf("Enter password: ");
    fflush(stdout);

    struct termios old_props, new_props;
    tcgetattr(STDIN_FILENO, &old_props);
    new_props = old_props;
    new_props.c_lflag &= ~(ECHO | ICANON); // FIXED
    tcsetattr(STDIN_FILENO, TCSANOW, &new_props);

    char ch;
    int i = 0;
    while ((ch = getchar()) != '\n' && ch != EOF) {
        if ((ch == '\b' || ch == 127) && i > 0) {
            i--;
            if (mask) printf("\b \b");
        } else {
            password[i++] = ch;
            if (mask) printf("*");
        }
    }
    password[i] = '\0';
    tcsetattr(STDIN_FILENO, TCSANOW, &old_props);
}

void fix_fgets_input(char* string) {
    int index = strcspn(string, "\n");
    string[index] = '\0';
}
