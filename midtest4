#include <stdio.h>

int main(void) {
    int n;
    
    printf("피라미드의 높이(n)를 입력하세요: ");
    if (scanf("%d", &n) != 1 || n <= 0) {
        printf("올바른 양의 정수를 입력해주세요.\n");
        return 1;
    }

    printf("\n");

    for (int i = 1; i <= n; i++) {
        // 1. 공백 출력 (피라미드 중앙 정렬)
        for (int j = 0; j < n - i; j++) {
            printf(" ");
        }

        // 2. 사용할 문자 결정 (홀수 행: '*', 짝수 행: '#')
        char symbol = (i % 2 != 0) ? '*' : '#';

        // 3. 기호 출력 (2 * i - 1 개)
        for (int j = 0; j < 2 * i - 1; j++) {
            printf("%c", symbol);
        }

        // 4. 줄바꿈
        printf("\n");
    }

    return 0;
}
