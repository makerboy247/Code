<!doctype html> 
<html>
<head>
    <title>My Little Platformer</title>
</head>
<body>
    <div id="SplashScreen" style="border:1px solid #000000; width: 1000px; height: 600px; 
                background-image: url('images/background_sprite1.png'); background-size: 1000px 600px;">
        <div align="center" style = "margin-top:100px; margin-right: 100px; margin-left: 100px;">
            <h1>My Little Platformer</h1>
            <small>by Rana M.Abdullah shahid</small>
            <h2>Instructions</h2>
            <hr>
            <h3>Arrow buttons (or W,A,S,D) to move and jump </h3>
            <h3>Spacebar to shoot</h3>
            <h3>Collect all coins to unlock the exit for next level</h3>
            <h3>Press 'c' to turn on cheat mode and 'v' to turn it off </h3>
            <input type="text" id="playerName" placeholder="Enter your name here" onkeyup="enableStartButton()" />
            <input disabled id="StartButton" type="botton" value="Start" onclick="startGame()" />
        </div>
    </div>

<canvas id="canvas" style="border:1px solid #000000; display:none"></canvas>
<div>
    <input id="RestartButton" type="button" value = "Play again!" onclick="restartGame()" style="position: absolute; left:370px; top:500px;display:none" />
</div>
<script>
// All of our JavaScript will go here. 
function enableStartButton() {
    document.getElementById("StartButton").disabled = false;
}
function restartGame() {
    document.getElementById("RestartButton").style.display = "none";
    givenTime = 100;
    startTime = new Date().getTime();
    isGameOver = false;
    isFirstTime = true;
    currentLevel = 1;
    monsters = [];
    boxes = [];
    goodThings = [];
    keys = [];
    initialiseMap();
    player.x = playerInitX;
    player.y = playerInitY;
    player.bullets = 8;
    player.velX = 0;
    player.velY = 0;
    player.score = initScore;
}

var isStarted = false;
// start game function 
function startGame() {
    var playerName = document.getElementById("playerName").value.trim();
    player.name = playerName == "" ? "Anonymous" : playerName;
    isStarted = true;
    startTime = new Date().getTime();
    initialiseMap();
    document.getElementById("SplashScreen").style.display = "none";
    document.getElementById("canvas").style.display = "block";
}
(function() {
    var requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;
    window.requestAnimationFrame = requestAnimationFrame;
})();
var canvas = document.getElementById("canvas"),
    ctx = canvas.getContext("2d"),
    width = 1000,
    height = 600,
    player = {
      x : width/2-90,
      y : height-25,
      width : 5,
      height : 5,
      speed: 3,
      velX: 0,
      velY: 0,
      jumping:false,
      grounded: false,
      projectileTimer: Date.now(),
      shootDelay: 200,
      name: "Alex",
      score: 0,
      cheatmode: false,
      bullets: 8
    },
    keys = [],
    friction = 0.80,
    gravity = 0.3,
    projectiles = [];
canvas.width = width;
canvas.height = height;

var playerInitX = width/2-90;
var playerInitY = height-25;
var initScore = 0;

var gameWidth = 800;

var monsters = [];
var flyingMonser;
var goodThings = [];
var exit;
var portalStart, portalEnd;

var score = 100;
var startTime;
var givenTime = 100;
var elapsedTime, remainingTime;

// level handling
var isGameOver = false;
var isNextLevel = false;
var isFirstTime = true;
var scoreTable;
var currentLevel = 1;

var facing = "E";
// arbitrary counter
var count = 0;
var  currX, currY;

// for platform creation
var boxes = []

var PATH_CHAR = "images/sprite_sheet2.png";

var CHAR_WIDTH = 24,
    CHAR_HEIGHT = 32,
    IMAGE_START_EAST_Y = 32,
    IMAGE_START_WEST_Y = 96,
    SPRITE_WIDTH = 72;

var COIN_WIDTH = 15,
    COIN_HEIGHT = 16,
    COIN_SPRITE_WIDTH = 60;

var FLYING_MONSTER_WIDTH = 32,
    FLYING_MONSTER_HEIGHT = 32,
    FLYING_MONSTER_SPRITE_WIDTH = 128;

var PROJECTILE_WIDTH = 23,
    PROJECTILE_HEIGHT = 7;

var projectileSpriteY = 0;

var coinSpriteX = 0;

