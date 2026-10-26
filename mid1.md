#include "raylib.h"
#include <stdbool.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600

#define PADDLE_WIDTH 120
#define PADDLE_HEIGHT 20
#define BALL_RADIUS 8
#define BRICK_ROWS 7
#define BRICK_COLS 10
#define BRICK_WIDTH 70
#define BRICK_HEIGHT 22
#define BRICK_PADDING 8
#define BRICK_OFFSET_X 45
#define BRICK_OFFSET_Y 60

// Game States
typedef enum {
    STATE_START,
    STATE_PLAYING,
    STATE_GAME_OVER,
    STATE_VICTORY
} GameState;

typedef struct {
    Vector2 position;
    Vector2 speed;
    float radius;
    bool active;
} Ball;

typedef struct {
    Rectangle rect;
    float speed;
} Paddle;

typedef struct {
    Rectangle rect;
    Color color;
    bool active;
} Brick;

// Rainbow colors corresponding to the 7 rows
const Color RAINBOW_COLORS[BRICK_ROWS] = {
    { 230, 41, 55, 255 },   // Red
    { 253, 249, 0, 255 },   // Orange/Yellow (Raylib Yellow)
    { 255, 161, 0, 255 },   // Orange
    { 0, 228, 48, 255 },    // Green
    { 0, 121, 241, 255 },   // Blue
    { 102, 191, 255, 255 }, // Sky Blue
    { 200, 122, 255, 255 }  // Violet / Purple
};

void InitBricks(Brick bricks[BRICK_ROWS][BRICK_COLS]) {
    for (int i = 0; i < BRICK_ROWS; i++) {
        for (int j = 0; j < BRICK_COLS; j++) {
            bricks[i][j].rect = (Rectangle){
                (float)(BRICK_OFFSET_X + j * (BRICK_WIDTH + BRICK_PADDING)),
                (float)(BRICK_OFFSET_Y + i * (BRICK_HEIGHT + BRICK_PADDING)),
                (float)BRICK_WIDTH,
                (float)BRICK_HEIGHT
            };
            bricks[i][j].color = RAINBOW_COLORS[i];
            bricks[i][j].active = true;
        }
    }
}

void ResetGame(Paddle *paddle, Ball *ball, int *score, int *lives, Brick bricks[BRICK_ROWS][BRICK_COLS]) {
    paddle->rect = (Rectangle){ (SCREEN_WIDTH - PADDLE_WIDTH) / 2.0f, SCREEN_HEIGHT - 50.0f, PADDLE_WIDTH, PADDLE_HEIGHT };
    paddle->speed = 7.0f;

    ball->position = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT - 80.0f };
    ball->speed = (Vector2){ 0.0f, 0.0f };
    ball->radius = BALL_RADIUS;
    ball->active = false;

    *score = 0;
    *lives = 3;

    InitBricks(bricks);
}

