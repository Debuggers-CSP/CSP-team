---
layout: opencs
title: Snake Game
permalink: /snake
---

<style>
    body {
        background: radial-gradient(circle at center, #111 0%, #000 100%);
        color: white;
        font-family: 'Segoe UI', sans-serif;
        height: 100vh;
        display: flex;
        align-items: center;
        justify-content: center;
        margin: 0;
        overflow: hidden;
    }

    .game-wrapper {
        text-align: center;
        width: 100%;
        max-width: 480px;
    }

    h2 {
        font-size: 40px;
        color: #00ff7f;
        text-shadow: 0 0 10px #00ff7f;
        margin-bottom: 10px;
    }

    p {
        font-size: 18px;
    }

    canvas {
        display: none;
        border: 5px solid white;
        border-radius: 10px;
        box-shadow: 0 0 25px 5px #00ff00;
        background-color: #1e1e1e;
        margin: 20px auto;
    }

    canvas:focus {
        outline: none;
    }

    #menu, #gameover, #setting {
        background-color: rgba(255,255,255,0.05);
        border-radius: 20px;
        padding: 20px;
        margin-top: 20px;
        box-shadow: 0 0 15px #00ff7f;
    }

    a {
        font-size: 22px;
        color: #00ff7f;
        display: block;
        margin: 10px 0;
        text-transform: uppercase;
        text-decoration: none;
        font-weight: bold;
        transition: 0.2s;
    }

    a:hover {
        color: #fff;
        text-shadow: 0 0 10px #00ff7f;
        cursor: pointer;
    }

    #setting input {
        display: none;
    }

    #setting label {
        padding: 5px 10px;
        border-radius: 8px;
        border: 2px solid #00ff7f;
        margin: 0 5px;
        cursor: pointer;
    }

    #setting input:checked + label {
        background-color: #00ff7f;
        color: #000;
    }
</style>

<div class="game-wrapper">
    <h2>Snake Game</h2>
    <p class="fs-4">Score: <span id="score_value">0</span> | Lives: <span id="lives_value">3</span></p>
    
    <!-- Menu -->
    <div id="menu" class="text-light">
        <p>Press <span style="background:#fff;color:#000">Space</span> to start!</p>
        <a id="new_game">New Game</a>
        <a id="setting_menu">Settings</a>
    </div>
    
    <!-- Game Over -->
    <div id="gameover" class="text-light" style="display:none;">
        <p>Game Over! Press <span style="background:#fff;color:#000">Space</span> to try again</p>
        <a id="new_game1">New Game</a>
        <a id="setting_menu1">Settings</a>
    </div>

    <!-- Canvas -->
    <canvas id="snake" width="320" height="320" tabindex="1"></canvas>

    <!-- Settings -->
    <div id="setting" class="text-light" style="display:none;">
        <p>Settings Screen</p>
        <a id="new_game2">Start Game</a>
        <p>Speed:
            <input id="speed1" type="radio" name="speed" value="120" checked/>
            <label for="speed1">Slow</label>
            <input id="speed2" type="radio" name="speed" value="75"/>
            <label for="speed2">Normal</label>
            <input id="speed3" type="radio" name="speed" value="35"/>
            <label for="speed3">Fast</label>
        </p>
        <p>Wall:
            <input id="wallon" type="radio" name="wall" value="1" checked/>
            <label for="wallon">On</label>
            <input id="walloff" type="radio" name="wall" value="0"/>
            <label for="walloff">Off</label>
        </p>
    </div>
</div>