var TEXT_PRELOADING = "Loading ...", 
    TEXT_PRELOADING_X = 200, 
    TEXT_PRELOADING_Y = 200;

var charImage = new Image();
charImage.src = PATH_CHAR;

var monsterImage = new Image();
monsterImage.src = "images/monster_sprite2.png";

var goodThingImage = new Image();
goodThingImage.src = "images/coin_sprite.png";

var tileImage = new Image();
tileImage.src = "images/tile_sprite.png";

var exitImage = new Image();
exitImage.src = "images/exit_sprite.png";

var bgImage = new Image();
bgImage.src = "images/background_sprite1.png";

var teleporterImage = new Image();
teleporterImage.src = "images/teleporter_sprite.png";

var projectileImage = new Image();
projectileImage.src = "images/projectile_sprite.png";

var flyingMonsterImage = new Image();
flyingMonsterImage.src = "images/bat_sprite.png";

var bgMusic = new Audio("sounds/background.wav");
bgMusic.loop = true;
bgMusic.play();

var shootSnd = new Audio("sounds/arrow.wav");
shootSnd.loop = false;

var hitSnd = new Audio("sounds/hit.wav");
hitSnd.loop = false;

var levelupSnd = new Audio("sounds/levelup.wav");
levelupSnd.loop = false;

var gameoverSnd = new Audio("sounds/gameover.wav");
gameoverSnd.loop = false;


currX = 0;
currY = IMAGE_START_EAST_Y;

// E: exit
// X: monster
// G: good thing
// P: start portal; T: end portal
// F: flying monster
// M: movable platform
// #: platform
var GAME_MAP = new Array(
    "#########################################",
    "                                        #",
    " E                                      #",
    "## GG                                   #",
    "  ##                                    #",
    "  ### G                                 #",
    "  ##################################    #",
    "             ##                       ###",
    "             ##  X                   ####",
    "         ############               #####",
    "# T ##                ###################",
    "#########         ##########            #",
    "            ##                          #",
    "           #####                      G #",
    "         ########  F               G MMM#",
    "    ######     ## G             MMMMMMMM#",
    "##              ###   ###               #",
    "###               #   ####### G         #",
    "######            #   #########      X G#",
    "#####     #                        ######",
    "        ####   X                  #######",
    "       #############             ########",
    "      #####                ###########  #",
    " P   ###                ####            #",
    "##                    ####              #", 
    "###           # #                       #",
    "#### G #      ######     X     X   ##   #",
    "########## X         X      X  X     GG #",
    "#################   #####################",
    "#########################################"
    )
function initialiseMap() {
    var y,x;
    for(y=0; y<GAME_MAP.length; y++) {
        var start = null, end = null;
        var isMovable = false;
        for(x=0; x<GAME_MAP[y].length; x++) {
            if(start==null && (GAME_MAP[y].charAt(x) == '#' || GAME_MAP[y].charAt(x) == 'M')) { 
                start = x;
                isMovable = GAME_MAP[y].charAt(x) == 'M' ? true : false;
            }
            if (start != null && GAME_MAP[y].charAt(x) == ' ') {
                end = x - 1;
            }
            if (start != null && x==GAME_MAP[y].length -1) {
                end = x;
            }
            if (start != null && end != null) {
                boxes.push({
                    x: start * 20,
                    y: y*20,
                    width: (end-start+1)*20,
                    height: 20,
                    velY: isMovable ? 0.1 : 0,
                    origY: y*20
                });
                start = end = null;
            }
            if (GAME_MAP[y].charAt(x) == 'X') {
                monsters.push({
                    x: x*20,
                    y: y*20,
                    width: 20,
                    height: 20,
                    direction: "E",
                    velX: Math.random() * (1- 0.4) + 0.4,
                    leftBoundary: x*20-30,
                    rightBoundary: x*20+30,
                    currSpriteX: 0,
                    currSpriteY: IMAGE_START_EAST_Y
                });
            }

            if(GAME_MAP[y].charAt(x) == 'G') {
                goodThings.push({
                    x: x*20,
                    y: y*20,
                    width: 20,
                    height: 20,
                    currSpriteX: 0
                });
            }

            if(GAME_MAP[y].charAt(x) == 'E') {
                exit = {
                    x: x*20,
                    y: y*20,
                    width: 20,
                    height: 20
                };
            }

            if(GAME_MAP[y].charAt(x) == 'P') {
                portalStart = {
                    x: x * 20-17,
                    y: y * 20,
                    width: 20,
                    height: 20
                };
            }
            if(GAME_MAP[y].charAt(x) == 'T') {
                portalEnd = {
                    x: x * 20-18,
                    y: y * 20,
                    width: 32,
                    height: 32
                };
            }
            if(GAME_MAP[y].charAt(x) == 'F') {
                flyingMonster = {
                    x: x * 20,
                    y: y * 20,
                    width: 20,
                    height: 20,
                    velX: 0.2,
                    velY: 0,
                    direction: "E",
                    currSpriteX: 0,
                    currSpriteY: 32
                };
            }
         }
    }
    // game frame
    boxes.push({
        x: 0,
        y: 0,
        width: 10,
        height: height,
        velY: 0
    });
    boxes.push({
        x: 0,
        y: height - 2,
        width: gameWidth,
        height: 50,
        velY: 0
    });
    boxes.push({
        x: gameWidth - 10,
        y: 0,
        width: 50,
        height: height,
        velY: 0
    });
}