int main(void) {
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Block Breaker - Raylib");
    SetTargetFPS(60);

    Paddle paddle;
    Ball ball;
    Brick bricks[BRICK_ROWS][BRICK_COLS];
    int score = 0;
    int lives = 3;
    GameState state = STATE_START;

    ResetGame(&paddle, &ball, &score, &lives, bricks);

    while (!WindowShouldClose()) {
        // --- Update ---
        switch (state) {
            case STATE_START:
                if (IsKeyPressed(KEY_SPACE) || IsKeyPressed(KEY_ENTER)) {
                    state = STATE_PLAYING;
                    ball.speed = (Vector2){ 4.0f, -5.0f };
                    ball.active = true;
                }
                break;

            case STATE_PLAYING:
                // Paddle Movement
                if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) paddle.rect.x -= paddle.speed;
                if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) paddle.rect.x += paddle.speed;

                // Clamp paddle inside screen
                if (paddle.rect.x < 0) paddle.rect.x = 0;
                if (paddle.rect.x > SCREEN_WIDTH - paddle.rect.width) 
                    paddle.rect.x = SCREEN_WIDTH - paddle.rect.width;

                // Ball Movement
                ball.position.x += ball.speed.x;
                ball.position.y += ball.speed.y;

                // Wall Collisions (Left/Right/Top)
                if (ball.position.x - ball.radius < 0) {
                    ball.position.x = ball.radius;
                    ball.speed.x *= -1.0f;
                }
                if (ball.position.x + ball.radius > SCREEN_WIDTH) {
                    ball.position.x = SCREEN_WIDTH - ball.radius;
                    ball.speed.x *= -1.0f;
                }
                if (ball.position.y - ball.radius < 0) {
                    ball.position.y = ball.radius;
                    ball.speed.y *= -1.0f;
                }

                // Bottom Collision (Lose Life)
                if (ball.position.y + ball.radius > SCREEN_HEIGHT) {
                    lives--;
                    if (lives <= 0) {
                        state = STATE_GAME_OVER;
                    } else {
                        // Reset ball and paddle position
                        ball.position = (Vector2){ paddle.rect.x + paddle.rect.width / 2.0f, paddle.rect.y - 15.0f };
                        ball.speed = (Vector2){ 4.0f, -5.0f };
                    }
                }

                // Paddle Collision
                if (CheckCollisionCircleRec(ball.position, ball.radius, paddle.rect)) {
                    ball.speed.y *= -1.0f;
                    // Add English based on where it hit the paddle
                    float hitFactor = (ball.position.x - (paddle.rect.x + paddle.rect.width / 2.0f)) / (paddle.rect.width / 2.0f);
                    ball.speed.x = hitFactor * 6.0f;
                    ball.position.y = paddle.rect.y - ball.radius; // Prevent sticking
                }

                // Brick Collisions
                bool victoryCheck = true;
                for (int i = 0; i < BRICK_ROWS; i++) {
                    for (int j = 0; j < BRICK_COLS; j++) {
                        if (bricks[i][j].active) {
                            victoryCheck = false;
                            if (CheckCollisionCircleRec(ball.position, ball.radius, bricks[i][j].rect)) {
                                bricks[i][j].active = false;
                                ball.speed.y *= -1.0f;
                                score += (BRICK_ROWS - i) * 10; // Higher rows give more points
                            }
                        }
                    }
                }

                if (victoryCheck) {
                    state = STATE_VICTORY;
                }
                break;

            case STATE_GAME_OVER:
            case STATE_VICTORY:
                if (IsKeyPressed(KEY_R)) {
                    ResetGame(&paddle, &ball, &score, &lives, bricks);
                    state = STATE_PLAYING;
                    ball.speed = (Vector2){ 4.0f, -5.0f };
                    ball.active = true;
                }
                break;
        }

        // --- Draw ---
        BeginDrawing();
        ClearBackground(RAYWHITE);

        if (state == STATE_START) {
            DrawText("RAINBOW BLOCK BREAKER", SCREEN_WIDTH / 2 - MeasureText("RAINBOW BLOCK BREAKER", 30) / 2, SCREEN_HEIGHT / 2 - 50, 30, DARKGRAY);
            DrawText("Press SPACE or ENTER to Start", SCREEN_WIDTH / 2 - MeasureText("Press SPACE or ENTER to Start", 20) / 2, SCREEN_HEIGHT / 2 + 10, 20, LIGHTGRAY);
            DrawText("Use A/D or Left/Right Arrow Keys to Move", SCREEN_WIDTH / 2 - MeasureText("Use A/D or Left/Right Arrow Keys to Move", 16) / 2, SCREEN_HEIGHT / 2 + 60, 16, GRAY);
        } else {
            // Draw Bricks
            for (int i = 0; i < BRICK_ROWS; i++) {
                for (int j = 0; j < BRICK_COLS; j++) {
                    if (bricks[i][j].active) {
                        DrawRectangleRec(bricks[i][j].rect, bricks[i][j].color);
                        DrawRectangleLinesEx(bricks[i][j].rect, 1, Fade(BLACK, 0.2f));
                    }
                }
            }

            // Draw Paddle
            DrawRectangleRec(paddle.rect, DARKPURPLE);

            // Draw Ball
            DrawCircleV(ball.position, ball.radius, MAROON);

            // Draw UI (Score & Lives)
            DrawText(TextFormat("SCORE: %04d", score), 20, 20, 20, DARKGRAY);
            DrawText(TextFormat("LIVES: %d", lives), SCREEN_WIDTH - 120, 20, 20, DARKGRAY);

            // Game Over / Victory Overlay
            if (state == STATE_GAME_OVER) {
                DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT, Fade(BLACK, 0.6f));
                DrawText("GAME OVER", SCREEN_WIDTH / 2 - MeasureText("GAME OVER", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, RED);
                DrawText("Press 'R' to Restart", SCREEN_WIDTH / 2 - MeasureText("Press 'R' to Restart", 20) / 2, SCREEN_HEIGHT / 2 + 20, 20, RAYWHITE);
            } else if (state == STATE_VICTORY) {
                DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT, Fade(BLACK, 0.6f));
                DrawText("YOU WIN!", SCREEN_WIDTH / 2 - MeasureText("YOU WIN!", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, GREEN);
                DrawText("Press 'R' to Play Again", SCREEN_WIDTH / 2 - MeasureText("Press 'R' to Play Again", 20) / 2, SCREEN_HEIGHT / 2 + 20, 20, RAYWHITE);
            }
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
