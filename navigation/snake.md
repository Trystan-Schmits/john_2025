---
layout: base
title: Snake
permalink: /snake/
---

<style>

    body{
    }
    .wrap{
        margin-left: auto;
        margin-right: auto;
    }

    canvas{
        display: none;
        border-style: solid;
        border-width: 10px;
        border-color: #FFFFFF;
    }
    canvas:focus{
        outline: none;
    }

    /* All screens style */
    #gameover p, #setting p, #menu p{
        font-size: 20px;
    }

    #gameover a, #setting a, #menu a{
        font-size: 30px;
        display: block;
    }

    #gameover a:hover, #setting a:hover, #menu a:hover{
        cursor: pointer;
    }

    #gameover a:hover::before, #setting a:hover::before, #menu a:hover::before{
        content: ">";
        margin-right: 10px;
    }

    #menu{
        display: block;
    }

    #gameover{
        display: none;
    }

    #setting{
        display: none;
    }

    #setting input{
        display:none;
    }

    #setting label{
        cursor: pointer;
    }

    #setting input:checked + label{
        background-color: #FFF;
        color: #000;
    }
</style>

<h2>Snake</h2>
<div class="container">
    <header class="pb-3 mb-4 border-bottom border-primary text-dark">
        <p class="fs-4">Score: <span id="score_value">0</span></p>
    </header>
    <div class="container bg-secondary" style="text-align:center;">
        <!-- Main Menu -->
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>
        <!-- Game Over -->
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <!-- Play Screen -->
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <!-- Settings Screen -->
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="12" checked/>
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="24"/>
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="60"/>
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
</div>

