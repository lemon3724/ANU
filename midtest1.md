#include "raylib.h"
#include <stdbool.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600

#define PLAYER_WIDTH 100
#define PLAYER_HEIGHT 20
#define BALL_RADIUS 8
#define BRICK_ROWS 7
#define BRICK_COLS 10
#define BRICK_WIDTH 70
#define BRICK_HEIGHT 20
#define BRICK_PADDING 10
#define BRICK_OFFSET_X 45
#define BRICK_OFFSET_Y 60

// 무지개 색상 배열 (빨, 주, 노, 초, 파, 남, 보)
Color rainbowColors[BRICK_ROWS] = {
    { 230, 41, 55, 255 },   // 빨강
    { 255, 109, 19, 255 },  // 주황
    { 253, 249, 0, 255 },   // 노랑
    { 0, 228, 48, 255 },    // 초록
    { 0, 121, 241, 255 },   // 파랑
    { 102, 47, 255, 255 },  // 남색
    { 200, 122, 255, 255 }  // 보라
};

typedef struct Paddle {
    Rectangle rect;
    float speed;
} Paddle;

typedef struct Ball {
    Vector2 pos;
    Vector2 speed;
    bool active;
} Ball;

typedef struct Brick {
    Rectangle rect;
    bool active;
    Color color;
} Brick;

