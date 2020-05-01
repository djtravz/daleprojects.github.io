---
permalink: /breakoutGame/
layout: redirects
---
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width">
    <title>Breakout Game - David Travers</title>
    <style>
        canvas {
            border: 1px solid #2b2b2b;
            background-color: #000000;
        }

        body {
            background-color: #292929;
        }

        p {
            color: #FFFFFF
        }
    </style>
</head>

<body style="margin:0;">
    <canvas id="ctv" width="50" height="50"></canvas>
    <script>
        /*TODO:
        Add comments to the whole thing
        Sound effects?
        Set it to a random value between 40 and 50 or something
        Make the power ups want to spawn at the bottom - distrubution upwards, 60%,20%,10%,5%,3%,2%
        play again/start button? maybe too much work
        */
        // Canvas declaration

        var cnv = document.getElementById("ctv"); // gets the canvas from html to a variable here in js

        cnv.width = window.innerWidth - 2; // sets the canvas width to be the width of the window size, -2 is so no scroll bars appear
        cnv.height = window.innerHeight - 6; // same thing as above, just with height

        var ctx = cnv.getContext("2d"); // gets the canvas context, so things can actually be done with it

        ctx.fillStyle = "#FFFFFF"; // just temporary because every fill command will have it there anyway
        ctx.font = '30px Arial'; // I dont use any text, but its there just in case


        // Game variable declaration
        var blocks = {
            all: [], // declares temporary variables
            size: 100,
            w: 0,
            h: 0,
            colors: ["#000000", "#FFFFFF", "#ff3636", "#19e8ff", "#ffb700", "#36ff50", "#962eff"], // the colors that the blocks can be
            draw: function() { // draws all the blocks to the screen
                //console.log("Drawing blocks")
                clear(); // clears the screen 
                for (var i = 0; i < this.all.length; i++) { // for every row
                    for (var j = 0; j < this.all[i].length; j++) { // for every block in row
                        if (this.all[i][j] > 0) { // if the block is not hit
                            ctx.fillStyle = this.colors[this.all[i][j]]; // set the fill color to the corresponding color
                            brick(j * this.size, i * this.size); // draws the block
                        }
                    }
                    //console.log("Drawing ",j*blocks.size,",",i*blocks.size);
                }
                //console.log("Next row");
            },
            addSpecial: function() { // adds a special block
                var n = randomInt(this.all.length - 1); // chooses a random row
                var m = randomInt(this.all[0].length - 1) // chhooses a random block in row
                if (this.all[n][m] == 1) { // if the block chosen is a regular block
                    this.all[n][m] = randomInt(4) + 2; // choose a special block type
                }
                //this.all[randomInt(this.all.length)][randomInt(this.all[0].length)] = randomInt(4) + 2;
            },
            hit: function(i, j) { // when a block is hit
                // console.log("Hit block " + i + "," + j + " with type " + this.all[i][j]);
                switch (this.all[i][j]) { // using the value of the location to determine its type
                    case 0: // already hit
                        break;
                    case 1: // normal block
                        break;
                    case 2: //extra token
                        addTok();
                        break;
                    case 3: // bigger paddle
                        padLength *= 1.1;
                        break;
                    case 4: // explosion
                        for (var o = -3; o <= 3; o++) {
                            this.hit(i + o, j);
                            this.hit(i, j + o);
                        }
                        for (var o = -2; o <= 2; o++) {
                            this.hit(i + o, j + 1);
                            this.hit(i + o, j - 1);
                        }
                        for (var o = -1; o <= 1; o++) {
                            this.hit(i + o, j + 2);
                            this.hit(i + o, j - 2);
                        }
                        break;
                    case 5: // no hit detection
                        detectHit = timer;
                        break;
                    case 6: // remove 10% of blocks
                        for (var i = 0; i < this.all.length; i++) {
                            for (var j = 0; j < this.all[i].length; j++) {
                                if (randomInt(50) == 5) {
                                    this.all[i][j] = false;
                                }
                            }
                        }
                        break;
                }
                this.all[i][j] = 0; // sets the block to hit
            }
        }

        function token(X, Y, volX, volY) { // constructor for a new token
            this.X = X; // sets the tokens values to the ones provided
            this.Y = Y;
            this.volX = volX;
            this.volY = volY;
            this.info = function() { // returns information about the tokens location and velocity for debugging
                return "Pos: " + this.X + "," + this.Y + "\nVelocity: " + this.volX + "," + this.volY;
            }
        }

        var detectHit = true; // declaring varibale for the special block that disables it
        var timer = 0; //declares a timer that will be used throughout the entire game

        var padX = Math.round(0.5 * cnv.width); // center of the paddle defined here so it is the middle
        var padLength = Math.round(0.1 * cnv.width); // length of the paddle is 10% of the total width (but this is going to be doubled later on)

        var toks = []; //makes a blank array for any more tokens that could be created


        // Beginning game
        addTok(); // adds the first token to the array
        init(); // begins initialization protocol
        blocks.draw(); // draws the blocks for the first time
        paddleUpdate(); // updates the paddle for the first time
        setInterval(tick, 10); // begins endless loop for every 10 ms

        function tick() { // runs every loop
            timer++; // increments the timer
            for (const toke of toks) { // loops through every token in the array
                move(toke); // moves that token
            }
            if ((timer % 100) == 0) { // every second
                blocks.addSpecial() // a new special block is added
            }
            if (timer > detectHit + 1000) { // after 10 seconds of the hit detection being turned off, the 
                detectHit = true; // afterward it started bonking again
            }
        }

        function paddleUpdate(mousePos) {
            if (mousePos != null) { // if the mouse position is supplied
                padX = mousePos.x; // update the middle to the x position of mouse 
                ctx.clearRect(0, Math.round(cnv.height * 0.95) - 1, cnv.width, 8); // clears the space for the paddle to update
            }
            ctx.fillStyle = "#0000ff"; // sets the color to blue
            ctx.fillRect(padX - padLength, Math.round(cnv.height * 0.95), padLength * 2, 5); // draws the paddle
        }

        function specialCheck(i, j, type) {
            specialCheck: {
                for (const blk of sBlocks) { // loops through every single special block
                    if (blk.iLoc == i && blk.jLoc == j) { // sees if the number supplied is the postition of a special block
                        if (type == 'hit') { // if it was a block that was hit
                            blk.hit(); // trigger that special block to be hit
                            break specialCheck; // breaks the loop if found to save on lag
                        } else if (type = 'color') { // if it was a block that is being drawn
                            return blk.color; // returns the color that is tied to the type of the block
                            break specialCheck; // breaks the loop if found to save on lag
                        }
                    }
                }
                return false; // no special block in that position
            }
        }

        function move(tok) {
            //console.log("Movement");
            ctx.clearRect(tok.X - 2, tok.Y - 2, 4, 4); // clears the previous position
            if (randomInt(5) != 1) { // 0,2,3,4,5
                tok.X += tok.volX; // moves the block in x position
            }
            if (randomInt(5) != 1) { // 0,2,3,4,5
                tok.Y += tok.volY;; // moves the block in y position
            }
            ctx.fillStyle = "#FF0000"; // sets the color to red
            ctx.fillRect(tok.X - 2, tok.Y - 2, 4, 4); // draws the square
            //console.log(tok.info());
            //ctx.fillStyle = "#FFFFFF";
            checkBonk(tok); // triggers the trajectory checking
        }

        function addTok() { // adds a new token at the center to the array
            toks.push(new token(Math.round(cnv.width * 0.5), Math.round(cnv.height * 0.9), 1, -1)); // constructs a new token at the center
        }

        function init() { // initilizes the canvas size and block size
            //console.log("Initializing...")

            var w = cnv.width; // sets the temporary width variable
            while (isPrime(w)) { // makes sure that it can actually get a number that can be calculated
                w--; // decriments until it is no longer prime
            }


            cnv.width = w; // updates the canvas width to the not prime number
            var h = cnv.height; // sets the temporary height value

            //console.log("Canvas size: ", w, ", ", h); // logs the width and height of the canvas

            var min = w; // sets the minimum value to something high
            for (var b = 1; b < w; b++) { // loops over until it gets to the width because theres no way its going to be that many
                mod = w % b; // gets the remainder
                if (mod <= min) { // if the remainder value is lower than the current minimum value
                    min = mod; // set the remainder value to the minimum
                    blocks.w = b; // sets the new value to the number of blocks on the width
                }
                blocks.size = w / blocks.w; // sets the size to the size of the blocks to how many it needs to fill the screen
                if (blocks.size < (w / 50)) { // if the width takes up more than half the screen
                    break; // break the loop
                }
            }

            blocks.h = Math.round((h * 0.7) / blocks.size); // sets the height value bassed on the size of the blocks and 70% of the screen

            //console.log(blocks.w);
            //console.log(blocks.h);
            //console.log(blocks.size);
            for (var j = 0; j < blocks.h; j++) { // for the number of blocks in the height
                blocks.all.push([]); // push a blank array to the main array
                for (var i = 0; i < blocks.w; i++) { // for the number of blocks in the width
                    blocks.all[j][i] = 1; // push a 1 to the new blank array for every width
                }
            }
            //console.log(blocks.all);
        }

        function checkBonk(tok) { // checks the collision

            //console.log("Checking collision");
            if (tok.Y < 1) { // if it bounces off the top
                tok.volY = 1; // set the trajectory down
            }
            if (tok.Y >= Math.round(cnv.height * 0.95) && tok.Y <= (Math.round(cnv.height * 0.95) + 7) && tok.X >= padX - padLength && tok.X <= padX + (padLength * 2)) { // if it hits the paddle
                tok.volY = -1; // set the trajectory up
            }
            if (tok.Y > cnv.height + 10) { // if it goes off the bottom of the screen
                tok.volX = 0; // stop all movement
                tok.volY = 0;
            }
            if (tok.X < 1) { // if it hits the left side
                tok.volX = 1; // set the trajectory to the right
            }
            if (tok.X > cnv.width) { // if it hits the right side
                tok.volX = -1 // set the trajectory to the left
            }
            bonkCheck: { // loop that can be broken
                for (var i = 0; i < blocks.all.length; i++) { // for every row
                    for (var j = 0; j < blocks.all[i].length; j++) { // for every block in row
                        if (blocks.all[i][j] > 0) { // if the block is still there
                            if (bonkTrue(i, j, 0, tok)) { // and it hit from the bottom or top
                                blocks.hit(i, j); // trigger the block to be hit
                                if (detectHit == true) { // if the nohit block has not been triggered
                                    bonk(0, tok); // trigger a rebound
                                }
                                blocks.draw(); // draw all the blocks
                                break bonkCheck; // breaks the rest of the loop to save on lag
                            } else if (bonkTrue(i, j, 1, tok)) { // if the block is hit from the sides
                                blocks.hit(i, j); // same as above
                                if (detectHit == true) {
                                    bonk(1, tok);
                                }
                                blocks.draw();
                                break bonkCheck;
                            }
                        }
                    }
                }
            }
        }

        function bonk(surf, tok) {
            if (surf == 0) { // if the surface is a horizontal one
                tok.volY *= -1; // inverse the Y velocity
                //console.log("Horizontal surface bonk");
            } else if (surf == 1) { // if the surface is a virtical one
                tok.volX *= -1; // inverse the X velocity
                //console.log("Virtical surface bonk");
            }
        }

        function bonkTrue(i, j, surf, tok) {
            if (surf == 0) {
                // if it is touching the bottom or top edge and it is between the two edges, return true
                return (tok.Y == (i * blocks.size) || tok.Y == ((i + 1) * blocks.size)) && (tok.X >= (j * blocks.size) && tok.X <= ((j + 1) * blocks.size));
            } else if (surf == 1) {
                // if it is touching the side edge sand it is between the top and bottom, return true
                return (tok.X == (j * blocks.size) || tok.X == ((j + 1) * blocks.size)) && (tok.Y >= (i * blocks.size) && tok.Y <= ((i + 1) * blocks.size));
            }
        }

        function brick(x, y) { // makes a block for the blocks.draw
            ctx.fillRect(x + 1, y + 1, blocks.size - 1, blocks.size - 1);
        }

        function clear() { // clears the entire canvas
            ctx.clearRect(0, 0, cnv.width, cnv.height);
            paddleUpdate();
        }

        // https://medium.com/@sarahdherr/prime-number-algorithm-in-js-f9fb2439c7ae
        function isPrime(num) {
            if (num <= 1) {
                return true
            } else if (num <= 3) {
                return true
            } else if (num % 2 === 0 || num % 3 === 0) {
                return false
            }

            let i = 5
            while (i * i <= num) {
                if (num % i === 0 || num % (i + 2) === 0) {
                    return false
                }
                i += 6
            }
            return true
        }

        // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random
        function randomInt(max) {
            return Math.floor(Math.random() * Math.floor(max + 1));
        }

        // https://www.html5canvastutorials.com/advanced/html5-canvas-mouse-coordinates/
        function getMousePos(canvas, evt) {
            var rect = canvas.getBoundingClientRect();
            return {
                x: evt.clientX - rect.left,
                y: evt.clientY - rect.top
            };
        }
        cnv.addEventListener('mousemove', function(evt) {
            var mousePos = getMousePos(cnv, evt);
            paddleUpdate(mousePos);
        }, false);
    </script>
</body>

</html>
