---
title: "TETRIS in Console"
date: 2024-11-05 17:07:00 +0000
categories: [coding, personal projects]
tags: [c++, oop]
published: true
---
{% raw %}
after doing some small exercises (like making a fully functional calculator) and exploring some advanced programming concepts namely OOP, I decided to take on a bigger project, something way harder and challenging, a tetris game in consol.
# making the block class:
```cpp
class blocks {
public:
    vector<pair<int, int>> block_layer;
    string name;
};
```
a simple class that contains: 
-a vector 'block layer' that holds the coordinates of the block.
-a string that contains the blocks intiats.
this class serves as the foundation for the rest of the code, moving on to how I showed the map!
# showing the grid:
```cpp
void show_map(int Gridlayer[12][12]) {
    for (int row = 1; row < 11; row++) {
        for (int column = 1; column < 11; column++) {
            if (Gridlayer[row][column] == 1) {
                cout << "O "; 
            } else {
                cout << ". "; 
            }
        }
        cout << '\n';
    }
}
```
this function takes the grid layer:
```cpp
    int Gridlayer[12][12] = {
        {2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2},
        {2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2}
    };
```
and loops through the rows and columns, if there the cell contains 1s it prints "O" (a better symbol for clarity, my original printed 1s which was weird), otherwise it will print a period, once it's done it jumps a line, simple enough. The 2s surrounding the grid are there purely to make the out of bounds and collision easier to manage(an idea I learned in one of my python projects).

# The first challenge:
making the blocks appear on the screen wasn't the hardest thing to do if I am to be honest, it porbably ranks the lowest in terms of the most difficult challenges I had to face, nonetheless it required a bit of thinking.
```cpp
void make_block_appear(int Gridlayer[12][12], blocks& I_block, blocks& O_block, blocks& S_block, blocks& Z_block, blocks& L_block, blocks& J_block, blocks& T_block, vector<pair<int, int>>& temp_layer, string& temp_rand) {
    vector<string> random_block_generator = {"I", "S", "Z", "T", "O", "L", "J"};
    srand(static_cast<unsigned int>(time(0)));
    string block_choice;  // Declare block_choice to hold the randomly selected block type
    
    // If temp_rand is not empty, use it; otherwise, choose a random block
    if (temp_rand.empty()) {
        block_choice = random_block_generator[rand() % 7];  // Random block selection
    }
    else {
        block_choice = temp_rand;  // Use predefined block (if temp_rand is passed)
    }

    temp_layer.clear();  // Clear previous block positions

    // Place the block based on block_choice
    if (block_choice == "I") {
        for (auto& pos : I_block.block_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1;
            }
        }
        temp_layer = I_block.block_layer;
    }
    else if (block_choice == "O") {
        for (auto& pos : O_block.block_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1;
            }
        }
        temp_layer = O_block.block_layer;
    }
    else if (block_choice == "S") {
        for (auto& pos : S_block.block_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1;
            }
        }
        temp_layer = S_block.block_layer;
    }
    else if (block_choice == "Z") {
        for (auto& pos : Z_block.block_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1;
            }
        }
        temp_layer = Z_block.block_layer;
    }
    else if (block_choice == "L") {
        for (auto& pos : L_block.block_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1;
            }
        }
        temp_layer = L_block.block_layer;
    }
    else if (block_choice == "J") {
        for (auto& pos : J_block.block_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1;
            }
        }
        temp_layer = J_block.block_layer;
    }
    else if (block_choice == "T") {
        for (auto& pos : T_block.block_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1;
            }
        }
        temp_layer = T_block.block_layer;
    }
}
```
the main idea was to seperate the grid from the blocks, having them on seperate layers provided easier and more robust control over the entire code. If we want to make the block appear, we first need to create a vector that has the names of the blocks, once that's done we will create a variable called "block choice" and we will randomly fill it with random element from the random block generator vector. 
For this we need the "#include <random>", I won't yap about how randomness works in cpp so I'll move on to the next step.
Once the block choice is filled we're going to run a series of if and else if statements that check for each block, if the block choice does equal one of the blocks, it will first check if it can legally be there, now if that's checked the grid will print the coordinates that were stored initially(if you remember that block layer vector we talked about in the blocks class).
we already declared the objects in int main:
```cpp
    blocks I_block = { {{1, 1}, {2, 1}, {3, 1}, {4, 1}}, "I" };
    blocks O_block = { {{1, 1}, {1, 2}, {2, 1}, {2, 2}}, "O" };
    blocks T_block = { {{1, 1}, {1, 2}, {1, 3}, {2, 2}}, "T" };
    blocks L_block = { {{1, 1}, {2, 1}, {3, 1}, {3, 2}}, "L" };
    blocks J_block = { {{1, 2}, {2, 2}, {3, 2}, {3, 1}}, "J" };
    blocks S_block = { {{2, 1}, {2, 2}, {1, 2}, {1, 3}}, "S" };
    blocks Z_block = { {{1, 1}, {1, 2}, {2, 2}, {2, 3}}, "Z" };
```
 and that's it!

 # the second challenge:
 technically this was the third challenge but to understand the third challenge we need to understand how I configured collision in this code. It's safe to say this was the hardest part about this entire project as it took me 3 days alone just to figure this out.

 Collision can be split into two:

