#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int com[3], user[3];
    int strikes, balls, attempts = 0;

    // 난수 생성 초기화
    srand(time(NULL));

    // 컴퓨터가 0~9 사이의 중복되지 않는 3자리 숫자 선택
    com[0] = rand() % 10;
    do {
        com[1] = rand() % 10;
    } while (com[1] == com[0]);

    do {
        com[2] = rand() % 10;
    } while (com[2] == com[0] || com[2] == com[1]);

    printf("숫자야구 게임을 시작합니다! (0~9 사이의 중복 없는 3자리)\n");

    while (1) {
        printf("3자리 숫자를 입력하세요 (예: 1 2 3): ");
        if (scanf("%d %d %d", &user[0], &user[1], &user[2]) != 3) {
            printf("올바른 숫자를 입력하세요.\n");
            while(getchar() != '\n'); // 버퍼 비우기
            continue;
        }

        // 입력값 중복 검사
        if (user[0] == user[1] || user[1] == user[2] || user[0] == user[2]) {
            printf("서로 다른 3개의 숫자를 입력해야 합니다.\n");
            continue;
        }

        attempts++;
        strikes = 0;
        balls = 0;

        // 스트라이크 및 볼 판정
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (user[i] == com[j]) {
                    if (i == j) {
                        strikes++;
                    } else {
                        balls++;
                    }
                }
            }
        }

        // 결과 출력
        if (strikes == 3) {
            printf("정답입니다! %d번 만에 맞추셨습니다.\n", attempts);
            break;
        } else if (strikes == 0 && balls == 0) {
            printf("OUT!\n");
        } else {
            printf("결과: %d 스트라이크, %d 볼\n", strikes, balls);
        }
    }

    return 0;
}
