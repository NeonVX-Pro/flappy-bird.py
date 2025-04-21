import pygame
import random
import sys

# Initialize pygame
pygame.init()

# Game constants
WINDOW_WIDTH = 400
WINDOW_HEIGHT = 600
GRAVITY = 0.25
BIRD_JUMP = -5
PIPE_GAP = 150
PIPE_FREQUENCY = 1500  # milliseconds

# Color definitions
BIRD_COLOR = (255, 255, 0)    # Yellow
PIPE_COLOR = (0, 255, 0)      # Green
BACKGROUND_COLOR = (78, 192, 202)  # Blue
GROUND_COLOR = (139, 69, 19)  # Brown
WHITE = (255, 255, 255)

# Create window
window = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Flappy Rectangle")
clock = pygame.time.Clock()
font = pygame.font.SysFont('Arial', 30)

class Bird:
    def __init__(self):
        self.x = 100
        self.y = WINDOW_HEIGHT // 2
        self.width = 30
        self.height = 30
        self.velocity = 0
        self.rect = pygame.Rect(self.x, self.y, self.width, self.height)
    
    def update(self):
        self.velocity += GRAVITY
        self.y += self.velocity
        self.rect.y = self.y
    
    def jump(self):
        self.velocity = BIRD_JUMP
    
    def draw(self):
        pygame.draw.rect(window, BIRD_COLOR, self.rect)
        # Draw eye
        pygame.draw.circle(window, WHITE, (self.x + 22, self.y + 10), 5)
        pygame.draw.circle(window, (0, 0, 0), (self.x + 22, self.y + 10), 2)

class Pipe:
    def __init__(self):
        self.gap_y = random.randint(150, WINDOW_HEIGHT - 150 - PIPE_GAP)
        self.x = WINDOW_WIDTH
        self.width = 60
        self.passed = False
        
        self.top_pipe = pygame.Rect(self.x, 0, self.width, self.gap_y)
        self.bottom_pipe = pygame.Rect(self.x, 
                                      self.gap_y + PIPE_GAP, 
                                      self.width, 
                                      WINDOW_HEIGHT - self.gap_y - PIPE_GAP)
    
    def update(self):
        self.x -= 2
        self.top_pipe.x = self.x
        self.bottom_pipe.x = self.x
    
    def draw(self):
        pygame.draw.rect(window, PIPE_COLOR, self.top_pipe)
        pygame.draw.rect(window, PIPE_COLOR, self.bottom_pipe)
        # Draw pipe edges
        pygame.draw.rect(window, (0, 200, 0), 
                        (self.x - 5, self.gap_y - 20, self.width + 10, 20))
        pygame.draw.rect(window, (0, 200, 0), 
                        (self.x - 5, self.gap_y + PIPE_GAP, self.width + 10, 20))

def main():
    bird = Bird()
    pipes = []
    last_pipe = pygame.time.get_ticks()
    score = 0
    game_active = True
    
    while True:
        clock.tick(60)
        
        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and game_active:
                    bird.jump()
                if event.key == pygame.K_SPACE and not game_active:
                    # Reset game
                    bird = Bird()
                    pipes = []
                    score = 0
                    last_pipe = pygame.time.get_ticks()
                    game_active = True
        
        # Background
        window.fill(BACKGROUND_COLOR)
        
        if game_active:
            # Bird update
            bird.update()
            
            # Generate pipes
            time_now = pygame.time.get_ticks()
            if time_now - last_pipe > PIPE_FREQUENCY:
                pipes.append(Pipe())
                last_pipe = time_now
            
            # Pipes update
            for pipe in pipes:
                pipe.update()
                
                # Collision detection
                if (bird.rect.colliderect)(pipe.top_pipe) or \
                   (bird.rect.colliderect(pipe.bottom_pipe)) or \
                   (bird.y + bird.height >= WINDOW_HEIGHT - 100) or \
                   (bird.y < 0):
                    game_active = False
                
                # Score counting
                if pipe.x + pipe.width < bird.x and not pipe.passed:
                    pipe.passed = True
                    score += 1
            
            # Remove off-screen pipes
            pipes = [pipe for pipe in pipes if pipe.x + pipe.width > 0]
        
        # Draw pipes
        for pipe in pipes:
            pipe.draw()
        
        # Draw ground
        pygame.draw.rect(window, GROUND_COLOR, 
                        (0, WINDOW_HEIGHT - 100, WINDOW_WIDTH, 100))
        
        # Draw bird
        bird.draw()
        
        # Draw score
        score_text = font.render(f"Score: {score}", True, WHITE)
        window.blit(score_text, (10, 10))
        
        # Game over message
        if not game_active:
            game_over_text = font.render("Game Over! Press SPACE", True, WHITE)
            window.blit(game_over_text, 
                       (WINDOW_WIDTH//2 - 150, WINDOW_HEIGHT//2 - 15))
        
        pygame.display.update()

if __name__ == "__main__":
    main()