-blocks colliding with boundary.
-block colliding with other blocks.

the first wasn't that hard, here is the code.
```cpp
string check_if_out_of_bound(vector<pair<int, int>>& temp_layer) {
    for(auto& pos : temp_layer) {
        if (pos.second == 11) { // Right boundary
            return "right_bound";
        } 
        else if (pos.second == 0) { // Left boundary
            return "left_bound";
        }
    }
    return " ";
}
void OutOfBoundaries(int Gridlayer[12][12], vector<pair<int, int>>& temp_layer) {
    string boundary = check_if_out_of_bound(temp_layer);
    for(auto& pos : temp_layer) {
        if (boundary == "right_bound") {
            pos.second -= 1;
        } 
        else if (boundary == "left_bound") { 
            pos.second += 1;
        }
    }
}
```
since for boundaries we only need to check three sides our job is much easier. For the left and right boundaries the check_if_out_of_bound function checks if the y coordinate of the block eualss 11 or 0 (11 being on the furthest right and 0 on the furthest left) if it is in there, it simply returns either right_bound or left _bound as a string. Once that's over we're going to offset the block one cell to the right or left (respectively). So it's not really colliding it with it but it gives the illusion of collision.

let's move on to the hardest part, blocks colliding with other blocks.
```cpp
vector<pair<int, int>> boundary(vector<pair<int, int>>& temp_layer, int Gridlayer[12][12]) {
    vector<pair<int, int>> boundary_vec;

    auto is_part_of_block = [&](int x, int y) {
        for (const auto& pos : temp_layer) {
            if (pos.first == x && pos.second == y) {
                return true;
            }
        }
        return false;
    };

    for (const auto& pos : temp_layer) {
        int x = pos.first;
        int y = pos.second;

        if (x < 11 && !is_part_of_block(x + 1, y)) {
            boundary_vec.push_back(make_pair(x + 1, y));  
        }
    }

 


    return boundary_vec;
}

bool collision(int Gridlayer[12][12], vector<pair<int, int>>& temp_layer) {
    vector<pair<int, int>> boundary_positions = boundary(temp_layer, Gridlayer);
    for (const auto& pos : boundary_positions) {
        if (Gridlayer[pos.first][pos.second] == 1) {
            return false; 
        }
    }
    return true; 
}
```
let's understand the idea behind it first. I took inspiration from when video games apply hitboxes, basically you take a vector that stores the coordinates of the cells under each cell of the block, then we run a loop and see if it equals 1 (in otherwords if a block is there or not) and that's how I configured collision, but, we can run into multiple problems.

firstly, how does the program recognize if the 1 under it is part of the block or part of the other block?
that's where the function inside the first function comes in, this code defines a lambda function named is_part_of_block, which checks whether a specific coordinate (given by x and y) exists in temp_layer. with that obstacle we can store each cell under the block in a vector and return it once we're done.

