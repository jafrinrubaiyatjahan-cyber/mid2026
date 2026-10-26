# 4번 문항

```
#include <stdio.h>

int main() {
    int N;
    printf("Enter the height N: ");
    if (scanf_s("%d", &N) != 1 || N <= 0) {
        printf("Please enter a valid integer.\n");
        return 1;
    }

    for (int i = 1; i <= N; i++) {
        // 1. Print spaces (N - i spaces for center alignment)
        for (int j = 0; j < N - i; j++) {
            printf(" ");
        }

        // 2. Print characters ('*' for odd rows, '#' for even rows, (2 * i - 1) characters)
        char ch = (i % 2 != 0) ? '*' : '#';
        for (int j = 0; j < (2 * i - 1); j++) {
            printf("%c", ch);
        }

        printf("\n");
    }

    return 0;
}
```
<img width="1135" height="609" alt="image" src="https://github.com/user-attachments/assets/d4ef81d9-ce68-43ce-9282-b0f1bbb8d166" />
