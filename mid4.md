#include <stdio.h>

int main() {
    int N;
    printf("높이 N을 입력하세요: ");
    if (scanf("%d", &N) != 1 || N <= 0) {
        printf("올바른 정수를 입력하세요.\n");
        return 1;
    }

    for (int i = 1; i <= N; i++) {
        // 1. 공백 출력 (중앙 정렬을 위해 N - i 개 만큼 공백 출력)
        for (int j = 0; j < N - i; j++) {
            printf(" ");
        }

        // 2. 문자 출력 (홀수 행은 '*', 짝수 행은 '#', 개수는 2 * i - 1 개)
        char ch = (i % 2 != 0) ? '*' : '#';
        for (int j = 0; j < (2 * i - 1); j++) {
            printf("%c", ch);
        }

        printf("\n");
    }

    return 0;
}