function spawnMonsters() {
    var y,x;
    for(y=0; y<GAME_MAP.length; y++) {
        var start = null, end = null;

        for(x=0; x<GAME_MAP[y].length; x++) {
            if (GAME_MAP[y].charAt(x) == 'X') {
                monsters.push({
                    x: x*20,
                    y: y*20,
                    width: 20,
                    height: 20,
                    direction: "E",
                    velX: Math.random(),
                    leftBoundary: x*20-30,
                    rightBoundary: x*20+30,
                    currSpriteX: 0,
                    currSpriteY: IMAGE_START_EAST_Y
                });
            }
        }
    }
}

function update() {
    // console.log("is started: " + isStarted);
    if(isStarted === true) {
        count++;
        if(!isGameOver == true) {
            executeCommands();
        }
        // update values
        updateProjectiles();
        updateSpriteSheetCoordinates();
        updatePlayerVelocity();
        updatePlayerCoordinates();
        updateMonstersCoordinates();
        updateFlyingMonsterTrajectory();

        clearCanvas();

        // draw objects
        drawBackground();
        drawExit();
        drawProjectiles();
        drawPlatforms();
        drawMonsters();
        drawFlyingMonster();
        drawPortals();
        drawPlayer();
        drawGoodThings();
        drawGUI();
        drawPlayerName();


        // handle collision    
        handlePlayerMonsterCollision();
        handleProjectileMonsterCollision();
        handleProjectilePlatformCollision();
        handleProjectileFlyingMonsterCollision()
        handlePlayerGoodThingCollision();
        handlePlayerExitCollision();
        handlePlayerPortalCollision();
        handlePlayerFlyingMonsterCollision();

        handleNextLevel();
        handleGameOver();
    }
    requestAnimationFrame(update);
}
function handlePlayerFlyingMonsterCollision() {
    if (
        flyingMonster != null &&
        player.x < flyingMonster.x + flyingMonster.width &&
        player.x + flyingMonster.width > flyingMonster.x &&
        player.y < flyingMonster.y + flyingMonster.height &&
        player.y + player.height >flyingMonster.y &&
        player.cheatmode == false
    ) {
        isGameOver = true;

    }
}

function updateFlyingMonsterTrajectory() {
    if(flyingMonster != null) {
        var endX = player.x;
        var endY = player.y;
        var startX = flyingMonster.x;
        var startY = flyingMonster.y;
        length = Math.sqrt(Math.pow((endX - startX), 2) + Math.pow((endY - startY), 2));

        flyingMonster.velX = isGameOver ? 0 : (endX - startX) / length;
        flyingMonster.velY = isGameOver ? 0 : (endY - startY) / length;

        var velX = flyingMonster.velX, 
            velY = flyingMonster.velY;

        if(velX < 0 && velX < -0.5) flyingMonster.velX += 0.5;
        if(velX > 0 && velX > 0.5) flyingMonster.velX -= 0.5;
        if(velY < 0 && velY < -0.5) flyingMonster.velY += 0.5;
        if(velY > 0 && velY > 0.5) flyingMonster.velY -= 0.5;

        flyingMonster.direction = flyingMonster.velX >= 0 ? "E" : "W";
    }
    
}

