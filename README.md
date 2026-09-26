import math
import os
import time

# --- Map Configuration ---
MAP_WIDTH = 16
MAP_HEIGHT = 16
MAP = (
    "################"
    "#..............#"
    "#..####...####.#"
    "#..#..#......#.#"
    "#..#..#......#.#"
    "#..####...####.#"
    "#..............#"
    "#.....####.....#"
    "#.....#..#.....#"
    "#.....####.....#"
    "#..............#"
    "#..###....###..#"
    "#..#........#..#"
    "#..#........#..#"
    "#..##########..#"
    "################"
)

# --- Screen Configuration ---
SCREEN_WIDTH = 80
SCREEN_HEIGHT = 40
FOV = math.pi / 4  # Field of View (45 degrees)
DEPTH = 16.0       # Maximum viewing distance

def render_frame(player_x, player_y, player_a):
    frame = []
    
    for x in range(SCREEN_WIDTH):
        # Calculate the projected ray angle for each vertical screen column
        ray_angle = (player_a - FOV / 2.0) + (x / SCREEN_WIDTH) * FOV
        
        distance_to_wall = 0.0
        hit_wall = False
        
        # Vector components of the ray direction
        eye_x = math.sin(ray_angle)
        eye_y = math.cos(ray_angle)
        
        # Cast ray until it hits a wall or max depth
        while not hit_wall and distance_to_wall < DEPTH:
            distance_to_wall += 0.1
            
            test_x = int(player_x + eye_x * distance_to_wall)
            test_y = int(player_y + eye_y * distance_to_wall)
            
            # Check if ray went out of bounds
            if test_x < 0 or test_x >= MAP_WIDTH or test_y < 0 or test_y >= MAP_HEIGHT:
                hit_wall = True
                distance_to_wall = DEPTH
            else:
                # Check if ray cell is a wall block '#'
                if MAP[test_y * MAP_WIDTH + test_x] == '#':
                    hit_wall = True
                    
        # Calculate distance to ceiling and floor based on perspective projection
        ceiling = int((SCREEN_HEIGHT / 2.0) - SCREEN_HEIGHT / distance_to_wall)
        floor = SCREEN_HEIGHT - ceiling
        
        # Shade walls based on depth map distance
        if distance_to_wall <= DEPTH * 0.25:   wall_char = '█' # Close
        elif distance_to_wall <= DEPTH * 0.5:  wall_char = '▓'
        elif distance_to_wall <= DEPTH * 0.75: wall_char = '▒'
        elif distance_to_wall <= DEPTH:        wall_char = '░' # Far
        else:                                  wall_char = ' ' # Out of sight
            
        # Draw the single vertical column array
        for y in range(SCREEN_HEIGHT):
            if y < ceiling:
                frame.append(' ') # Ceiling
            elif y > ceiling and y <= floor:
                frame.append(wall_char) # Wall
            else:
                # Floor shading based on distance
                b = 1.0 - ((y - SCREEN_HEIGHT / 2.0) / (SCREEN_HEIGHT / 2.0))
                if b < 0.25:   floor_char = 'x'
                elif b < 0.5:  floor_char = '~'
                elif b < 0.75: floor_char = '-'
                elif b < 0.9:  floor_char = '.'
                else:          floor_char = ' '
                frame.append(floor_char)
                
    # Format and print the finalized rendering buffer
    lines = [ "".join(frame[i:i+SCREEN_WIDTH]) for i in range(0, len(frame), SCREEN_WIDTH) ]
    os.system('cls' if os.name == 'nt' else 'clear')
    print("\n".join(lines))

def main():
    # Initial player state variables
    player_x = 2.0
    player_y = 2.0
    player_a = 0.0  # Angle
    
    # Run a quick interactive preview loop rotating the camera automatically
    for _ in range(50):
        render_frame(player_x, player_y, player_a)
        player_a += 0.1  # Rotate camera point-of-view angle slowly
        time.sleep(0.05)

if __name__ == "__main__":
    main()
