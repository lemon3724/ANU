#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <stdbool.h>

// 중복 없는 3자리 랜덤 숫자 생성 함수
void generateSecret(int secret[]) {
    secret[0] = rand() % 10;
    do {
        secret[1] = rand() % 10;
    } while (secret[1] == secret[0]);
    
    do {
        secret[2] = rand() % 10;
    } while (secret[2] == secret[0] || secret[2] == secret[1]);
}

int main(void) {
    // 난수 생성 초기화
    srand((unsigned int)time(NULL));

    int secret[3];
    int attempts = 0;
    const int maxAttempts = 10; // 최대 10번 기회

    generateSecret(secret);

    printf("=========================================\n");
    printf("     C언어 콘솔 숫자 야구 게임 (Bulls & Cows)\n");
    printf("=========================================\n");
    printf("- 0부터 9 사이의 서로 다른 숫자 3개를 입력하세요.\n");
    printf("- 예시: 1 2 3 또는 123 (띄어쓰기 무관)\n");
    printf("- 총 %d번의 기회가 주어집니다.\n", maxAttempts);
    printf("-----------------------------------------\n\n");

    while (attempts < maxAttempts) {
        int guess[3];
        int input;
        
        printf("[시도 %d/%d] 3자리 숫자 입력: ", attempts + 1, maxAttempts);
        
        // 정수 입력 받기 (3자리 숫자가 한 번에 들어올 경우 처리)
        if (scanf("%d", &input) != 1) {
            printf("잘못된 입력입니다. 숫자를 입력해주세요.\n");
            // 입력 버퍼 비우기
            while (getchar() != '\n');
            continue;
        }

        // 3자리 숫자를 각각 분리
        guess[0] = input / 100;
        guess[1] = (input / 10) % 10;
        guess[2] = input % 10;

        // 입력 유효성 검사 (0~9 범위 및 3자리인지 확인)
        if (input < 100 || input > 999) {
            printf("-> 오류: 3자리 숫자를 입력해야 합니다. (예: 123)\n\n");
            continue;
        }

        // 중복 숫자 입력 검사
        if (guess[0] == guess[1] || guess[0] == guess[2] || guess[1] == guess[2]) {
            printf("-> 오류: 서로 다른 숫자를 입력해야 합니다.\n\n");
            continue;
        }

        int strikes = 0;
        int balls = 0;

        // 스트라이크 및 볼 판정
        for (int i = 0; i < 3; i++) {
            if (guess[i] == secret[i]) {
                strikes++;
            } else {
                for (int j = 0; j < 3; j++) {
                    if (guess[i] == secret[j]) {
                        balls++;
                        break;
                    }
                }
            }
        }

        attempts++;

        // 결과 출력
        if (strikes == 0 && balls == 0) {
            printf("-> 결과: Out!\n\n");
        } else {
            printf("-> 결과: %d 스트라이크, %d 볼\n\n", strikes, balls);
        }

        // 승리 조건 확인
        if (strikes == 3) {
            printf("=========================================\n");
            printf("🎉 정답입니다! %d번 만에 맞추셨습니다! 🎉\n", attempts);
            printf("=========================================\n");
            return 0;
        }
    }

    // 패배 조건 (기회 소진)
    printf("=========================================\n");
    printf("😢 게임 오버! 기회를 모두 소진했습니다.\n");
    printf("정답은 [%d%d%d] 였습니다.\n", secret[0], secret[1], secret[2]);
    printf("=========================================\n");

    return 0;
}