function drawFlyingMonster() {
    if(flyingMonster != null) {
        flyingMonster.x += flyingMonster.velX;
        flyingMonster.y += flyingMonster.velY;
        flyingMonster.currSpriteY = flyingMonster.direction == "E" ? 32 : 96;
        if (count%10==0) flyingMonster.currSpriteX += FLYING_MONSTER_WIDTH;
        if (flyingMonster.currSpriteX >= FLYING_MONSTER_SPRITE_WIDTH) {
            flyingMonster.currSpriteX = 0;
        }
        // ctx.fillStyle = "red";
        // ctx.fillRect(flyingMonster.x, flyingMonster.y, flyingMonster.width, flyingMonster.height);
        ctx.drawImage(flyingMonsterImage, flyingMonster.currSpriteX, flyingMonster.currSpriteY, 
                        FLYING_MONSTER_WIDTH, FLYING_MONSTER_HEIGHT, flyingMonster.x-5, flyingMonster.y-5,
                        FLYING_MONSTER_WIDTH, FLYING_MONSTER_HEIGHT);
    }
}

function drawPortals() {
    // ctx.fillRect(portalStart.x, portalStart.y, portalStart.width, portalStart.height);
    // ctx.fillRect(portalEnd.x, portalEnd.y, portalEnd.width, portalEnd.height);
    ctx.drawImage(teleporterImage, 0, 0, 32, 32, portalStart.x, portalStart.y-9, 32, 32);
    ctx.drawImage(teleporterImage, 0, 0, 32, 32, portalEnd.x, portalEnd.y-9, 32, 32);
}
function drawBackground() {
    ctx.drawImage(bgImage,0,3, 518,436,
                0,0,gameWidth,height);    
}
function drawPlayerName() {
    if(!isGameOver) {
        ctx.font = "10px Arial";
        ctx.fillStyle = "black";
        var len = player.name.length; 
        ctx.fillText(player.name, player.x-len*2, player.y-25);
    }
}
function ScoreRecord(name, score) {
    this.name = name;
    this.score = score;
}

function getHighScoreTable() {
    var scoreTable = new Array();
    // console.log("doc cookie: " + document.cookie);
    for(var i=0; i<10; i++) {
        var cookieName = "player" + i;
        // console.log("player i: " + cookieName);
        // var scoreRecord = getCookie(cookieName);
        var scoreRecord = localStorage.getItem(cookieName);
        if(scoreRecord == null) {
            // console.log("score record is null");
            break;
        } 
        var name = scoreRecord.split("~")[0];
        var score = scoreRecord.split("~")[1];
        // console.log("name: " + name);
        scoreTable.push(new ScoreRecord(name, score));
    }
    return scoreTable;
}
//
// This function stores the high score table to the cookies
//
function setHighScoreTable(table) {
    for (var i = 0; i < 10; i++) {
        // If i is more than the length of the high score table exit
        // from the for loop
        if (i >= table.length) break;

        // Contruct the cookie name
        var cookieName = "player" + i;
        var name = table[i].name;
        var score = table[i].score;

        // Store the ith record as a cookie using the cookie name
        localStorage.setItem(cookieName, name +"~"+score);
    }
}

function addHighScore(name, score) {
    var table = getHighScoreTable();
    // console.log("table length: " + table.length);

    for(var i=0; i<table.length; i++) {
        if(score >= table[i].score) {
            var record = new ScoreRecord(name, score);
            // console.log("splicing table");
            table.splice(i, 0, record);
            return table;
        }
    }
    if (table.length <= 10) {
        table.push(new ScoreRecord(name, score));
    }
    return table;
}
function setCookie(name, value, expires, path, domain, secure) { 
     var curCookie = name + "=" + escape(value) + 
     ((expires) ? "; expires=" + expires.toGMTString() : "") + 
     ((path) ? "; path=" + path : "") + 
     ((domain) ? "; domain=" + domain : "") + 
     ((secure) ? "; secure" : ""); 
     // console.log("curCookie: " + curCookie);
     document.cookie = curCookie; 
     // console.log("document cookie: " + document.cookie);
} 


function getCookie(name) {
     var dc = document.cookie; 
     var prefix = name + "="; 
     var begin = dc.indexOf("; " + prefix); 
     if (begin == -1) { 
        begin = dc.indexOf(prefix); 
     if (begin != 0) return null; 
     } else begin += 2; 
     var end = document.cookie.indexOf(";", begin); 
     if (end == -1) end = dc.length; 
     return unescape(dc.substring(begin + prefix.length, end)); 
}

