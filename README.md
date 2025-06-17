import pygame
import random
import sys

# 游戏常量
WIDTH, HEIGHT = 800, 600
GRID_SIZE = 30
MARGIN = 50
ROWS = 16
COLS = 16
MINES = 40

# 颜色
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GRAY = (192, 192, 192)
DARK_GRAY = (128, 128, 128)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
COLORS = [BLUE, (0, 128, 0), RED, (0, 0, 128), (128, 0, 0), 
          (0, 128, 128), BLACK, (128, 128, 128)]

class Minesweeper:
    def __init__(self):
        pygame.init()
        self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption("扫雷")
        self.clock = pygame.time.Clock()
        self.font = pygame.font.SysFont('Arial', 20)
        self.reset_game()
        
    def reset_game(self):
        self.board = [[0 for _ in range(COLS)] for _ in range(ROWS)]
        self.revealed = [[False for _ in range(COLS)] for _ in range(ROWS)]
        self.flagged = [[False for _ in range(COLS)] for _ in range(ROWS)]
        self.game_over = False
        self.win = False
        self.place_mines()
        self.calculate_numbers()
        
    def place_mines(self):
        mines_placed = 0
        while mines_placed < MINES:
            x, y = random.randint(0, ROWS-1), random.randint(0, COLS-1)
            if self.board[x][y] != -1:
                self.board[x][y] = -1
                mines_placed += 1
                
    def calculate_numbers(self):
        for i in range(ROWS):
            for j in range(COLS):
                if self.board[i][j] != -1:
                    count = 0
                    for di in [-1, 0, 1]:
                        for dj in [-1, 0, 1]:
                            ni, nj = i + di, j + dj
                            if 0 <= ni < ROWS and 0 <= nj < COLS and self.board[ni][nj] == -1:
                                count += 1
                    self.board[i][j] = count
                    
    def draw(self):
        self.screen.fill(WHITE)
        
        # 绘制格子
        for i in range(ROWS):
            for j in range(COLS):
                rect = pygame.Rect(
                    MARGIN + j * GRID_SIZE,
                    MARGIN + i * GRID_SIZE,
                    GRID_SIZE, GRID_SIZE
                )
                
                if self.revealed[i][j]:
                    pygame.draw.rect(self.screen, GRAY, rect)
                    if self.board[i][j] > 0:
                        text = self.font.render(str(self.board[i][j]), True, COLORS[self.board[i][j]-1])
                        self.screen.blit(text, (rect.x + 10, rect.y + 5))
                    elif self.board[i][j] == -1:
                        pygame.draw.circle(self.screen, BLACK, rect.center, 10)
                else:
                    pygame.draw.rect(self.screen, DARK_GRAY, rect)
                    if self.flagged[i][j]:
                        pygame.draw.polygon(self.screen, RED, [
                            (rect.x + 15, rect.y + 5),
                            (rect.x + 5, rect.y + 25),
                            (rect.x + 25, rect.y + 25)
                        ])
                
                pygame.draw.rect(self.screen, BLACK, rect, 1)
        
        # 游戏状态显示
        if self.game_over:
            status = "游戏结束! " + ("你赢了!" if self.win else "你输了!")
            text = self.font.render(status, True, RED)
            self.screen.blit(text, (WIDTH//2 - 100, 10))
            
        pygame.display.flip()
        
    def handle_click(self, pos, button):
        if self.game_over:
            return
            
        j = (pos[0] - MARGIN) // GRID_SIZE
        i = (pos[1] - MARGIN) // GRID_SIZE
        
        if 0 <= i < ROWS and 0 <= j < COLS:
            if button == 1 and not self.flagged[i][j]:  # 左键点击
                if self.board[i][j] == -1:  # 点到地雷
                    self.game_over = True
                    self.win = False
                    self.reveal_all()
                else:
                    self.reveal_cells(i, j)
                    self.check_win()
            elif button == 3:  # 右键标记
                if not self.revealed[i][j]:
                    self.flagged[i][j] = not self.flagged[i][j]
                    
    def reveal_cells(self, i, j):
        if not (0 <= i < ROWS and 0 <= j < COLS) or self.revealed[i][j] or self.flagged[i][j]:
            return
            
        self.revealed[i][j] = True
        
        if self.board[i][j] == 0:
            for di in [-1, 0, 1]:
                for dj in [-1, 0, 1]:
                    self.reveal_cells(i + di, j + dj)
                    
    def reveal_all(self):
        for i in range(ROWS):
            for j in range(COLS):
                self.revealed[i][j] = True
                
    def check_win(self):
        for i in range(ROWS):
            for j in range(COLS):
                if self.board[i][j] != -1 and not self.revealed[i][j]:
                    return
        self.game_over = True
        self.win = True
        self.reveal_all()
        
    def run(self):
        running = True
        while running:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    self.handle_click(event.pos, event.button)
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_r:  # 按R重置游戏
                        self.reset_game()
                        
            self.draw()
            self.clock.tick(60)
            
        pygame.quit()
        sys.exit()

if __name__ == "__main__":
    game = Minesweeper()
    game.run()