the second function deals with the actual collision. there is nothing fancy about it, it just runs a loop and checks if the boundary vector we returned from the previous function is 1 or not (if it is then there is collision and it returns false).

# the third challenge:
before we move to the third challenge which was movement, we're gonna take quick detour to just show the game over function.
```cpp
void gameover() {
    cout << "Game Over! Your score is: " << score << endl;
    exit(0); // Exit the program
}
```
okay it's clear enough, so we can start with explaining movement.

```cpp
void movement(string user_input, int Gridlayer[12][12], 
               blocks& I_block, blocks& O_block, blocks& S_block, 
               blocks& Z_block, blocks& L_block, blocks& J_block, 
               blocks& T_block, vector<pair<int, int>>& temp_layer) {
    
    // Clear old positions in the grid
    for (auto& pos : temp_layer) {
        if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
            Gridlayer[pos.first][pos.second] = 0;
        }
    }

    // Move down
    if (user_input == "s") {
        for (auto& off_set : temp_layer) {
            off_set.first += 1; 
            OutOfBoundaries(Gridlayer, temp_layer);
            
        }
    }
    // Move right
    else if (user_input == "d") {
        for (auto& off_set : temp_layer) {
            off_set.second += 1; 
            OutOfBoundaries(Gridlayer, temp_layer);
            
        }
    }
    // Move left
    else if (user_input == "q") {
        for (auto& off_set : temp_layer) {
            off_set.second -= 1; 
            OutOfBoundaries(Gridlayer, temp_layer);
            
        }
    }
    // Rotation
    else if (user_input == "r") {
        rotation(Gridlayer, temp_layer); 
        OutOfBoundaries(Gridlayer, temp_layer);
        
    }

    // Check for collisions with the updated positions in temp_layer
    if (collision(Gridlayer, temp_layer)) {
        // If no collision  then update the grid with new positions
        for (const auto& pos : temp_layer) {
            if (pos.first >= 0 && pos.first < 12 && pos.second >= 0 && pos.second < 12) {
                Gridlayer[pos.first][pos.second] = 1; // add new positions in grid
            }
        }
    } else {

        for (auto& pos : temp_layer) {
            if (pos.first <= 2) { // Check if any part of the block is at the top row
               gameover(); 
               return;
            }
        }
        // Collision detected
        for (const auto& pos : temp_layer) {
            Gridlayer[pos.first][pos.second] = 1; // Keep current block in the grid
        }
        
        make_block_appear(Gridlayer, I_block, O_block, S_block, Z_block, L_block, J_block, T_block, temp_layer);
    }
}
```
building off our idea of having seperate layers, movement works by removing the block by making every cell with the block's coordinate equal 0 (an empty block) then it registers the user's movement (etiher left down right or rotation), with each movement it checks if the movement the user inputed is out of boundaries (notice we're not talking about block to block collision), if it is the function we talked about is called and it offsets the block right or left depending on where it is.

the second part of the movement function checks for what the collision function returns, if there is collision (the else statement) it keeps the block in its current position and it also calls the gameover function, the code checks if the game has ended by seeing if he cna move AFTER there is collision, if he can't that's game over, if the game is not over it makes a new block apppear since we don't need to move the block anymore.. If there is no collision it add the new block position to the gird (merging layer 0 the grid with layer 1 the block). 

# has the block hit the ground?
I talked about how the code checks for the left and right boundaries but not the ground boundary, this is the code for it:
```cpp
bool isBlockOnGround(vector<pair<int, int>>& temp_layer, int Gridlayer[12][12]) {
   
    for (auto& pos : temp_layer) {
        if (pos.first == 10 ) {  
            return true;  
        }

    }
    if (!collision(Gridlayer, temp_layer)) {
        return false; 
    }
    return false; 
}
```
pretty self explanatory code, if the x position (rows) reach the 10th row then it reached the ground, the function is bool type so it returns true when the block hits the ground, this is useful because we will check for this in int main.

# rotation:
rotation wasn't as hard as people told me it was, although my rotation isn't as robust as the original tetris. 
```cpp
void rotation(int Gridlayer[12][12], vector<pair<int, int>>& temp_layer) {
    int center_X, center_Y;
    int min_X = numeric_limits<int>::max(); 
    int min_Y = numeric_limits<int>::max(); 
    int max_X = numeric_limits<int>::min();
    int max_Y = numeric_limits<int>::min();

   
    for (const auto& offset : temp_layer) {
        if (offset.first < min_X) {
            min_X = offset.first; 
        }
        if (offset.first > max_X) {
            max_X = offset.first; 
        }
        if (offset.second < min_Y) {
            min_Y = offset.second;
        }
        if (offset.second > max_Y) {
            max_Y = offset.second; 
        }
    }

   
    center_X = (min_X + max_X) / 2;
    center_Y = (min_Y + max_Y) / 2;

    
    for (auto& offset : temp_layer) {
        int oldX = offset.first;
        int oldY = offset.second;
        
        
        offset.first = center_X + (oldY - center_Y); 
        offset.second = center_Y - (oldX - center_X); 
    }
}
```
on a 2D plane if you want to rotate (x,y) by 90 degrees in reference to the origin the formula is pretty simple, the new coordinate is (-y,x). Applying this directly in the code is going to give you problems, the block will jump to multiple places as its reference point keeps changing and the rows and columns keeps switching. With some research I saw that the tetris blocks rotate in reference to a cell in the block itself, basically the cell in that block doesn't change its position no amter how many times you rotate it.

the code finds this reference point by finding the center point of the block:
    center_X = (min_X + max_X) / 2;
    center_Y = (min_Y + max_Y) / 2;
if we have that we just need to apply the formula and we're done.
# destroying blocks and scoring:
```cpp
void displayScore() {

    if (score < 500) {
        cout << "LEVEL 1" << "\n";   
        
    }
    else if (score>=500 && score<1000 ) {
        cout << "LEVEL 2" << "\n";  

    }
    else if (score>=1000 && score<1500 ) {
        cout << "LEVEL 2" << "\n";  

    }
    cout << "Your current score is: " << score << "\n";

}
bool is_row_full(int Gridlayer[12][12], int& full_row) {
    for (int row = 10; row > 1; row--) {  
        int count_ones = 0; 
        for (int col = 1; col < 11; col++) {  
            if (Gridlayer[row][col] == 1) {
                count_ones++;
            }
        }
        if (count_ones == 10) { 
            full_row = row;
            score += 100; 
            return true;
        }       
    } 
    return false;
}

void destroy_row(int Gridlayer[12][12], vector<pair<int, int>>& temp_layer) {       
    int full_row; 
    while (is_row_full(Gridlayer, full_row)) {
        for (int row = full_row; row > 1; row--) {  
            for (int col = 1; col < 11; col++) {  
                Gridlayer[row][col] = Gridlayer[row - 1][col];
            }
        }
        
        for (int col = 1; col < 11; col++) {  
            Gridlayer[1][col] = 0;
        }
    }
    displayScore(); 
}
```
the code is divided to three parts:
Row Detection and Scoring: The is_row_full function scans the game grid, starting from the second to last row (index 10) to the first row (index 1), checking if any row is completely filled with blocks (as in if all columns in that row contain a 1). If a full row is found, it updates the full_row reference, adds 100 points to the score, and returns true.

Row Destruction and Grid Update: The destroy_row function repeatedly calls is_row_full to check for full rows. When a full row is detected, the function shifts all rows above it down by one row. This is done by copying the content of each row to the row below it. After shifting, the top-most row (index 1) is reset to all 0s, effectively clearing it.

Score Display: After a row is destroyed and the grid is updated, the displayScore function is called to show the current score. It also adjusts the displayed level based on the updated score: level 1 for scores under 500, level 2 for scores between 500 and 999, and level 3 for scores over 1000.

# UI improvement:
I added these two functions for better UI:
```cpp
void clearConsole() {
    cout << "\033[2J\033[1;1H"; 
}
void custom_print(string variable) {
    cout << "-" << variable << '\n';
}
```
the first clears the consol so the user doesn't have to see all of the bloat, the second just makes the prompts stand out better.

{% endraw %}