function handleNextLevel() {
    if(isNextLevel == true) {
        // notify player that he finished the current level
        alert("You owned this level :)");
        // reposition char at origin
        player.x = playerInitX;
        player.y = playerInitY;
        player.velX = 0;
        player.velY = 0;
        player.bullets = 8;
        keys = [];
        // update score + remaining secs
        player.score += remainingTime;
        player.score += (100*currentLevel);
        // update GUI with current level number
        currentLevel++;
        // update GUI with new timer + remaining sec
        givenTime = 100 + remainingTime;
        startTime = new Date().getTime();
        // populate map with more monsters
        boxes = [];
        monsters = [];
        goodThings = [];
        initialiseMap();
        for(var i=1; i<currentLevel; i++) {
            spawnMonsters();
        }
        isNextLevel = false;
    }
}

function handleGameOver() {
    if(isGameOver == true) { 
        if(isFirstTime == true) {
            var table = addHighScore(player.name, player.score);
            setHighScoreTable(table);
            gameoverSnd.play();
        }
        drawGameOverScreen();
        document.getElementById("RestartButton").style.display = "block";
        isFirstTime = false;
    }
}
function drawGameOverScreen() {
    ctx.fillStyle = "rgba(0, 0, 200, 0.7)";
    ctx.fillRect(50, 50, gameWidth-100, height-100);
    ctx.fillStyle = "white";
    ctx.font = "100px Arial";
    ctx.fillText("Game Over", 150, 150);

    var table = getHighScoreTable();
    ctx.fillStyle = "white";
    ctx.fillRect(80, 170, gameWidth-160, height-300);
    ctx.font = "30px Arial";
    ctx.fillStyle = "black";
    ctx.fillText("High Score", 320, 210);
    ctx.font = "15px Arial";
    for(var i=0; i<table.length; i++) {
        var name = table[i].name;
        var score = table[i].score;
        // console.log("name.length: " + name.length + ", score.length: " + score.length);
        var numSpaces = 35 - name.length - score.length;
        if(numSpaces + name.length + score.length != 35) console.log("not 35");
        var str = name+ whiteSpace(numSpaces) + score;
        if(str.length != 36) console.log("str len: " + str.length);
        ctx.fillText(name, 310, 240+i*20);
        ctx.fillText(score, 310+145, 240+i*20);
        // console.log("string len: " + (name+ whiteSpace(35 - name.length - score.length) + score).length);
    }
}
function whiteSpace(num) {
    var sp = " ";
    for(var i=0; i<num ;i++) {
        sp += " ";
    }
    return sp;
}
function drawExit() {
    if(goodThings.length == 0 ) {
        // ctx.fillStyle = "gold";
        // ctx.fillRect(exit.x, exit.y, exit.width, exit.height);
        ctx.drawImage(exitImage,0,0, 28,28,
            exit.x,exit.y-8,28,28);
    }
}

function drawGUI() {
    ctx.font="15px Arial";
    ctx.fillStyle = "black";
    ctx.fillText("Cheat mode: " + (player.cheatmode ? "On" : "Off"), gameWidth+60, width/2);
    ctx.font="20px Arial";
    ctx.fillStyle = "black";
    ctx.fillText("Level " + currentLevel, gameWidth+80, width/2-300);
    ctx.fillText("Bullets: " + player.bullets, gameWidth+70, width/2-150);
    ctx.fillText("Score:",gameWidth+80,width/2-100);
    ctx.strokeRect(gameWidth+60,width/2-90, 100, 30);
    var score = player.score;
    if(score<10) {
        ctx.fillText(score,gameWidth+100,width/2-70);
    } else if (score < 1000) {
        ctx.fillText(score,gameWidth+95,width/2-70);
    } else {
        ctx.fillText(score,gameWidth+90,width/2-70);
    }
    ctx.fillText("Time:",gameWidth+80,width/2-250);
    ctx.fillStyle = "black";
    ctx.strokeRect(gameWidth+60,width/2-240, 100, 30);



    elapsedTime = parseInt((new Date().getTime()-startTime)/1000, 10);
    remainingTime = givenTime - elapsedTime;
    if (!isGameOver && remainingTime >= 0) {
        // hack for smooth linear transition of RGB values from green to red
        var percent = 1 - remainingTime/givenTime;
        var R = Math.round((255 * 100*percent) / 100);
        var G = Math.round((255 * (100 - 100*percent)) / 100) ;
        var B = 0;
        // timer bar
        ctx.fillStyle = "rgb(" + R + "," + G + "," + B + ")";
        ctx.fillRect(gameWidth+60+1, width/2-239, 98*(remainingTime/givenTime), 28);
        // timer text
        ctx.fillStyle = "black";
        ctx.fillText(remainingTime,gameWidth+100,width/2-219);
    } 
    if(remainingTime <= 0 && player.cheatmode == false) {
        isGameOver = true;
    }
}

