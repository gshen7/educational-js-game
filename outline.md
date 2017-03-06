##Short Presentation on Javascript Basics

##Coding Demo of Brick Breaker Game
0. show the style sheet, go through the skeleton describing each part
1. start drawing [PROVIDED]
    * draw a ball (in drawBall function) - note: can also draw other shapes (we will later)
    ``` 
    ctx.beginPath();
    ctx.arc(50, 50, 10, 0, Math.PI*2);
    ctx.fillStyle = "#0095DD";
    ctx.fill();
    ctx.closePath();
    ```
    * add a line that calls drawBall in the main draw function
2. add some movement [LIVE CODING]
    * declare some variables for the position of the ball and replace the first parameters of the drawBall arc call
    ```
    var x = canvas.width/2;
    var y = canvas.height-30;
    ```
    * declare some variables for the movement of the ball
    ```
    var dx = 3;
    var dy = -3;
    ```
    * change position whenever you draw
    ``` 
    x += dx;
    y += dy;
    ```
    * try it - you'll see it leave a "trail" - add this to clear the canvas before you draw
    ```
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ```
3. the ball just leaves the screen now, need to bounce it off sides [LIVE CODING]
    * declare a variable for the radius of the ball and replace the third parameter in arc call
    ```
    var ballRadius = 10;
    ```
    * now need some conditional statements - coordinate system starts at top left - start with - put this in draw
    ```
    if(y + dy < 0) {
        dy = -dy;
    }  
    ```
    * that only deals with the top edge, for the bottom edge and left and right edges you want to add :
    ```
    if(y + dy > canvas.height) {
        dy = -dy;
    }  
     if(x + dx < 0) {
        dx = -dx;
    }  
    if(x + dx > canvas.width) {
        dx = -dx;
    } 
    ```
    * test it - half the ball goes off the screen, change by taking into account radius - and also making it less verbose
    ```
    if(x + dx > canvas.width-ballRadius || x + dx < ballRadius) {
    dx = -dx;
    }
    if(y + dy > canvas.height-ballRadius || y + dy < ballRadius) {
        dy = -dy;
    }
    ```
4. paddle [SOME PROVIDED, SOME LIVE]
    * draw a paddle, rectangle instead of circle at the bottom of canvas - same kind of process - declare some vars for size and position, drawPaddle function
    ```
    var paddleHeight = 10;
    var paddleWidth = 75;
    var paddleX = (canvas.width-paddleWidth)/2;

    //.
    //.
    //.
    //PROVIDED
     function drawPaddle() {
        ctx.beginPath();
        ctx.rect(paddleX, canvas.height-paddleHeight, paddleWidth, paddleHeight);
        ctx.fillStyle = "#0095DD";
        ctx.fill();
        ctx.closePath();
    }
    ```
    * control the paddle - declare some vars for input, already added the event listeners and have some functions to use with it - self explanatory key down, key up and mouse move, for the key down and key up, just found the keycodes by googling, for the mouse move, need to first check relative x is in canvas (distance from left edge to cursor in canvas), then position the paddle x at that spot minus half of the paddle with so its in the middle (because paddle prints left edge at X)
    ```
    var rightPressed = false;
    var leftPressed = false;

    //[PROVIDED] 
    document.addEventListener("keydown", keyDownHandler, false);
    document.addEventListener("keyup", keyUpHandler, false);
    document.addEventListener("mousemove", mouseMoveHandler, false);

    function keyDownHandler(e) {
        if(e.keyCode == 39) {
            rightPressed = true;
        }
        else if(e.keyCode == 37) {
            leftPressed = true;
        }
    }
    function keyUpHandler(e) {
        if(e.keyCode == 39) {
            rightPressed = false;
        }
        else if(e.keyCode == 37) {
            leftPressed = false;
        }
    }
    function mouseMoveHandler(e) {
        var relativeX = e.clientX - canvas.offsetLeft;
        if(relativeX > 0 && relativeX < canvas.width) {
            paddleX = relativeX - paddleWidth/2;
        }
    }
    ```
    * this only detects the input, now we have to do something with it - in the draw function
    ```
    if(rightPressed && paddleX < canvas.width-paddleWidth) {
        paddleX += 7;
    }
    else if(leftPressed && paddleX > 0) {
        paddleX -= 7;
    }
    ```
    * lastly, call drawPaddle from main draw function