<script>
(function(){
    const canvas = document.getElementById("snake");
    const ctx = canvas.getContext("2d");

    const ele_score = document.getElementById("score_value");
    const ele_lives = document.getElementById("lives_value");
    const speed_setting = document.getElementsByName("speed");
    const wall_setting = document.getElementsByName("wall");

    const screen_menu = document.getElementById("menu");
    const screen_game_over = document.getElementById("gameover");
    const screen_setting = document.getElementById("setting");

    const button_new_game = document.getElementById("new_game");
    const button_new_game1 = document.getElementById("new_game1");
    const button_new_game2 = document.getElementById("new_game2");
    const button_setting_menu = document.getElementById("setting_menu");
    const button_setting_menu1 = document.getElementById("setting_menu1");

    const BLOCK = 10;
    let snake, snake_dir, snake_next_dir, snake_speed, food, score, wall, lives, obstacles;

    const SCREEN_MENU = 0, SCREEN_GAME = 1, SCREEN_OVER = 2, SCREEN_SETTING = 3;
    let SCREEN = SCREEN_MENU;

    const setScreen = (s) => {
        SCREEN = s;
        screen_menu.style.display = s === SCREEN_MENU ? "block" : "none";
        screen_setting.style.display = s === SCREEN_SETTING ? "block" : "none";
        screen_game_over.style.display = s === SCREEN_OVER ? "block" : "none";
        canvas.style.display = s === SCREEN_GAME || s === SCREEN_OVER ? "block" : "none";
        if(s === SCREEN_GAME) canvas.focus();
    };

    const setSnakeSpeed = (val)=> snake_speed = parseInt(val);
    const setWall = (val)=> wall = parseInt(val);

    const altScore = (v)=> ele_score.textContent = v;
    const altLives = (v)=> ele_lives.textContent = v;

    const checkBlock = (x, y, _x, _y) => x === _x && y === _y;

    const addFood = (i) => {
        let newX, newY;
        do {
            newX = Math.floor(Math.random() * (canvas.width / BLOCK));
            newY = Math.floor(Math.random() * (canvas.height / BLOCK));
        } while (snake.some(s => checkBlock(s.x, s.y, newX, newY)) || obstacles.some(o => checkBlock(o.x, o.y, newX, newY)));
        food[i].x = newX;
        food[i].y = newY;
    };

    const addObstacles = () => {
        obstacles = [];
        for (let i = 0; i < 6; i++) {
            let x = Math.floor(Math.random() * (canvas.width / BLOCK));
            let y = Math.floor(Math.random() * (canvas.height / BLOCK));
            obstacles.push({x, y});
        }
    };

    const drawBlock = (x, y, color, glow) => {
        ctx.shadowColor = glow;
        ctx.shadowBlur = 10;
        ctx.fillStyle = color;
        ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
        ctx.shadowBlur = 0;
    };

    const mainLoop = () => {
        let _x = snake[0].x;
        let _y = snake[0].y;
        snake_dir = snake_next_dir;
        if (snake_dir === 0) _y--;
        else if (snake_dir === 1) _x++;
        else if (snake_dir === 2) _y++;
        else if (snake_dir === 3) _x--;

        snake.pop();
        snake.unshift({x: _x, y: _y});

        if(wall === 1){
            if(_x < 0 || _x >= canvas.width / BLOCK || _y < 0 || _y >= canvas.height / BLOCK){
                loseLife();
                if(lives <= 0){ setScreen(SCREEN_OVER); return; }
            }
        } else {
            if(_x < 0) snake[0].x = canvas.width / BLOCK - 1;
            if(_x >= canvas.width / BLOCK) snake[0].x = 0;
            if(_y < 0) snake[0].y = canvas.height / BLOCK - 1;
            if(_y >= canvas.height / BLOCK) snake[0].y = 0;
        }

        for(let i=1;i<snake.length;i++){
            if(checkBlock(snake[0].x, snake[0].y, snake[i].x, snake[i].y)){
                loseLife();
                if(lives <= 0){ setScreen(SCREEN_OVER); return; }
            }
        }
        for(let o of obstacles){
            if(checkBlock(snake[0].x, snake[0].y, o.x, o.y)){
                loseLife();
                if(lives <= 0){ setScreen(SCREEN_OVER); return; }
            }
        }

        for(let i=0;i<food.length;i++){
            if(checkBlock(snake[0].x, snake[0].y, food[i].x, food[i].y)){
                for(let j=0;j<5;j++) snake.push({...snake[snake.length-1]});
                score += food[i].type === "normal" ? 10 : 30;
                altScore(score);
                addFood(i);
            }
        }

        ctx.fillStyle = "#1e1e1e";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        for(let o of obstacles) drawBlock(o.x, o.y, "#808080", "#808080");
        for(let f of food) drawBlock(f.x, f.y, f.type==="normal"?"#ff0000":"#ffd700", f.type==="normal"?"#ff0000":"#ffd700");
        for(let s of snake) drawBlock(s.x, s.y, "#00ff00", "#00ff00");

        setTimeout(mainLoop, snake_speed);
    };

    const loseLife = () => {
        lives--;
        altLives(lives);
        if(lives > 0){
            snake = [{x: 10, y: 10}];
            snake_next_dir = 1;
        }
    };

    const newGame = () => {
        setScreen(SCREEN_GAME);
        score = 0; altScore(score);
        lives = 3; altLives(lives);
        snake = [{x: 10, y: 10}];
        snake_next_dir = 1;
        food = [{x:0, y:0, type:"normal"}, {x:0, y:0, type:"special"}];
        addObstacles();
        addFood(0); addFood(1);
        mainLoop();
    };

    const changeDir = (key) => {
        const k = key.toLowerCase();
        if(["arrowup","w"].includes(k) && snake_dir !== 2) snake_next_dir = 0;
        if(["arrowright","d"].includes(k) && snake_dir !== 3) snake_next_dir = 1;
        if(["arrowdown","s"].includes(k) && snake_dir !== 0) snake_next_dir = 2;
        if(["arrowleft","a"].includes(k) && snake_dir !== 1) snake_next_dir = 3;
    };

    window.onload = () => {
        button_new_game.onclick = newGame;
        button_new_game1.onclick = newGame;
        button_new_game2.onclick = newGame;
        button_setting_menu.onclick = ()=> setScreen(SCREEN_SETTING);
        button_setting_menu1.onclick = ()=> setScreen(SCREEN_SETTING);

        setSnakeSpeed(120);
        for(let s of speed_setting) s.addEventListener("click", ()=> setSnakeSpeed(s.value));

        setWall(1);
        for(let w of wall_setting) w.addEventListener("click", ()=> setWall(w.value));

        window.addEventListener("keydown", (e)=>{
            if(e.code==="Space" && SCREEN!==SCREEN_GAME) newGame();
            if(SCREEN===SCREEN_GAME) changeDir(e.key);
        });
    };
})();
</script>