<script>
    (function(){
        /* Attributes of Game */
        /////////////////////////////////////////////////////////////
        // Canvas & Context
        const canvas = document.getElementById("snake");
        const ctx = canvas.getContext("2d");
        //snake images
        const snake_images = {
            straight: new Image(10,10),
            straight2: new Image(10,10),
            turn: new Image(10,10),
            turn2: new Image(10,10),
            turn3: new Image(10,10),
            turn4: new Image(10,10),
            tail: new Image(10,10),
            tail2: new Image(10,10),
            tail3: new Image(10,10),
            tail4: new Image(10,10),
            head: new Image(10,10),
            head2: new Image(10,10),
            head3: new Image(10,10),
            head4: new Image(10,10),
        }
        snake_images.straight.src = "{{site.baseurl}}/images/snake/snakeStraight.png";
        snake_images.straight2.src = "{{site.baseurl}}/images/snake/snakeStraight2.png";
        snake_images.turn.src = "{{site.baseurl}}/images/snake/snakeTurn.png";
        snake_images.turn2.src = "{{site.baseurl}}/images/snake/snakeTurn2.png";
        snake_images.turn3.src = "{{site.baseurl}}/images/snake/snakeTurn3.png";
        snake_images.turn4.src = "{{site.baseurl}}/images/snake/snakeTurn4.png";
        snake_images.tail.src = "{{site.baseurl}}/images/snake/snakeTail.png";
        snake_images.tail2.src = "{{site.baseurl}}/images/snake/snakeTail2.png";
        snake_images.tail3.src = "{{site.baseurl}}/images/snake/snakeTail3.png";
        snake_images.tail4.src = "{{site.baseurl}}/images/snake/snakeTail4.png";
        snake_images.head.src = "{{site.baseurl}}/images/snake/snakeHead.png";
        snake_images.head2.src = "{{site.baseurl}}/images/snake/snakeHead2.png";
        snake_images.head3.src = "{{site.baseurl}}/images/snake/snakeHead3.png";
        snake_images.head4.src = "{{site.baseurl}}/images/snake/snakeHead4.png";
        // HTML Game IDs
        const SCREEN_SNAKE = 0;
        const screen_snake = document.getElementById("snake");
        const ele_score = document.getElementById("score_value");
        const speed_setting = document.getElementsByName("speed");
        const wall_setting = document.getElementsByName("wall");
        // HTML Screen IDs (div)
        const SCREEN_MENU = -1, SCREEN_GAME_OVER=1, SCREEN_SETTING=2;
        const screen_menu = document.getElementById("menu");
        const screen_game_over = document.getElementById("gameover");
        const screen_setting = document.getElementById("setting");
        // HTML Event IDs (a tags)
        const button_new_game = document.getElementById("new_game");
        const button_new_game1 = document.getElementById("new_game1");
        const button_new_game2 = document.getElementById("new_game2");
        const button_setting_menu = document.getElementById("setting_menu");
        const button_setting_menu1 = document.getElementById("setting_menu1");
        // Game Control
        const BLOCK = 10;   // size of block rendering
        let SCREEN = SCREEN_MENU;
        let snake;
        let snake_dir;
        let snake_next_dir;
        let snake_speed;
        let food = {x: 0, y: 0};
        let score;
        let wall;
        let active = false;


        const directionEnum = {
            up: 0,
            down: 1,
            right: 2,
            left: 3,
        }
        Object.freeze(directionEnum); //freeze the object to be unchangeable

        /* Display Control */
        /////////////////////////////////////////////////////////////
        // 0 for the game
        // 1 for the main menu
        // 2 for the settings screen
        // 3 for the game over screen
        let showScreen = function(screen_opt){
            SCREEN = screen_opt;
            switch(screen_opt){
                case SCREEN_SNAKE:
                    screen_snake.style.display = "block";
                    screen_menu.style.display = "none";
                    screen_setting.style.display = "none";
                    screen_game_over.style.display = "none";
                    break;
                case SCREEN_GAME_OVER:
                    screen_snake.style.display = "block";
                    screen_menu.style.display = "none";
                    screen_setting.style.display = "none";
                    screen_game_over.style.display = "block";
                    break;
                case SCREEN_SETTING:
                    screen_snake.style.display = "none";
                    screen_menu.style.display = "none";
                    screen_setting.style.display = "block";
                    screen_game_over.style.display = "none";
                    break;
            }
        }
        /* Actions and Events  */
        /////////////////////////////////////////////////////////////
        window.onload = function(){
            // HTML Events to Functions
            button_new_game.onclick = function(){newGame();};
            button_new_game1.onclick = function(){newGame();};
            button_new_game2.onclick = function(){newGame();};
            button_setting_menu.onclick = function(){showScreen(SCREEN_SETTING);};
            button_setting_menu1.onclick = function(){showScreen(SCREEN_SETTING);};
            // speed
            setSnakeSpeed(12);
            for(let i = 0; i < speed_setting.length; i++){
                speed_setting[i].addEventListener("click", function(){
                    for(let i = 0; i < speed_setting.length; i++){
                        if(speed_setting[i].checked){
                            setSnakeSpeed(speed_setting[i].value);
                        }
                    }
                });
            }
            // wall setting
            setWall(1);
            for(let i = 0; i < wall_setting.length; i++){
                wall_setting[i].addEventListener("click", function(){
                    for(let i = 0; i < wall_setting.length; i++){
                        if(wall_setting[i].checked){
                            setWall(wall_setting[i].value);
                        }
                    }
                });
            }
            // activate window events
            window.addEventListener("keydown", function(evt) {
                // spacebar detected
                if(evt.code === "Space" && SCREEN !== SCREEN_SNAKE)
                    newGame();
            }, true);
        }
        /* Snake is on the Go (Driver Function)  */
        /////////////////////////////////////////////////////////////
        let mainLoop = function(){
            let _x = snake[0].x;
            let _y = snake[0].y;
            snake_dir = snake_next_dir;   // read async event key
            // Direction 0 - Up, 1 - Right, 2 - Down, 3 - Left
            switch(snake_dir){
                case 0: _y--; break;
                case 1: _x++; break;
                case 2: _y++; break;
                case 3: _x--; break;
            }
            snake.pop(); // tail is removed
            snake.unshift({x: _x, y: _y}); // head is new in new position/orientation
            // Wall Checker
            if(wall === 1){
                // Wall on, Game over test
                if (snake[0].x < 0 || snake[0].x === canvas.width / BLOCK || snake[0].y < 0 || snake[0].y === canvas.height / BLOCK){
                    showScreen(SCREEN_GAME_OVER);
                    active = false;
                    return;
                }
            }else{
                // Wall Off, Circle around
                for(let i = 0, x = snake.length; i < x; i++){
                    if(snake[i].x < 0){
                        snake[i].x = snake[i].x + (canvas.width / BLOCK);
                    }
                    if(snake[i].x === canvas.width / BLOCK){
                        snake[i].x = snake[i].x - (canvas.width / BLOCK);
                    }
                    if(snake[i].y < 0){
                        snake[i].y = snake[i].y + (canvas.height / BLOCK);
                    }
                    if(snake[i].y === canvas.height / BLOCK){
                        snake[i].y = snake[i].y - (canvas.height / BLOCK);
                    }
                }
            }
            // Snake vs Snake checker
            for(let i = 1; i < snake.length; i++){
                // Game over test
                if (snake[0].x === snake[i].x && snake[0].y === snake[i].y){
                    showScreen(SCREEN_GAME_OVER);
                    active = false;
                    return;
                }
            }
            // Snake eats food checker
            if(checkBlock(snake[0].x, snake[0].y, food.x, food.y)){
                snake[snake.length] = {x: snake[0].x, y: snake[0].y};
                altScore(++score);
                addFood();
                activeDot(food.x, food.y);
            }
            // Repaint canvas
            ctx.beginPath();
            ctx.fillStyle = "royalblue";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            // Paint snake
            if(snake.length > 1){
                    let deltaXFront = snake[1].x-snake[0].x;
                    let deltaYFront = snake[1].y-snake[0].y;


                    if(deltaXFront==1 && deltaYFront==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn2);
                    }
                    if(deltaXFront==1 && deltaYFront==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn3);
                    }
                     if(deltaXFront==-1 && deltaYFront==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn);
                    }
                    if(deltaXFront==-1 && deltaYFront==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn4);
                    }
                    if(deltaXFront==1 && deltaYFront==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn4);
                    }
                    if(deltaXFront==-1 && deltaYFront==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn3);
                    }
                    if(deltaXFront==1 && deltaYFront==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn);
                    }
                    if(deltaXFront==-1 && deltaYFront==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn2);
                    }     
                activeDot2(snake[0].x,snake[0].y,"#42f554"); //head
                activeDot2(snake[snake.length-1].x,snake[snake.length-1].y,"#42f554"); //tail
            }else{
                activeDot2(snake[0].x,snake[0].y,"#42f554"); //head
                activeDot2(snake[snake.length-1].x,snake[snake.length-1].y,"#42f554"); //tail
            }
            
            for(let i = 1; i < snake.length-1; i++){
                    let deltaXFront = snake[i].x-snake[i-1].x;
                    let deltaXBack = snake[i].x-snake[i+1].x;
                    let deltaYFront = snake[i].y-snake[i-1].y;
                    let deltaYBack = snake[i].y-snake[i+1].y;

                    if(deltaXFront==1 && deltaYFront-deltaYBack==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn2);
                        continue;
                    }
                    if(deltaXFront==1 && deltaYFront-deltaYBack==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn3);
                        continue;
                    }
                     if(deltaXFront==-1 && deltaYFront-deltaYBack==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn);
                        continue;
                    }
                    if(deltaXFront==-1 && deltaYFront-deltaYBack==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn4);
                        continue;
                    }
                    if(deltaXFront-deltaXBack==1 && deltaYFront==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn4);
                        continue;
                    }
                    if(deltaXFront-deltaXBack==-1 && deltaYFront==1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn3);
                        continue;
                    }
                    if(deltaXFront-deltaXBack==1 && deltaYFront==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn);
                        continue;
                    }
                    if(deltaXFront-deltaXBack==-1 && deltaYFront==-1){
                        activeDot3(snake[i].x, snake[i].y,snake_images.turn2);
                        continue;
                    }      
                    if (Math.abs(deltaXFront-deltaXBack) == 2){
                        activeDot3(snake[i].x, snake[i].y,snake_images.straight);
                        continue;
                    }
                    if (Math.abs(deltaYFront-deltaYBack) == 2){
                        activeDot3(snake[i].x, snake[i].y,snake_images.straight2);
                        continue;
                    }
            }
            // Paint food
            activeDot(food.x, food.y);
            // Debug
            //document.getElementById("debug").innerHTML = snake_dir + " " + snake_next_dir + " " + snake[0].x + " " + snake[0].y;
            // Recursive call after speed delay, déjà vu

            setTimeout(function() {if(active==true){animId = requestAnimationFrame(mainLoop)};}, 1000 / snake_speed);
        }
        /* New Game setup */
        /////////////////////////////////////////////////////////////
        let newGame = function(){
            // snake game screen
            showScreen(SCREEN_SNAKE);
            screen_snake.focus();
            // game score to zero
            score = 0;
            altScore(score);
            // initial snake
            snake = [];
            snake.push({x: 0, y: 15});
            snake_next_dir = 1;
            // food on canvas
            addFood();
            // activate canvas event
            canvas.onkeydown = function(evt) {
                evt.preventDefault();
                let direction = 0;
                switch(evt.key.toLowerCase()){
                    case "w":
                        direction = directionEnum.up;
                        break;
                    case "a":
                        direction = directionEnum.left;
                        break;
                    case "s":
                        direction = directionEnum.down;
                        break;
                    case "d":
                        direction = directionEnum.right;
                        break;
                    case "arrowup":
                        direction = directionEnum.up;
                        break;
                    case "arrowright":
                        direction = directionEnum.right;
                        break;
                    case "arrowleft":
                        direction = directionEnum.left;
                        break;
                    case "arrowdown":
                        direction = directionEnum.down;
                        break;
                }
                changeDir(direction);
            }
            active = true;
            mainLoop();
        }
        /* Key Inputs and Actions */
        /////////////////////////////////////////////////////////////
        let changeDir = function(direction){

            switch(direction) {
                case directionEnum.left:    // left
                    if (snake_dir !== 1)    // not right
                        snake_next_dir = 3; // then switch left
                    break;
                case  directionEnum.up:    // up
                    if (snake_dir !== 2)    // not down
                        snake_next_dir = 0; // then switch up
                    break;
                case  directionEnum.right: // right
                    if (snake_dir !== 3)    // not left
                        snake_next_dir = 1; // then switch right
                    break;
                case directionEnum.down: // down
                    if (snake_dir !== 0)    // not up
                        snake_next_dir = 2; // then switch down
                    break;
            }
        }
        /* Dot for Food or Snake part */
        /////////////////////////////////////////////////////////////
        let activeDot = function(x, y){
            ctx.fillStyle = "#FFFFFF";
            ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
        }
        function activeDot2(x,y,color){
            ctx.fillStyle = color;
            ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
        }
        function activeDot3(x,y,image){
            ctx.drawImage(image, 0, 0, image.width, image.height, x*BLOCK, y*BLOCK, BLOCK, BLOCK);
        }
        /* Random food placement */
        /////////////////////////////////////////////////////////////
        let addFood = function(){
            food.x = Math.floor(Math.random() * ((canvas.width / BLOCK) - 1));
            food.y = Math.floor(Math.random() * ((canvas.height / BLOCK) - 1));
            for(let i = 0; i < snake.length; i++){
                if(checkBlock(food.x, food.y, snake[i].x, snake[i].y)){
                    addFood();
                }
            }
        }
        /* Collision Detection */
        /////////////////////////////////////////////////////////////
        let checkBlock = function(x, y, _x, _y){
            return (x === _x && y === _y);
        }
        /* Update Score */
        /////////////////////////////////////////////////////////////
        let altScore = function(score_val){
            ele_score.innerHTML = String(score_val);
        }
        /////////////////////////////////////////////////////////////
        // Change the snake speed...
        // 150 = slow
        // 100 = normal
        // 50 = fast
        let setSnakeSpeed = function(speed_value){
            snake_speed = speed_value;
        }
        /////////////////////////////////////////////////////////////
        let setWall = function(wall_value){
            wall = wall_value;
            if(wall === 0){screen_snake.style.borderColor = "#606060";}
            if(wall === 1){screen_snake.style.borderColor = "#FFFFFF";}
        }
    })();
</script>