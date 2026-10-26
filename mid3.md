# 3번 문항

```
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int com[3], user[3];
    int strikes, balls, attempts = 0;

    // Initialize random number generation
    srand(time(NULL));

    // Computer selects 3 unique random digits between 0 and 9
    com[0] = rand() % 10;
    do {
        com[1] = rand() % 10;
    } while (com[1] == com[0]);

    do {
        com[2] = rand() % 10;
    } while (com[2] == com[0] || com[2] == com[1]);

    printf("Starting the Number Baseball Game! (3 unique digits between 0 and 9)\n");

    while (1) {
        printf("Enter 3 numbers (e.g., 1 2 3): ");
        if (scanf_s("%d %d %d", &user[0], &user[1], &user[2]) != 3) {
            printf("Please enter valid numbers.\n");
            while (getchar() != '\n'); // Clear input buffer
            continue;
        }

        // Check for duplicate inputs
        if (user[0] == user[1] || user[1] == user[2] || user[0] == user[2]) {
            printf("You must enter 3 distinct numbers.\n");
            continue;
        }

        attempts++;
        strikes = 0;
        balls = 0;

        // Determine strikes and balls
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (user[i] == com[j]) {
                    if (i == j) {
                        strikes++;
                    }
                    else {
                        balls++;
                    }
                }
            }
        }

        // Print results
        if (strikes == 3) {
            printf("Correct! You got it in %d attempts.\n", attempts);
            break;
        }
        else if (strikes == 0 && balls == 0) {
            printf("OUT!\n");
        }
        else {
            printf("Result: %d Strike(s), %d Ball(s)\n", strikes, balls);
        }
    }

    return 0;
}
}
```
<img width="1136" height="630" alt="image" src="https://github.com/user-attachments/assets/e767e956-c427-487b-9af2-36631826f908" />
