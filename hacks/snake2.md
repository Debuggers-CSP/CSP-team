--- 
layout: opencs 
title: Snake Game 
permalink: /snake 
---

<script>
(function(){
    const canvas = document.getElementById("snake");
    const ctx = canvas.getContext("2d");

    const SCREEN_MENU = -1, SCREEN_SNAKE = 0, SCREEN_GAME_OVER = 1, SCREEN_SETTING = 2;
    const screen_snake = document.getElementById("snake");
    const screen_menu = document.getElementById("menu");
    const screen_game_over = document.getElementById("gameover");
    const screen_setting = document.getElementById("setting");
    const ele_score = document.getElementById("score_value");
    const speed_setting = document.getElementsByName("speed");
    const wall_setting = document.getElementsByName("wall");

    const button_new_game = document.getElementById("new_game");
    const button_new_game1 = document.getElementById("new_game1");
    const button_new_game2 = document.getElementById("new_game2");
    const button_setting_menu = document.getElementById("setting_menu");
    const button_setting_menu1 = document.getElementById("setting_menu1");

    const BLOCK = 10;
    const BACKGROUND_COLOR = "#2c2f36";
    const SNAKE_COLOR = "#00ff00";
    const FOOD_COLOR = "#ff0000";

    let SCREEN = SCREEN_MENU;
    let snake, snake_dir, snake_next_dir, snake_speed, wall, score, food;
    let gameLoopInterval = null;

    let showScreen = function(screen_opt){
        SCREEN = screen_opt;
        screen_snake.style.display = (screen_opt === SCREEN_SNAKE || screen_opt === SCREEN_GAME_OVER) ? "block" : "none";
        screen_menu.style.display = (screen_opt === SCREEN_MENU) ? "block" : "none";
        screen_game_over.style.display = (screen_opt === SCREEN_GAME_OVER) ? "block" : "none";
        screen_setting.style.display = (screen_opt === SCREEN_SETTING) ? "block" : "none";
    }

    window.onload = function(){
        button_new_game.onclick = function(){ newGame(); };
        button_new_game1.onclick = function(){ newGame(); };
        button_new_game2.onclick = function(){ newGame(); };
        button_setting_menu.onclick = function(){ showScreen(SCREEN_SETTING); };
        button_setting_menu1.onclick = function(){ showScreen(SCREEN_SETTING); };

        setSnakeSpeed(240);
        setWall(1);

        for(let i = 0; i < speed_setting.length; i++){
            speed_setting[i].addEventListener("click", function(){
                for(let j = 0; j < speed_setting.length; j++){
                    if(speed_setting[j].checked){
                        setSnakeSpeed(speed_setting[j].value);
                    }
                }
            });
        }

        for(let i = 0; i < wall_setting.length; i++){
            wall_setting[i].addEventListener("click", function(){
                for(let j = 0; j < wall_setting.length; j++){
                    if(wall_setting[j].checked){
                        setWall(wall_setting[j].value);
                    }
                }
            });
        }

        window.addEventListener("keydown", function(evt) {
            if (SCREEN !== SCREEN_SNAKE) return;

            const dir = snake_dir;
            switch(evt.key) {
                case "ArrowUp":
                    if (dir !== 2) snake_next_dir = 0;
                    break;
                case "ArrowRight":
                    if (dir !== 3) snake_next_dir = 1;
                    break;
                case "ArrowDown":
                    if (dir !== 0) snake_next_dir = 2;
                    break;
                case "ArrowLeft":
                    if (dir !== 1) snake_next_dir = 3;
                    break;
            }
        });
    }

    let mainLoop = function(){
        let _x = snake[0].x;
        let _y = snake[0].y;

        snake_dir = snake_next_dir;

        switch(snake_dir){
            case 0: _y--; break;
            case 1: _x++; break;
            case 2: _y++; break;
            case 3: _x--; break;
        }

        if (wall === 1) {
            if (_x < 0 || _x >= canvas.width / BLOCK || _y < 0 || _y >= canvas.height / BLOCK) {
                return gameOver();
            }
        } else {
            if (_x < 0) _x = canvas.width / BLOCK - 1;
            if (_x >= canvas.width / BLOCK) _x = 0;
            if (_y < 0) _y = canvas.height / BLOCK - 1;
            if (_y >= canvas.height / BLOCK) _y = 0;
        }

        // Move snake
        snake.pop();
        snake.unshift({x: _x, y: _y});

        // Check for self-collision
        for(let i = 1; i < snake.length; i++){
            if(snake[0].x === snake[i].x && snake[0].y === snake[i].y) return gameOver();
        }

        // Food collision
        if (snake[0].x === food.x && snake[0].y === food.y) {
            score += 10;
            ele_score.textContent = score;
            let new_growth = Math.floor(Math.random() * 5) + 1;
            for(let i = 0; i < new_growth; i++) {
                snake.push({...snake[snake.length - 1]});
            }
            randomizeFood();
        }

        draw();
    }

    function draw(){
        ctx.fillStyle = BACKGROUND_COLOR;
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Draw snake
        ctx.fillStyle = SNAKE_COLOR;
        for(let i = 0; i < snake.length; i++){
            ctx.fillRect(snake[i].x * BLOCK, snake[i].y * BLOCK, BLOCK, BLOCK);
        }

        // Draw food
        ctx.fillStyle = FOOD_COLOR;
        ctx.fillRect(food.x * BLOCK, food.y * BLOCK, BLOCK, BLOCK);
    }

    function newGame() {
        showScreen(SCREEN_SNAKE);
        snake = [
            {x: 5, y: 5},
            {x: 4, y: 5},
            {x: 3, y: 5}
        ];
        snake_dir = 1; // right
        snake_next_dir = 1;
        score = 0;
        ele_score.textContent = score;
        randomizeFood();

        if (gameLoopInterval) {
            clearInterval(gameLoopInterval);
        }

        gameLoopInterval = setInterval(function(){
            mainLoop();
        }, snake_speed);

        canvas.focus();
    }

    function gameOver() {
        clearInterval(gameLoopInterval);
        showScreen(SCREEN_GAME_OVER);
    }

    function randomizeFood() {
        const max_x = canvas.width / BLOCK;
        const max_y = canvas.height / BLOCK;
        food = {
            x: Math.floor(Math.random() * max_x),
            y: Math.floor(Math.random() * max_y)
        };
    }

    function setSnakeSpeed(val) {
        snake_speed = parseInt(val);
        if (gameLoopInterval) {
            clearInterval(gameLoopInterval);
            gameLoopInterval = setInterval(mainLoop, snake_speed);
        }
    }

    function setWall(val) {
        wall = parseInt(val);
    }
})();
</script>