function updateMonstersCoordinates() {
    for (var i=0; i<monsters.length; i++) {
        var monster = monsters[i];
        if(monster.direction=="E") {
            monster.x += monster.velX;
        } else {
            monster.x -= monster.velX;
        }

        if(monster.x > monster.rightBoundary) {
            monster.direction = "W";
        }
        if(monster.x < monster.leftBoundary) {
            monster.direction = "E";
        }
        // update sprite sheet coords
        if (count%10==0) monster.currSpriteX += CHAR_WIDTH;
        if(monster.direction=="E") {
            monster.currSpriteY = IMAGE_START_EAST_Y;
        } else {
            monster.currSpriteY = IMAGE_START_WEST_Y;
        }
        if (monster.currSpriteX >= SPRITE_WIDTH) {
            monster.currSpriteX = 0;
        }
    }
}

function Projectile(x, y, direction, size, color, speed) {
    this.x = x;
    this.y = y;
    this.direction = facing;
    this.size = size;
    this.color = color;
    this.speed = speed;
    this.currSpriteY = facing == "E" ? 0 : 7;
}

function updateProjectiles() {
    for (var key in projectiles) {
        //console.log("inside update projectiles");
        if(projectiles[key].direction == "E")
            projectiles[key].x += 5; 
        else 
            projectiles[key].x -= 5; 
        if (projectiles[key].x > canvas.width || projectiles[key].x < 0) {
            // console.log("removed projectile");
            projectiles.splice(key, 1);
        }
    }
}

function drawSquare(x, y, size, color) {
    // console.log("inside draw square");
    ctx.fillStyle = "red";
    ctx.fillRect(Math.round(x), Math.round(y), size, size);
}
function clearCanvas() {
    ctx.clearRect(0, 0, width, height);
}
function updatePlayerVelocity() {
    player.velX *= friction;
    player.velY += gravity;
}
function handleProjectileMonsterCollision() {
    for (var key in projectiles) {
        for(var i=0; i<monsters.length; i++) {
            var monster = monsters[i];
            if (projectiles[key] == null) {
                // console.log("projectile null");
                continue;
            }
            if (
                projectiles[key].x < monster.x + monster.width &&
                projectiles[key].x + 10 > monster.x &&
                projectiles[key].y < monster.y + monster.height &&
                projectiles[key].y + 10 > monster.y
            ) {
                // item.x = Math.random() * canvas.width;
                // item.y = height - 10;
                hitSnd.play();
                monsters.splice(i, 1);
                projectiles.splice(key, 1);
                player.score += 10;
            }
        }
    }
}

function handleProjectileFlyingMonsterCollision() {
    for (var key in projectiles) {
        if (projectiles[key] == null) {
            // console.log("projectile null");
            continue;
        }
        if (
            flyingMonster != null &&
            projectiles[key].x < flyingMonster.x + flyingMonster.width &&
            projectiles[key].x + 10 > flyingMonster.x &&
            projectiles[key].y < flyingMonster.y + flyingMonster.height &&
            projectiles[key].y + 10 > flyingMonster.y
        ) {
            flyingMonster = null;
            hitSnd.play();
            projectiles.splice(key, 1);
            player.score += 50;
        }

    }
}

function handleProjectilePlatformCollision() {
    for(var key in projectiles) {
        for(var i=0; i<boxes.length; i++) {
            // var isCollided = colCheck(projectiles[key], boxes[i]);
            if(projectiles[key] == null) continue;
            if (
                projectiles[key].x < boxes[i].x + boxes[i].width &&
                projectiles[key].x + 10 > boxes[i].x &&
                projectiles[key].y < boxes[i].y + boxes[i].height &&
                projectiles[key].y + 10 > boxes[i].y
            ) {
                projectiles.splice(key, 1);
            }
        }
    }
}

