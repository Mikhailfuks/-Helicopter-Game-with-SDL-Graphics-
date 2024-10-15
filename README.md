#include <SDL.h>
#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>

const int SCREEN_WIDTH = 800;
const int SCREEN_HEIGHT = 600;
const int HELICOPTER_WIDTH = 50;
const int HELICOPTER_HEIGHT = 30;
const int OBSTACLE_WIDTH = 50;

// Function to check for collision between two rectangles
bool checkCollision(SDL_Rect a, SDL_Rect b) {
    return SDL_HasIntersection(&a, &b);
}

int main(int argc, char* argv[]) {
    SDL_Init(SDL_INIT_VIDEO);
    SDL_Window* window = SDL_CreateWindow("Helicopter Game", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, SCREEN_WIDTH, SCREEN_HEIGHT, 0);
    SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);

    // Initialize helicopter
    SDL_Rect helicopter = {100, SCREEN_HEIGHT / 2, HELICOPTER_WIDTH, HELICOPTER_HEIGHT};
    
    std::vector<SDL_Rect> obstacles;
    std::srand(static_cast<unsigned int>(std::time(0)));

    // Game loop
    bool running = true;
    SDL_Event event;
    Uint32 lastObstacleTime = SDL_GetTicks();
    int obstacleSpeed = 5;
    int gravity = 3;
    int helicopterSpeed = 10;
    
    while (running) {
        // Event polling
        while (SDL_PollEvent(&event)) {
            if (event.type == SDL_QUIT) {
                running = false;
            }
            
            // Handle key events for helicopter movement
            if (event.type == SDL_KEYDOWN) {
                if (event.key.keysym.sym == SDLK_UP) {
                    helicopter.y -= helicopterSpeed;
                }
                else if (event.key.keysym.sym == SDLK_DOWN) {
                    helicopter.y += helicopterSpeed;
                }
            }
        }

        // Apply gravity
        helicopter.y += gravity;

        // Prevent helicopter from going out of bounds
        if (helicopter.y < 0) helicopter.y = 0;
        if (helicopter.y + HELICOPTER_HEIGHT > SCREEN_HEIGHT) helicopter.y = SCREEN_HEIGHT - HELICOPTER_HEIGHT;

        // Create obstacles
        Uint32 currentTime = SDL_GetTicks();
        if (currentTime - lastObstacleTime > 2000) {
            SDL_Rect newObstacle = {SCREEN_WIDTH, std::rand() % (SCREEN_HEIGHT - 100), OBSTACLE_WIDTH, 100};
            obstacles.push_back(newObstacle);
            lastObstacleTime = currentTime;
        }

        // Move obstacles to the left
        for (auto& obstacle : obstacles) {
            obstacle.x -= obstacleSpeed;
        }

        // Remove off-screen obstacles
        if (!obstacles.empty() && obstacles.front().x < -OBSTACLE_WIDTH) {
            obstacles.erase(obstacles.begin());
        }

        // Check for collisions
        for (const auto& obstacle : obstacles) {
            if (checkCollision(helicopter, obstacle)) {
                std::cout << "Game Over!" << std::endl;

                running = false;
                break;
            }
        }

        // Rendering
        SDL_SetRenderDrawColor(renderer, 135, 206, 235, 255); // Sky color
        SDL_RenderClear(renderer);

        // Draw helicopter
        SDL_SetRenderDrawColor(renderer, 255, 0, 0, 255); // Red helicopter
        SDL_RenderFillRect(renderer, &helicopter);

        // Draw obstacles
        SDL_SetRenderDrawColor(renderer, 0, 255, 0, 255); // Green obstacles
        for (const auto& obstacle : obstacles) {
            SDL_RenderFillRect(renderer, &obstacle);
        }

        SDL_RenderPresent(renderer);
        SDL_Delay(16); // Roughly 60 FPS
    }

    // Clean up
    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();
    
    return 0;
}