int main(void) {
    // 초기화
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Breakout - raylib");
    SetTargetFPS(60);

    // 패들 초기화
    Paddle paddle = {
        { SCREEN_WIDTH / 2.0f - PLAYER_WIDTH / 2.0f, SCREEN_HEIGHT - 50.0f, (float)PLAYER_WIDTH, (float)PLAYER_HEIGHT },
        8.0f
    };

    // 공 초기화
    Ball ball = {
        { SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT - 70.0f },
        { 0.0f, 0.0f },
        false
    };

    // 블록 초기화 (무지개 패턴)
    Brick bricks[BRICK_ROWS][BRICK_COLS];
    for (int i = 0; i < BRICK_ROWS; i++) {
        for (int j = 0; j < BRICK_COLS; j++) {
            bricks[i][j].rect = (Rectangle){
                (float)(BRICK_OFFSET_X + j * (BRICK_WIDTH + BRICK_PADDING)),
                (float)(BRICK_OFFSET_Y + i * (BRICK_HEIGHT + BRICK_PADDING)),
                (float)BRICK_WIDTH,
                (float)BRICK_HEIGHT
            };
            bricks[i][j].active = true;
            bricks[i][j].color = rainbowColors[i];
        }
    }

    int score = 0;
    int lives = 3;
    bool gameOver = false;
    bool gameWon = false;

    // 메인 게임 루프
    while (!WindowShouldClose()) {
        // --- 업데이트 ---
        if (!gameOver && !gameWon) {
            // 스페이스바로 공 발사
            if (!ball.active) {
                ball.pos.x = paddle.rect.x + paddle.rect.width / 2.0f;
                ball.pos.y = paddle.rect.y - BALL_RADIUS - 2;
                if (IsKeyPressed(KEY_SPACE)) {
                    ball.active = true;
                    ball.speed = (Vector2){ 5.0f, -5.0f };
                }
            }

            // 패들 이동 (좌우 방향키 또는 A/D)
            if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) paddle.rect.x -= paddle.speed;
            if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) paddle.rect.x += paddle.speed;

            // 패들 화면 경계 처리
            if (paddle.rect.x < 0) paddle.rect.x = 0;
            if (paddle.rect.x + paddle.rect.width > SCREEN_WIDTH) paddle.rect.x = SCREEN_WIDTH - paddle.rect.width;

            // 공 이동
            if (ball.active) {
                ball.pos.x += ball.speed.x;
                ball.pos.y += ball.speed.y;

                // 벽 충돌 (좌우)
                if (ball.pos.x - BALL_RADIUS < 0) {
                    ball.pos.x = BALL_RADIUS;
                    ball.speed.x *= -1.0f;
                }
                if (ball.pos.x + BALL_RADIUS > SCREEN_WIDTH) {
                    ball.pos.x = SCREEN_WIDTH - BALL_RADIUS;
                    ball.speed.x *= -1.0f;
                }
                // 벽 충돌 (상단)
                if (ball.pos.y - BALL_RADIUS < 0) {
                    ball.pos.y = BALL_RADIUS;
                    ball.speed.y *= -1.0f;
                }

                // 바닥 충돌 (라이프 감소)
                if (ball.pos.y + BALL_RADIUS > SCREEN_HEIGHT) {
                    lives--;
                    ball.active = false;
                    if (lives <= 0) {
                        gameOver = true;
                    }
                }

                // 패들 충돌
                Rectangle ballRect = { ball.pos.x - BALL_RADIUS, ball.pos.y - BALL_RADIUS, BALL_RADIUS * 2, BALL_RADIUS * 2 };
                if (CheckCollisionRecs(ballRect, paddle.rect)) {
                    ball.speed.y *= -1.0f;
                    // 패들의 맞은 위치에 따라 공의 X 방향 속도 조절 (재미 요소)
                    float hitFactor = (ball.pos.x - (paddle.rect.x + paddle.rect.width / 2.0f)) / (paddle.rect.width / 2.0f);
                    ball.speed.x = hitFactor * 6.0f;
                    ball.pos.y = paddle.rect.y - BALL_RADIUS; // 공이 패들 안에 파묻히는 것 방지
                }

                // 블록 충돌 체크
                bool allBricksCleared = true;
                for (int i = 0; i < BRICK_ROWS; i++) {
                    for (int j = 0; j < BRICK_COLS; j++) {
                        if (bricks[i][j].active) {
                            allBricksCleared = false;
                            if (CheckCollisionCircleRec(ball.pos, BALL_RADIUS, bricks[i][j].rect)) {
                                bricks[i][j].active = false;
                                ball.speed.y *= -1.0f;
                                score += (BRICK_ROWS - i) * 10; // 위쪽 블록일수록 높은 점수
                            }
                        }
                    }
                }

                if (allBricksCleared) {
                    gameWon = true;
                }
            }
        }
        else {
            // 게임 오버 혹은 승리 시 재시작 (ENTER 키)
            if (IsKeyPressed(KEY_ENTER)) {
                score = 0;
                lives = 3;
                gameOver = false;
                gameWon = false;
                ball.active = false;
                paddle.rect.x = SCREEN_WIDTH / 2.0f - PLAYER_WIDTH / 2.0f;

                // 블록 리셋
                for (int i = 0; i < BRICK_ROWS; i++) {
                    for (int j = 0; j < BRICK_COLS; j++) {
                        bricks[i][j].active = true;
                    }
                }
            }
        }

        // --- 렌더링 ---
        BeginDrawing();
        ClearBackground(RAYWHITE);

        if (!gameOver && !gameWon) {
            // 블록 그리기 (무지개 색상)
            for (int i = 0; i < BRICK_ROWS; i++) {
                for (int j = 0; j < BRICK_COLS; j++) {
                    if (bricks[i][j].active) {
                        DrawRectangleRec(bricks[i][j].rect, bricks[i][j].color);
                        DrawRectangleLines((int)bricks[i][j].rect.x, (int)bricks[i][j].rect.y, (int)bricks[i][j].rect.width, (int)bricks[i][j].rect.height, Fade(BLACK, 0.2f));
                    }
                }
            }

            // 패들 그리기
            DrawRectangleRec(paddle.rect, DARKGRAY);

            // 공 그리기
            DrawCircleV(ball.pos, (float)BALL_RADIUS, MAROON);

            // UI 표시 (점수, 라이프)
            DrawText(TextFormat("SCORE: %04d", score), 20, 20, 20, DARKGRAY);
            DrawText(TextFormat("LIVES: %d", lives), SCREEN_WIDTH - 120, 20, 20, DARKGRAY);

            if (!ball.active) {
                DrawText("PRESS SPACE TO START", SCREEN_WIDTH / 2.0f - 110, SCREEN_HEIGHT / 2.0f + 50, 20, GRAY);
            }
        }
        else if (gameOver) {
            DrawText("GAME OVER", SCREEN_WIDTH / 2.0f - MeasureText("GAME OVER", 40) / 2, SCREEN_HEIGHT / 2.0f - 40, 40, RED);
            DrawText("PRESS [ENTER] TO RESTART", SCREEN_WIDTH / 2.0f - MeasureText("PRESS [ENTER] TO RESTART", 20) / 2, SCREEN_HEIGHT / 2.0f + 20, 20, DARKGRAY);
        }
        else if (gameWon) {
            DrawText("YOU WIN!", SCREEN_WIDTH / 2.0f - MeasureText("YOU WIN!", 40) / 2, SCREEN_HEIGHT / 2.0f - 40, 40, GREEN);
            DrawText(TextFormat("FINAL SCORE: %d", score), SCREEN_WIDTH / 2.0f - MeasureText(TextFormat("FINAL SCORE: %d", score), 20) / 2, SCREEN_HEIGHT / 2.0f + 10, 20, DARKGRAY);
            DrawText("PRESS [ENTER] TO RESTART", SCREEN_WIDTH / 2.0f - MeasureText("PRESS [ENTER] TO RESTART", 20) / 2, SCREEN_HEIGHT / 2.0f + 50, 20, DARKGRAY);
        }

        EndDrawing();
    }

    // 종료
    CloseWindow();
    return 0;
}
