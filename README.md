import pygame
import random
import math
import os

# Initialize Pygame
pygame.init()
pygame.mixer.init()

# Constants
WINDOW_WIDTH = 800
WINDOW_HEIGHT = 600
BUBBLE_SIZE = 60
BUBBLE_SPEED = 2
SPAWN_RATE = 400  # milliseconds
GAME_DURATION = 30000  # 30 seconds in milliseconds
FPS = 60

# Colors
BLUE_GRADIENT_START = (0, 198, 255)
BLUE_GRADIENT_END = (0, 114, 255)
WHITE = (255, 255, 255)
BUBBLE_COLOR = (255, 255, 255, 150)  # White with transparency

class Bubble:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.size = BUBBLE_SIZE
        self.speed = random.uniform(1, 3)
        self.alpha = 150
        self.glow_radius = 0
        
    def update(self):
        self.y -= self.speed
        # Add slight floating animation
        self.glow_radius = 15 + 5 * math.sin(pygame.time.get_ticks() * 0.01)
        
    def draw(self, screen):
        # Draw glow effect
        glow_surface = pygame.Surface((self.size + 30, self.size + 30), pygame.SRCALPHA)
        pygame.draw.circle(glow_surface, (255, 255, 255, 30), 
                         (self.size//2 + 15, self.size//2 + 15), self.glow_radius)
        screen.blit(glow_surface, (self.x - 15, self.y - 15))
        
        # Draw bubble
        bubble_surface = pygame.Surface((self.size, self.size), pygame.SRCALPHA)
        pygame.draw.circle(bubble_surface, (255, 255, 255, self.alpha), 
                         (self.size//2, self.size//2), self.size//2)
        
        # Add highlight for bubble effect
        pygame.draw.circle(bubble_surface, (255, 255, 255, 200), 
                         (self.size//2 - 10, self.size//2 - 10), self.size//6)
        
        screen.blit(bubble_surface, (self.x, self.y))
        
    def get_rect(self):
        return pygame.Rect(self.x, self.y, self.size, self.size)
        
    def is_off_screen(self):
        return self.y < -self.size

class BubblePopGame:
    def __init__(self):
        self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
        pygame.display.set_caption("Bubble Pop Game")
        self.clock = pygame.time.Clock()
        self.font = pygame.font.Font(None, 36)
        self.big_font = pygame.font.Font(None, 72)
        
        # Game state
        self.score = 0
        self.bubbles = []
        self.game_start_time = pygame.time.get_ticks()
        self.last_bubble_spawn = 0
        self.game_over = False
        self.running = True
        
        # Try to load background music (optional)
        try:
            # You can replace this with a local music file
            # pygame.mixer.music.load("background_music.ogg")
            # pygame.mixer.music.set_volume(0.5)
            # pygame.mixer.music.play(-1)
            pass
        except:
            print("Background music not found - continuing without music")
            
        # Load pop sound effect (optional)
        try:
            # You can add a pop sound effect here
            # self.pop_sound = pygame.mixer.Sound("pop.wav")
            pass
        except:
            self.pop_sound = None
    
    def create_gradient_background(self):
        """Create a gradient background"""
        for y in range(WINDOW_HEIGHT):
            ratio = y / WINDOW_HEIGHT
            r = int(BLUE_GRADIENT_START[0] * (1 - ratio) + BLUE_GRADIENT_END[0] * ratio)
            g = int(BLUE_GRADIENT_START[1] * (1 - ratio) + BLUE_GRADIENT_END[1] * ratio)
            b = int(BLUE_GRADIENT_START[2] * (1 - ratio) + BLUE_GRADIENT_END[2] * ratio)
            pygame.draw.line(self.screen, (r, g, b), (0, y), (WINDOW_WIDTH, y))
    
    def spawn_bubble(self):
        """Spawn a new bubble at random position"""
        x = random.randint(0, WINDOW_WIDTH - BUBBLE_SIZE)
        y = WINDOW_HEIGHT
        bubble = Bubble(x, y)
        self.bubbles.append(bubble)
    
    def handle_events(self):
        """Handle pygame events"""
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                self.running = False
            elif event.type == pygame.MOUSEBUTTONDOWN and not self.game_over:
                mouse_pos = pygame.mouse.get_pos()
                # Check if any bubble was clicked
                for bubble in self.bubbles[:]:  # Use slice to avoid modification during iteration
                    if bubble.get_rect().collidepoint(mouse_pos):
                        self.score += 1
                        self.bubbles.remove(bubble)
                        # Play pop sound if available
                        if hasattr(self, 'pop_sound') and self.pop_sound:
                            self.pop_sound.play()
                        break
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and self.game_over:
                    self.restart_game()
    
    def update_bubbles(self):
        """Update bubble positions and remove off-screen bubbles"""
        for bubble in self.bubbles[:]:
            bubble.update()
            if bubble.is_off_screen():
                self.bubbles.remove(bubble)
    
    def draw_ui(self):
        """Draw user interface elements"""
        # Draw score
        score_text = self.font.render(f"Score: {self.score}", True, WHITE)
        self.screen.blit(score_text, (10, 10))
        
        # Draw time remaining
        elapsed_time = pygame.time.get_ticks() - self.game_start_time
        time_remaining = max(0, (GAME_DURATION - elapsed_time) // 1000)
        time_text = self.font.render(f"Time: {time_remaining}", True, WHITE)
        self.screen.blit(time_text, (10, 50))
        
        # Draw game over screen
        if self.game_over:
            overlay = pygame.Surface((WINDOW_WIDTH, WINDOW_HEIGHT))
            overlay.set_alpha(128)
            overlay.fill((0, 0, 0))
            self.screen.blit(overlay, (0, 0))
            
            game_over_text = self.big_font.render("Time's Up!", True, WHITE)
            final_score_text = self.font.render(f"Final Score: {self.score}", True, WHITE)
            restart_text = self.font.render("Press SPACE to play again", True, WHITE)
            
            # Center the text
            game_over_rect = game_over_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 - 50))
            score_rect = final_score_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2))
            restart_rect = restart_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 + 50))
            
            self.screen.blit(game_over_text, game_over_rect)
            self.screen.blit(final_score_text, score_rect)
            self.screen.blit(restart_text, restart_rect)
    
    def restart_game(self):
        """Restart the game"""
        self.score = 0
        self.bubbles = []
        self.game_start_time = pygame.time.get_ticks()
        self.last_bubble_spawn = 0
        self.game_over = False
    
    def run(self):
        """Main game loop"""
        while self.running:
            current_time = pygame.time.get_ticks()
            
            # Handle events
            self.handle_events()
            
            if not self.game_over:
                # Check if game time is up
                if current_time - self.game_start_time >= GAME_DURATION:
                    self.game_over = True
                
                # Spawn bubbles
                if current_time - self.last_bubble_spawn >= SPAWN_RATE:
                    self.spawn_bubble()
                    self.last_bubble_spawn = current_time
                
                # Update bubbles
                self.update_bubbles()
            
            # Draw everything
            self.create_gradient_background()
            
            # Draw bubbles
            for bubble in self.bubbles:
                bubble.draw(self.screen)
            
            # Draw UI
            self.draw_ui()
            
            # Update display
            pygame.display.flip()
            self.clock.tick(FPS)
        
        pygame.quit()

if __name__ == "__main__":
    game = BubblePopGame()
    game.run()
    import pygame
import random
import math
import os

# Initialize Pygame
pygame.init()
pygame.mixer.init()

# Constants
WINDOW_WIDTH = 1200
WINDOW_HEIGHT = 800
BUBBLE_SIZE = 100
BUBBLE_SPEED = 2
SPAWN_RATE = 400  # milliseconds
GAME_DURATION = 60000  # 30 seconds in milliseconds
FPS = 60

# Colors
BLUE_GRADIENT_START = (0, 198, 255)
BLUE_GRADIENT_END = (0, 114, 255)
WHITE = (255, 255, 255)
BUBBLE_COLOR = (255, 255, 255, 150)  # White with transparency

class Bubble:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.size = BUBBLE_SIZE
        self.speed = random.uniform(1, 3)
        self.alpha = 150
        self.glow_radius = 0
        
    def update(self):
        self.y -= self.speed
        # Add slight floating animation
        self.glow_radius = 15 + 5 * math.sin(pygame.time.get_ticks() * 0.01)
        
    def draw(self, screen):
        # Draw glow effect
        glow_surface = pygame.Surface((self.size + 30, self.size + 30), pygame.SRCALPHA)
        pygame.draw.circle(glow_surface, (255, 255, 255, 30), 
                         (self.size//2 + 15, self.size//2 + 15), self.glow_radius)
        screen.blit(glow_surface, (self.x - 15, self.y - 15))
        
        # Draw bubble
        bubble_surface = pygame.Surface((self.size, self.size), pygame.SRCALPHA)
        pygame.draw.circle(bubble_surface, (255, 255, 255, self.alpha), 
                         (self.size//2, self.size//2), self.size//2)
        
        # Add highlight for bubble effect
        pygame.draw.circle(bubble_surface, (255, 255, 255, 200), 
                         (self.size//2 - 10, self.size//2 - 10), self.size//6)
        
        screen.blit(bubble_surface, (self.x, self.y))
        
    def get_rect(self):
        return pygame.Rect(self.x, self.y, self.size, self.size)
        
    def is_off_screen(self):
        return self.y < -self.size

class BubblePopGame:
    def __init__(self):
        self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
        pygame.display.set_caption("Bubble Pop Game")
        self.clock = pygame.time.Clock()
        self.font = pygame.font.Font(None, 36)
        self.big_font = pygame.font.Font(None, 72)
        
        # Game state
        self.score = 0
        self.bubbles = []
        self.game_start_time = pygame.time.get_ticks()
        self.last_bubble_spawn = 0
        self.game_over = False
        self.running = True
        
        # Try to load background music (optional)
        try:
            # You can replace this with a local music file
            # pygame.mixer.music.load("background_music.ogg")
            # pygame.mixer.music.set_volume(0.5)
            # pygame.mixer.music.play(-1)
            pass
        except:
            print("Background music not found - continuing without music")
            
        # Load pop sound effect (optional)
        try:
            # You can add a pop sound effect here
            # self.pop_sound = pygame.mixer.Sound("pop.wav")
            pass
        except:
            self.pop_sound = None
    
    def create_gradient_background(self):
        """Create a gradient background"""
        for y in range(WINDOW_HEIGHT):
            ratio = y / WINDOW_HEIGHT
            r = int(BLUE_GRADIENT_START[0] * (1 - ratio) + BLUE_GRADIENT_END[0] * ratio)
            g = int(BLUE_GRADIENT_START[1] * (1 - ratio) + BLUE_GRADIENT_END[1] * ratio)
            b = int(BLUE_GRADIENT_START[2] * (1 - ratio) + BLUE_GRADIENT_END[2] * ratio)
            pygame.draw.line(self.screen, (r, g, b), (0, y), (WINDOW_WIDTH, y))
    
    def spawn_bubble(self):
        """Spawn a new bubble at random position"""
        x = random.randint(0, WINDOW_WIDTH - BUBBLE_SIZE)
        y = WINDOW_HEIGHT
        bubble = Bubble(x, y)
        self.bubbles.append(bubble)
    
    def handle_events(self):
        """Handle pygame events"""
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                self.running = False
            elif event.type == pygame.MOUSEBUTTONDOWN and not self.game_over:
                mouse_pos = pygame.mouse.get_pos()
                # Check if any bubble was clicked
                for bubble in self.bubbles[:]:  # Use slice to avoid modification during iteration
                    if bubble.get_rect().collidepoint(mouse_pos):
                        self.score += 1
                        self.bubbles.remove(bubble)
                        # Play pop sound if available
                        if hasattr(self, 'pop_sound') and self.pop_sound:
                            self.pop_sound.play()
                        break
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and self.game_over:
                    self.restart_game()
    
    def update_bubbles(self):
        """Update bubble positions and remove off-screen bubbles"""
        for bubble in self.bubbles[:]:
            bubble.update()
            if bubble.is_off_screen():
                self.bubbles.remove(bubble)
    
    def draw_ui(self):
        """Draw user interface elements"""
        # Draw score
        score_text = self.font.render(f"Score: {self.score}", True, WHITE)
        self.screen.blit(score_text, (10, 10))
        
        # Draw time remaining
        elapsed_time = pygame.time.get_ticks() - self.game_start_time
        time_remaining = max(0, (GAME_DURATION - elapsed_time) // 1000)
        time_text = self.font.render(f"Time: {time_remaining}", True, WHITE)
        self.screen.blit(time_text, (10, 50))
        
        # Draw game over screen
        if self.game_over:
            overlay = pygame.Surface((WINDOW_WIDTH, WINDOW_HEIGHT))
            overlay.set_alpha(128)
            overlay.fill((0, 0, 0))
            self.screen.blit(overlay, (0, 0))
            
            game_over_text = self.big_font.render("Time's Up!", True, WHITE)
            final_score_text = self.font.render(f"Final Score: {self.score}", True, WHITE)
            restart_text = self.font.render("Press SPACE to play again", True, WHITE)
            
            # Center the text
            game_over_rect = game_over_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 - 50))
            score_rect = final_score_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2))
            restart_rect = restart_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 + 50))
            
            self.screen.blit(game_over_text, game_over_rect)
            self.screen.blit(final_score_text, score_rect)
            self.screen.blit(restart_text, restart_rect)
    
    def restart_game(self):
        """Restart the game"""
        self.score = 0
        self.bubbles = []
        self.game_start_time = pygame.time.get_ticks()
        self.last_bubble_spawn = 0
        self.game_over = False
    
    def run(self):
        """Main game loop"""
        while self.running:
            current_time = pygame.time.get_ticks()
            
            # Handle events
            self.handle_events()
            
            if not self.game_over:
                # Check if game time is up
                if current_time - self.game_start_time >= GAME_DURATION:
                    self.game_over = True
                
                # Spawn bubbles
                if current_time - self.last_bubble_spawn >= SPAWN_RATE:
                    self.spawn_bubble()
                    self.last_bubble_spawn = current_time
                
                # Update bubbles
                self.update_bubbles()
            
            # Draw everything
            self.create_gradient_background()
            
            # Draw bubbles
            for bubble in self.bubbles:
                bubble.draw(self.screen)
            
            # Draw UI
            self.draw_ui()
            
            # Update display
            pygame.display.flip()
            self.clock.tick(FPS)
        
        pygame.quit()

if __name__ == "__main__":
    game = BubblePopGame()
    game.run()# Pygames