function handlePlayerMonsterCollision() {
    // handling collision with monster
    for(var i=0; i<monsters.length; i++) {
        var monster = monsters[i];
        if (
            player.x < monster.x + monster.width &&
            player.x + monster.width > monster.x &&
            player.y < monster.y + monster.height &&
            player.y + player.height > monster.y &&
            player.cheatmode == false
        ) {
            // item.x = Math.random() * canvas.width;
            // item.y = height - 10;
            isGameOver = true;
            // console.log("you are dead");
        }
    }
}

function handlePlayerGoodThingCollision() {
    // handling collision with monster
    for(var i=0; i<goodThings.length; i++) {
        var g = goodThings[i];
        if (
            player.x < g.x + g.width &&
            player.x + g.width > g.x &&
            player.y < g.y + g.height &&
            player.y + player.height >g.y
        ) {
            // item.x = Math.random() * canvas.width;
            // item.y = height - 10;
            goodThings.splice(i, 1);
            player.score += 10;
        }
    }
}
function handlePlayerPortalCollision() {
    if (
        player.x < portalStart.x + portalStart.width &&
        player.x + portalStart.width > portalStart.x &&
        player.y < portalStart.y + portalStart.height &&
        player.y + player.height >portalStart.y 
    ) {
        // console.log("you won this level");
        player.x = portalEnd.x;
        player.y = portalEnd.y - 20;
    }
}

function handlePlayerExitCollision() {
    if (
        player.x < exit.x + exit.width &&
        player.x + exit.width > exit.x &&
        player.y < exit.y + exit.height &&
        player.y + player.height >exit.y &&
        goodThings.length == 0
    ) {
        // item.x = Math.random() * canvas.width;
        // item.y = height - 10;
        isNextLevel = true;
        levelupSnd.play();
        // console.log("you won this level");
    }
}


function drawGoodThings() {
    for(var i=0; i<goodThings.length; i++) {
        var g = goodThings[i];
        // ctx.fillStyle = "green";
        // ctx.fillRect(g.x, g.y, g.width, g.height);
        if(count%10==0) {
            g.currSpriteX += COIN_WIDTH;
            if (g.currSpriteX >= COIN_SPRITE_WIDTH) {

                g.currSpriteX = 0;
            }
        }
        ctx.drawImage(goodThingImage,g.currSpriteX, 0, COIN_WIDTH,COIN_HEIGHT,
            g.x,g.y,COIN_WIDTH,COIN_HEIGHT);
    }
}

