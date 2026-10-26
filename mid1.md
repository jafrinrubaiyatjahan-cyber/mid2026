# 1번 문항

```#include "raylib.h"
#include <stdbool.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600

#define PADDLE_WIDTH 120
#define PADDLE_HEIGHT 20
#define BALL_RADIUS 8

#define BRICK_ROWS 6
#define BRICK_COLS 10
#define BRICK_WIDTH 70
#define BRICK_HEIGHT 20
#define BRICK_PADDING 8
#define BRICK_OFFSET_X 45
#define BRICK_OFFSET_Y 60

typedef struct {
    Rectangle rect;
    Color color;
    bool active;
} Brick;

typedef struct {
    Rectangle rect;
    float speed;
} Paddle;

typedef struct {
    Vector2 position;
    Vector2 speed;
    float radius;
    bool active;
} Ball;

int main(void) {
    // Initialize window
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Block Breaker - Raylib");
    SetTargetFPS(60);

    // Rainbow colors for the 6 rows
    Color rainbowColors[BRICK_ROWS] = {
        RED,
        ORANGE,
        YELLOW,
        GREEN,
        BLUE,
        PURPLE
    };

    // Initialize Paddle
    Paddle paddle = {
        { (SCREEN_WIDTH - PADDLE_WIDTH) / 2.0f, SCREEN_HEIGHT - 50.0f, PADDLE_WIDTH, PADDLE_HEIGHT },
        8.0f
    };

    // Initialize Ball
    Ball ball = {
        { SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f },
        { 0, 0 },
        BALL_RADIUS,
        false
    };

    // Initialize Bricks
    Brick bricks[BRICK_ROWS][BRICK_COLS];
    for (int i = 0; i < BRICK_ROWS; i++) {
        for (int j = 0; j < BRICK_COLS; j++) {
            bricks[i][j].rect = (Rectangle){
                BRICK_OFFSET_X + j * (BRICK_WIDTH + BRICK_PADDING),
                BRICK_OFFSET_Y + i * (BRICK_HEIGHT + BRICK_PADDING),
                BRICK_WIDTH,
                BRICK_HEIGHT
            };
            bricks[i][j].color = rainbowColors[i];
            bricks[i][j].active = true;
        }
    }

    int score = 0;
    int lives = 3;
    bool gameOver = false;
    bool gameWon = false;

    while (!WindowShouldClose()) {
        // --- UPDATE ---
        if (!gameOver && !gameWon) {
            // Launch ball with Space or Up Arrow
            if (!ball.active) {
                ball.position.x = paddle.rect.x + paddle.rect.width / 2.0f;
                ball.position.y = paddle.rect.y - ball.radius - 2.0f;
                if (IsKeyPressed(KEY_SPACE) || IsKeyPressed(KEY_UP)) {
                    ball.speed = (Vector2){ 5.0f, -5.0f };
                    ball.active = true;
                }
            }

            // Paddle movement (Left/Right or A/D)
            if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) paddle.rect.x -= paddle.speed;
            if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) paddle.rect.x += paddle.speed;

            // Screen boundaries for paddle
            if (paddle.rect.x < 0) paddle.rect.x = 0;
            if (paddle.rect.x > SCREEN_WIDTH - paddle.rect.width) paddle.rect.x = SCREEN_WIDTH - paddle.rect.width;

            // Ball movement & collisions
            if (ball.active) {
                ball.position.x += ball.speed.x;
                ball.position.y += ball.speed.y;

                // Wall collisions (Left, Right, Top)
                if (ball.position.x - ball.radius < 0) {
                    ball.position.x = ball.radius;
                    ball.speed.x *= -1;
                }
                if (ball.position.x + ball.radius > SCREEN_WIDTH) {
                    ball.position.x = SCREEN_WIDTH - ball.radius;
                    ball.speed.x *= -1;
                }
                if (ball.position.y - ball.radius < 0) {
                    ball.position.y = ball.radius;
                    ball.speed.y *= -1;
                }

                // Bottom collision (Lose Life)
                if (ball.position.y + ball.radius > SCREEN_HEIGHT) {
                    lives--;
                    ball.active = false;
                    if (lives <= 0) {
                        gameOver = true;
                    }
                }

                // Paddle collision
                if (CheckCollisionCircleRec(ball.position, ball.radius, paddle.rect)) {
                    ball.speed.y *= -1;
                    // Deflection angle based on where the ball hits the paddle
                    float hitFactor = (ball.position.x - (paddle.rect.x + paddle.rect.width / 2.0f)) / (paddle.rect.width / 2.0f);
                    ball.speed.x = hitFactor * 6.0f;
                    ball.position.y = paddle.rect.y - ball.radius;
                }

                // Brick collisions
                bool allCleared = true;
                for (int i = 0; i < BRICK_ROWS; i++) {
                    for (int j = 0; j < BRICK_COLS; j++) {
                        if (bricks[i][j].active) {
                            allCleared = false;
                            if (CheckCollisionCircleRec(ball.position, ball.radius, bricks[i][j].rect)) {
                                bricks[i][j].active = false;
                                ball.speed.y *= -1;
                                score += (BRICK_ROWS - i) * 10; // Higher rows give more points
                            }
                        }
                    }
                }
                if (allCleared) {
                    gameWon = true;
                }
            }
        }
        else {
            // Restart game on ENTER
            if (IsKeyPressed(KEY_ENTER)) {
                score = 0;
                lives = 3;
                gameOver = false;
                gameWon = false;
                ball.active = false;
                for (int i = 0; i < BRICK_ROWS; i++) {
                    for (int j = 0; j < BRICK_COLS; j++) {
                        bricks[i][j].active = true;
                    }
                }
            }
        }

        // --- DRAW ---
        BeginDrawing();
        ClearBackground(RAYWHITE);

        // Draw rainbow bricks
        for (int i = 0; i < BRICK_ROWS; i++) {
            for (int j = 0; j < BRICK_COLS; j++) {
                if (bricks[i][j].active) {
                    DrawRectangleRec(bricks[i][j].rect, bricks[i][j].color);
                    DrawRectangleLines((int)bricks[i][j].rect.x, (int)bricks[i][j].rect.y,
                        (int)bricks[i][j].rect.width, (int)bricks[i][j].rect.height, Fade(BLACK, 0.15f));
                }
            }
        }

        // Draw Paddle and Ball
        DrawRectangleRec(paddle.rect, DARKGRAY);
        DrawCircleV(ball.position, ball.radius, MAROON);

        // UI text
        DrawText(TextFormat("Score: %i", score), 20, 20, 20, DARKGRAY);
        DrawText(TextFormat("Lives: %i", lives), SCREEN_WIDTH - 110, 20, 20, DARKGRAY);

        if (!ball.active && !gameOver && !gameWon) {
            const char* prompt = "Press SPACE or UP to Launch Ball";
            DrawText(prompt, SCREEN_WIDTH / 2 - MeasureText(prompt, 20) / 2, SCREEN_HEIGHT / 2 + 60, 20, DARKGRAY);
        }

        // Game Over / Win overlays
        if (gameOver) {
            DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT, Fade(BLACK, 0.7f));
            DrawText("GAME OVER", SCREEN_WIDTH / 2 - MeasureText("GAME OVER", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, RED);
            DrawText("Press ENTER to Restart", SCREEN_WIDTH / 2 - MeasureText("Press ENTER to Restart", 20) / 2, SCREEN_HEIGHT / 2 + 20, 20, RAYWHITE);
        }
        else if (gameWon) {
            DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT, Fade(BLACK, 0.7f));
            DrawText("YOU WIN!", SCREEN_WIDTH / 2 - MeasureText("YOU WIN!", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, GREEN);
            DrawText("Press ENTER to Restart", SCREEN_WIDTH / 2 - MeasureText("Press ENTER to Restart", 20) / 2, SCREEN_HEIGHT / 2 + 20, 20, RAYWHITE);
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}

<img width="805" height="669" alt="image" src="https://github.com/user-attachments/assets/4c4be659-37eb-4b8e-9090-e70d8c34e6ad"/>
