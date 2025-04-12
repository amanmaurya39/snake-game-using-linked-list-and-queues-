# snake-game-using-linked-list-and-queues-
import tkinter as tk
from collections import deque
import random

# Linked list node for snake body
class Node:
    def __init__(self, position):
        self.position = position
        self.next = None

class Snake:
    def __init__(self):
        self.body = deque()
        self.head = Node((100, 100))
        self.body.append(self.head)
        self.current_direction = 'Right'
        self.directions = {
            'Up': (0, -20),
            'Down': (0, 20),
            'Left': (-20, 0),
            'Right': (20, 0)
        }

    def change_direction(self, new_direction):
        opposite = {'Up': 'Down', 'Down': 'Up', 'Left': 'Right', 'Right': 'Left'}
        if new_direction != opposite[self.current_direction]:
            self.current_direction = new_direction

    def move(self, grow=False):
        dx, dy = self.directions[self.current_direction]
        new_head_pos = (self.body[-1].position[0] + dx, self.body[-1].position[1] + dy)
        new_node = Node(new_head_pos)
        self.body[-1].next = new_node
        self.body.append(new_node)

        if not grow:
            self.body.popleft()

        return new_head_pos

    def is_collision(self, pos):
        return any(segment.position == pos for segment in self.body)

class SnakeGame:
    def __init__(self, root):
        self.root = root
        self.root.title("🐍 Snake Game using Queue & Linked List")
        self.canvas = tk.Canvas(root, width=600, height=400, bg='black')
        self.canvas.pack()

        self.snake = Snake()
        self.food_position = None
        self.food = None
        self.running = True
        self.score = 0

        self.root.bind("<KeyPress>", self.on_key_press)
        self.spawn_food()
        self.update()

    def on_key_press(self, event):
        key = event.keysym
        if key in self.snake.directions:
            self.snake.change_direction(key)

    def spawn_food(self):
        while True:
            x = random.randint(0, 29) * 20
            y = random.randint(0, 19) * 20
            if not self.snake.is_collision((x, y)):
                self.food_position = (x, y)
                if self.food:
                    self.canvas.delete(self.food)
                self.food = self.canvas.create_oval(x, y, x+20, y+20, fill="red")
                break

    def update(self):
        if not self.running:
            return

        new_head_pos = self.snake.move(grow=(self.snake.body[-1].position == self.food_position))

        # Check wall collision
        x, y = new_head_pos
        if x < 0 or x >= 600 or y < 0 or y >= 400 or \
                any(segment.position == new_head_pos for segment in list(self.snake.body)[:-1]):
            self.game_over()
            return

        # Clear and redraw everything
        self.canvas.delete("snake")
        for segment in self.snake.body:
            x, y = segment.position
            self.canvas.create_rectangle(x, y, x+20, y+20, fill="green", tags="snake")

        # Check food
        if new_head_pos == self.food_position:
            self.snake.move(grow=True)
            self.spawn_food()
            self.score += 1

        self.root.after(100, self.update)

    def game_over(self):
        self.running = False
        self.canvas.create_text(300, 200, text="GAME OVER", fill="white", font=("Arial", 24, "bold"))
        self.canvas.create_text(300, 240, text=f"Score: {self.score}", fill="yellow", font=("Arial", 16, "bold"))

if __name__ == "__main__":
    root = tk.Tk()
    game = SnakeGame(root)
    root.mainloop()
