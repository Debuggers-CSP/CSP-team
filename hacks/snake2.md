--- 
layout: opencs 
title: Snake Game 
permalink: /snake 
---

---
layout: opencs
title: Snake Game
permalink: /snake
---

<style>
    body {
        background-color: #1e1e1e;
    }

    .wrap {
        margin-left: auto;
        margin-right: auto;
    }

    canvas {
        display: block;
        border-style: dotted;
        border-width: 10px;
        border-color: #26c62e6b;
    }

    canvas:focus {
        outline: none;
    }

    #gameover p, #setting p, #menu p {
        font-size: 40px;
    }

    #gameover a, #setting a, #menu a {
        font-size: 60px;
        display: block;
    }

    #gameover a:hover, #setting a:hover, #menu a:hover {
        cursor: pointer;
    }

    #gameover a:hover::before, #setting a:hover::before, #menu a:hover::before {
        content: ">";
        margin-right: 20px;
    }

    #menu {
        display: block;
    }

    #gameover, #setting {
        display: none;
    }

    #setting input {
        display: none;
    }

    #setting label {
        cursor: pointer;
    }

    #setting input:checked + label {
        background-color: #881313ff;
        color: #000000;
    }
</style>

<h2>Snake</h2>
<div class="container">
    <p class="fs-4">Score: <span id="score_value">0</span></p>
    <div class="container bg-secondary" style="text-align:center;">
        <!-- Main Menu -->
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake. Use <strong>Arrow Keys</strong> to play.</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>
        <!-- Game Over -->
        <div id="gameover" class="py-4 text-light">
            <p>Game Over. Click <strong>New Game</strong> to play again.</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <!-- Play Screen -->
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <!-- Settings Screen -->
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen. Use <strong>Arrow Keys</strong> to play.</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="240" checked />
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="75" />
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="35" />
                <label for="speed3">Fast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked />
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0" />
                <label for="walloff">Off</label>
            </p>
        </div>
    </div>
</div>

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

        snake.pop();
        snake.unshift({x: _x, y: _y});

        if (wall === 1) {
            if (_x < 0 || _x >= canvas.width / BLOCK || _y < 0 || _y >= canvas.height / BLOCK) {
                return gameOver();
            }
        } else {
            if (_x < 0) _x = canvas.width / BLOCK - 1;
            if (_x >= canvas.width / BLOCK) _x = 0;
            if (_y < 0) _y = canvas.height / BLOCK - 1;
            if (_y >= canvas.height / BLOCK) _y = 0;
            snake[0].x = _x;
            snake[0].y = _y;
        }

        if (snake[0].x === food.x && snake[0].y === food.y) {
            score += 10;
            let new_growth = Math.floor(Math.random() * 5) + 1;
            for(let i = 0; i < new_growth; i++) {
                snake.push({...snake[snake.length - 1]});
            }
            randomizeFood();
        }

        for(let i = 1; i < snake.length; i++){
            if(snake[0].x === snake[i].x && snake[0].y === snake[i].y) return gameOver();