5. right now you can miss the ball but it'll just bounce back up - change that [LIVE]
    * when the ball hits the bottom (which we detected before) dont just change direction, do something else 
    ```
    if(y + dy < ballRadius) {
    dy = -dy;
    } else if(y + dy > canvas.height-ballRadius) {
        alert("GAME OVER");
        document.location.reload();
    }
    ```
    * we also haven't let the paddle actually hit the ball - thats just another conditional nested inside the other
    ``` 
    if(y + dy < ballRadius) {
        dy = -dy;
    } else if(y + dy > canvas.height-ballRadius) {
        if(x > paddleX && x < paddleX + paddleWidth) {
            dy = -dy;
        }
        else {
            alert("GAME OVER");
            document.location.reload();
        }
    }
    ```
6. bricks to break [SOME PROVIDED, SOME LIVE]
    * a bunch of variables to declare then instead of making each break individually we use a loop structure and a 2d array
    ```
    var brickRowCount = 3;
    var brickColumnCount = 5;
    var brickWidth = 75;
    var brickHeight = 20;
    var brickPadding = 10;
    var brickOffsetTop = 30;
    var brickOffsetLeft = 30;
    
    var bricks = [];
    for(c=0; c<brickColumnCount; c++) {
        bricks[c] = [];
        for(r=0; r<brickRowCount; r++) {
            bricks[c][r] = { x: 0, y: 0};
        }
    }
    ```
    * [PROVIDED] the actual drawing function involves a loop too
    ```
    function drawBricks() {
        for(c=0; c<brickColumnCount; c++) {
            for(r=0; r<brickRowCount; r++) {
                var brickX = (c*(brickWidth+brickPadding))+brickOffsetLeft;
                var brickY = (r*(brickHeight+brickPadding))+brickOffsetTop;
                bricks[c][r].x = brickX;
                bricks[c][r].y = brickY;
                ctx.beginPath();
                ctx.rect(brickX, brickY, brickWidth, brickHeight);
                ctx.fillStyle = "#0095DD";
                ctx.fill();
                ctx.closePath();
            }
        }
    }
    ```
    * add drawbircks call in main draw function
7. [LIVE] actually breaking the bricks
    * collision detection function - collision logic is true if ball x greater than brick x and ball x less than brick x plus brick width and ball y greater than brick y and ball y less than brick y plus height of brick
    ```
    function collisionDetection() {
        for(c=0; c<brickColumnCount; c++) {
            for(r=0; r<brickRowCount; r++) {
            var b = bricks[c][r];
                if(x > b.x && x < b.x+brickWidth && y > b.y && y < b.y+brickHeight) {
                    dy = -dy;
                }
            }
        }
    }
    ```
    * making them disappear - there are better ways to do this but this is easiest - add a parameter to bricks - status and change so they're only drawn/compared to for collision if status is 1 and when colliding, status changes to 0
    ```
    bricks[c][r] = { x: 0, y: 0, status: 1 }; // replace in bricks initialization

    //.
    //.
    //.

    if(bricks[c][r].status == 1) {
        //all the brick drawing
    }

    //.
    //.
    //.

    if(b.status == 1) { //contain in collision detection
        if(x > b.x && x < b.x+brickWidth && y > b.y && y < b.y+brickHeight) {
            dy = -dy;
            b.status = 0; //add
        }
    }
    ```
    * also add call to collision detection 
8. score [LIVE and PROVIDED]
    * need a variable for score and lives and draw functions for both to display
    ```
    var score = 0;
    var lives = 3;

    //.
    //.
    //.

    //PROVIDED
    function drawScore() {
        ctx.font = "16px Arial";
        ctx.fillStyle = "#0095DD";
        ctx.fillText("Score: "+score, 8, 20);
    }
    function drawLives() {
        ctx.font = "16px Arial";
        ctx.fillStyle = "#0095DD";
        ctx.fillText("Lives: "+lives, canvas.width-65, 20);
    }
    ```
    * change score each time a brick is collided with and display win message after all bricks destroyed (check score every time in collision detection)
    ```
    if(x > b.x && x < b.x+brickWidth && y > b.y && y < b.y+brickHeight) {
        dy = -dy;
        b.status = 0; 
        score++; //add
        if(score == brickRowCount*brickColumnCount) {
            alert("YOU WIN, CONGRATULATIONS!");
            document.location.reload();
        }
    }
    ```
    * change the game over logic to instead take a life until 0 then gameover (in draw)
    ```
    lives--;
    if(!lives) {
        alert("GAME OVER");
        document.location.reload();
    }
    else {
        x = canvas.width/2;
        y = canvas.height-30;
        dx = 3;
        dy = -3;
        paddleX = (canvas.width-paddleWidth)/2;
    }
    ```
    * add draw score, lives call in draw 

##Depending on Time, Make Pong or More Levels?

