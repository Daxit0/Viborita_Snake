# Viborita_Snake
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Juego de la Viborita</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f2f2f2;
        }
        canvas {
            border: 1px solid #333;
        }
    </style>
</head>
<body>
    <canvas id="snakeCanvas" width="400" height="400"></canvas>
    <script>
        document.addEventListener("DOMContentLoaded", function() {
            const canvas = document.getElementById("snakeCanvas");
            const ctx = canvas.getContext("2d");

            const blockSize = 20;
            const widthInBlocks = canvas.width / blockSize;
            const heightInBlocks = canvas.height / blockSize;

            let score = 0;
            let snake;
            let apple;

            function init() {
                snake = new Snake([[6, 4], [5, 4], [4, 4]], "right");
                apple = new Apple([10, 10]);
                score = 0;
                drawScore();
                snake.draw();
                apple.draw();
                gameLoop();
            }

            function drawScore() {
                ctx.font = "20px Arial";
                ctx.fillStyle = "black";
                ctx.textAlign = "left";
                ctx.fillText("Puntuación: " + score, blockSize, blockSize);
            }

            function gameLoop() {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                drawScore();
                snake.move();
                snake.draw();
                apple.draw();
                if (snake.checkCollision()) {
                    gameOver();
                } else {
                    if (snake.isEatingApple(apple)) {
                        score++;
                        snake.ateApple = true;
                        do {
                            apple.setNewPosition();
                        } while (apple.isOnSnake(snake));
                    }
                    setTimeout(gameLoop, 100);
                }
            }

            function gameOver() {
                ctx.font = "60px Arial";
                ctx.fillStyle = "black";
                ctx.textAlign = "center";
                ctx.textBaseline = "middle";
                ctx.fillText("Fin del Juego", canvas.width / 2, canvas.height / 2);
            }

            function Snake(body, direction) {
                this.body = body;
                this.direction = direction;
                this.ateApple = false;

                this.draw = function() {
                    ctx.fillStyle = "green";
                    for (let i = 0; i < this.body.length; i++) {
                        ctx.fillRect(this.body[i][0] * blockSize, this.body[i][1] * blockSize, blockSize, blockSize);
                    }
                };

                this.move = function() {
                    let nextPosition = this.body[0].slice();
                    switch (this.direction) {
                        case "left":
                            nextPosition[0] -= 1;
                            break;
                        case "right":
                            nextPosition[0] += 1;
                            break;
                        case "up":
                            nextPosition[1] -= 1;
                            break;
                        case "down":
                            nextPosition[1] += 1;
                            break;
                        default:
                            throw("Invalid Direction");
                    }
                    this.body.unshift(nextPosition);
                    if (!this.ateApple) {
                        this.body.pop();
                    } else {
                        this.ateApple = false;
                    }
                };

                this.setDirection = function(newDirection) {
                    let allowedDirections;
                    switch (this.direction) {
                        case "left":
                        case "right":
                            allowedDirections = ["up", "down"];
                            break;
                        case "up":
                        case "down":
                            allowedDirections = ["left", "right"];
                            break;
                        default:
                            throw("Invalid Direction");
                    }
                    if (allowedDirections.indexOf(newDirection) > -1) {
                        this.direction = newDirection;
                    }
                };

                this.checkCollision = function() {
                    let wallCollision = false;
                    let snakeCollision = false;
                    const head = this.body[0];
                    const rest = this.body.slice(1);
                    const snakeX = head[0];
                    const snakeY = head[1];
                    const minX = 0;
                    const minY = 0;
                    const maxX = widthInBlocks - 1;
                    const maxY = heightInBlocks - 1;
                    const outsideHorizontalBounds = snakeX < minX || snakeX > maxX;
                    const outsideVerticalBounds = snakeY < minY || snakeY > maxY;

                    if (outsideHorizontalBounds || outsideVerticalBounds) {
                        wallCollision = true;
                    }

                    for (let i = 0; i < rest.length; i++) {
                        if (snakeX === rest[i][0] && snakeY === rest[i][1]) {
                            snakeCollision = true;
                        }
                    }

                    return wallCollision || snakeCollision;
                };

                this.isEatingApple = function(appleToEat) {
                    const head = this.body[0];
                    if (head[0] === appleToEat.position[0] && head[1] === appleToEat.position[1]) {
                        return true;
                    } else {
                        return false;
                    }
                };
            }

            function Apple(position) {
                this.position = position;

                this.draw = function() {
                    ctx.fillStyle = "red";
                    ctx.beginPath();
                    const radius = blockSize / 2;
                    const x = this.position[0] * blockSize + radius;
                    const y = this.position[1] * blockSize + radius;
                    ctx.arc(x, y, radius, 0, Math.PI * 2, true);
                    ctx.fill();
                };

                this.setNewPosition = function() {
                    const newX = Math.round(Math.random() * (widthInBlocks - 1));
                    const newY = Math.round(Math.random() * (heightInBlocks - 1));
                    this.position = [newX, newY];
                };

                this.isOnSnake = function(snakeToCheck) {
                    let isOnSnake = false;
                    for (let i = 0; i < snakeToCheck.body.length; i++) {
                        if (this.position[0] === snakeToCheck.body[i][0] && this.position[1] === snakeToCheck.body[i][1]) {
                            isOnSnake = true;
                        }
                    }
                    return isOnSnake;
                };
            }

            document.addEventListener("keydown", function(event) {
                const newDirection = getNewDirection(event.keyCode);
                if (newDirection) {
                    snake.setDirection(newDirection);
                }
            });

            function getNewDirection(keyCode) {
                switch (keyCode) {
                    case 37:
                        return "left";
                    case 38:
                        return "up";
                    case 39:
                        return "right";
                    case 40:
                        return "down";
                    default:
                        return null;
                }
            }

            init();
        });
    </script>
</body>
</html>