function drawMonsters() {

    for(var i=0; i<monsters.length; i++) {
        var monster = monsters[i];
        // ctx.fillStyle = "red";
        // ctx.fillRect(monster.x, monster.y, monster.width, monster.height);
        ctx.drawImage(monsterImage,monster.currSpriteX,monster.currSpriteY, CHAR_WIDTH,CHAR_HEIGHT,
                    monster.x-4,monster.y-11,CHAR_WIDTH,CHAR_HEIGHT);
    }
}
function updatePlayerCoordinates() {
    if(player.grounded){
         player.velY = 0;
    }
    player.x += player.velX;
    player.y += player.velY;
}
// var m = 0;
function drawPlatforms() {
    ctx.fillStyle = "black";
    ctx.beginPath();
    
    player.grounded = false;
    for (var i = 0; i < boxes.length; i++) {
        // ctx.rect(boxes[i].x, boxes[i].y, boxes[i].width, boxes[i].height);
        ctx.drawImage(tileImage,81,20, 19,20,
                    boxes[i].x, boxes[i].y, boxes[i].width, boxes[i].height);

        // update y coord, platform moves if its velY is non-zero
        boxes[i].y -= boxes[i].velY;

        // toggle movable platform direction
        if (Math.round(Math.abs(boxes[i].origY - boxes[i].y)) == 20) {
            // console.log("changed movable platform direction");
            boxes[i].velY = -(boxes[i].velY);
        }
        
        var dir = colCheck(player, boxes[i]);

        if (dir === "l" || dir === "r") {
            player.velX = 0;
            player.jumping = false;
        } else if (dir === "b") {
            player.grounded = true;
            player.jumping = false;
        } else if (dir === "t") {
            player.velY *= -1;
        }

    }
    // m+=0.01;
    // ctx.drawImage(tileImage,81,20, 19,20,
    //         gameWidth/2, m, 40, 20);
    // ctx.fill();
}
function drawPlayer() {
    // draw a small red box, which will eventually become our player.
    // ctx.fillStyle = "rgba(255, 0, 0, 0.5)";
    // ctx.fillRect(player.x, player.y, CHAR_WIDTH, CHAR_HEIGHT);
    if(!isGameOver) {
        ctx.drawImage(charImage,currX,currY, CHAR_WIDTH,CHAR_HEIGHT,
                    player.x-10,player.y-23,CHAR_WIDTH,CHAR_HEIGHT);
    }
}
function drawProjectiles() {
    for (var key in projectiles) {
        // drawSquare(projectiles[key].x, projectiles[key].y, projectiles[key].size, projectiles[key].color);
        ctx.drawImage(projectileImage,0, projectiles[key].currSpriteY, PROJECTILE_WIDTH, PROJECTILE_HEIGHT,
                    projectiles[key].x,projectiles[key].y,PROJECTILE_WIDTH,PROJECTILE_HEIGHT);
    }
}
function updateSpriteSheetCoordinates() {
    if(facing=="E") {
        currY = IMAGE_START_EAST_Y;
    } else {
        currY = IMAGE_START_WEST_Y;
    }
    if (currX >= SPRITE_WIDTH) {
        currX = 0;
    }
}
function executeCommands() {
    if (keys[38] || keys[87]) {
        // up arrow or space
        if (!player.jumping && player.grounded) {
            player.jumping = true;
            player.grounded = false;
            player.velY = -player.speed * 2;
        }
    }
    if (keys[39] || keys[68]) {
        // right arrow
        if (player.velX < player.speed) {
            player.velX++;
            if (count%2==0)
                currX += CHAR_WIDTH;
            facing = "E";
        }
    }
    if (keys[37] || keys[65]) {
        // left arrow
        if (player.velX > -player.speed) {
            player.velX--;
            if (count%2==0)
                currX += CHAR_WIDTH;
            facing = "W";
        }
    }
    // button z
    // TODO change to spacebar
    if(keys[32]) {
        if(Date.now() - player.projectileTimer > player.shootDelay && (player.bullets > 0 || player.cheatmode == true) ) {
            projectiles.push(
                new Projectile(
                    player.x + player.width/2,
                    player.y-10,
                    facing,
                    10,
                    '#0f0',
                    1000
                    )
                );
            shootSnd.play();
            player.bullets = player.bullets > 0 ? player.bullets - 1 : 0;
        }
        player.projectileTimer = Date.now();
    }
    // button c: cheat mode on
    if(keys[67]) {
        player.cheatmode = true;
    }

    // button v: cheat mode off
    if (keys[86]) {
        player.cheatmode = false;
    }
}
function colCheck(shapeA, shapeB) {
    // get the vectors to check against
    var vX = (shapeA.x + (shapeA.width / 2)) - (shapeB.x + (shapeB.width / 2)),
        vY = (shapeA.y + (shapeA.height / 2)) - (shapeB.y + (shapeB.height / 2)),
        // add the half widths and half heights of the objects
        hWidths = (shapeA.width / 2) + (shapeB.width / 2),
        hHeights = (shapeA.height / 2) + (shapeB.height / 2),
        colDir = null;

    // if the x and y vector are less than the half width or half height, they we must be inside the object, causing a collision
    if (Math.abs(vX) < hWidths && Math.abs(vY) < hHeights) {
        // figures out on which side we are colliding (top, bottom, left, or right)
        var oX = hWidths - Math.abs(vX),
            oY = hHeights - Math.abs(vY);
        if (oX >= oY) {
            if (vY > 0) {
                colDir = "t";
                shapeA.y += oY;
            } else {
                colDir = "b";
                shapeA.y -= oY;
            }
        } else {
            if (vX > 0) {
                colDir = "l";
                shapeA.x += oX;
            } else {
                colDir = "r";
                shapeA.x -= oX;
            }
        }
    }
    return colDir;
}


window.addEventListener("load", function(){
  update();
});

document.body.addEventListener("keydown", function(e) {
    keys[e.keyCode] = true;
});
 
document.body.addEventListener("keyup", function(e) {
    keys[e.keyCode] = false;
});
</script>
</body>
</html>
