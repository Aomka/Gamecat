@{
    
}

<h1>@ViewData["Title"]</h1>

<style>
    canvas {
        border: 1px solid black;
        display: block;
        margin: auto;
        background-color: lightblue;
    }

    #treasureBox {
        display: none;
        position: absolute;
        background-color: gold;
        width: 100px;
        height: 50px;
        text-align: center;
        line-height: 50px;
        font-weight: bold;
        border: 2px solid black;
        z-index: 1;
    }
</style>

<canvas id="gameCanvas" width="800" height="400"></canvas>
<div id="treasureBox">ตั๋วสมพรปรารถนา</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    // สร้างตัวละครแมว
    let catImage = new Image();
    catImage.src = 'https://community.gamedev.tv/uploads/db2322/original/3X/a/9/a94a989e070141a536ebf7c8ab52b26cff609935.gif';

    let cat = {
        x: 50,
        y: 200,
        width: 50,
        height: 50,
        speed: 5,
        moveUp: false,
        moveDown: false,
        draw: function () {
            ctx.drawImage(catImage, this.x, this.y, this.width, this.height);
        },
        update: function () {
            if (this.moveUp && this.y > 0) {
                this.y -= this.speed;
            }
            if (this.moveDown && this.y < canvas.height - this.height) {
                this.y += this.speed;
            }
        }
    };
    // สร้างมอนสเตอร์
    let monsters = [];
    function createMonster() {
        let monster = {
            x: canvas.width,
            y: Math.random() * (canvas.height - 50),
            width: 50,
            height: 50,
            speed: 3,
            draw: function () {
                ctx.fillStyle = 'red';
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }
        };
        monsters.push(monster);
    }

    // สร้างกระสุนเพื่อยิงมอนสเตอร์
    let bullets = [];
    function shoot() {
        let bullet = {
            x: cat.x + cat.width,
            y: cat.y + cat.height / 2 - 2.5,
            width: 10,
            height: 5,
            speed: 7,
            draw: function () {
                ctx.fillStyle = 'yellow';
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }
        };
        bullets.push(bullet);
    }

    // ตัวแปรสำหรับชีวิตและการนับจำนวนมอนสเตอร์ที่ยิงได้
    let lives = 3;
    let monstersShot = 0;
    let treasureBoxVisible = false;
    let gameOver = false;

    // ฟังก์ชันอัพเดตสถานะของเกมในทุกเฟรม
    function updateGame() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // อัปเดตและวาดแมว
        cat.update();
        cat.draw();

        // วาดกระสุนและทำการเคลื่อนที่
        bullets.forEach((bullet, index) => {
            bullet.x += bullet.speed;
            bullet.draw();
            if (bullet.x > canvas.width) {
                bullets.splice(index, 1); // ลบกระสุนที่ออกนอกจอ
            }
        });

        // วาดและเคลื่อนที่มอนสเตอร์
        monsters.forEach((monster, index) => {
            monster.x -= monster.speed;
            monster.draw();

            // ตรวจจับการชนระหว่างกระสุนกับมอนสเตอร์
            bullets.forEach((bullet, bulletIndex) => {
                if (bullet.x < monster.x + monster.width &&
                    bullet.x + bullet.width > monster.x &&
                    bullet.y < monster.y + monster.height &&
                    bullet.y + bullet.height > monster.y) {
                    monsters.splice(index, 1); // ลบมอนสเตอร์ที่โดนยิง
                    bullets.splice(bulletIndex, 1); // ลบกระสุนที่ยิงโดน
                    monstersShot++; // เพิ่มจำนวนมอนสเตอร์ที่ยิง
                }
            });

            // ตรวจจับการชนระหว่างแมวกับมอนสเตอร์
            if (cat.x < monster.x + monster.width &&
                cat.x + cat.width > monster.x &&
                cat.y < monster.y + monster.height &&
                cat.y + cat.height > monster.y) {
                lives--; // ลดชีวิต
                monsters.splice(index, 1); // ลบมอนสเตอร์ที่ชน
                if (lives <= 0) {
                    gameOver = true; // กำหนดว่าเกมจบแล้ว
                }
            }

            if (monster.x + monster.width < 0) {
                monsters.splice(index, 1); // ลบมอนสเตอร์ที่วิ่งออกนอกจอ
            }
        });

        // แสดงจำนวนชีวิต
        ctx.fillStyle = 'black';
        ctx.font = '20px Arial';
        ctx.fillText(`Lives: ${'❤️'.repeat(lives)}`, 10, 20);
        ctx.fillText(`Monsters Shot: ${monstersShot}`, 10, 50);

        // แสดงกล่องสมบัติเมื่อยิงมอนสเตอร์ครบ 100 ตัว
        if (monstersShot >= 100 && !treasureBoxVisible) {
            treasureBoxVisible = true;
            document.getElementById('treasureBox').style.display = 'block';
            document.getElementById('treasureBox').style.left = `${canvas.width / 2 - 50}px`;
            document.getElementById('treasureBox').style.top = `${canvas.height / 2 - 25}px`;
        }

        // แสดงข้อความ Game Over
        if (gameOver) {
            ctx.fillStyle = 'red';
            ctx.font = '30px Arial';
            ctx.fillText('Game Over!', canvas.width / 2 - 70, canvas.height / 2);
            ctx.fillText('Click to Restart', canvas.width / 2 - 90, canvas.height / 2 + 40);
            return; // หยุดอัปเดตเกมเมื่อเกมจบ
        }
    }

    // ตรวจจับการกดและปล่อยปุ่มคีย์บอร์ด
    document.addEventListener('keydown', (e) => {
        if (e.code === 'KeyW') {
            cat.moveUp = true;
        }
        if (e.code === 'KeyS') {
            cat.moveDown = true;
        }
    });

    document.addEventListener('keyup', (e) => {
        if (e.code === 'KeyW') {
            cat.moveUp = false;
        }
        if (e.code === 'KeyS') {
            cat.moveDown = false;
        }
    });

    // Loop ของเกมทำงานอย่างต่อเนื่อง
    function gameLoop() {
        updateGame();
        requestAnimationFrame(gameLoop);
    }

    // สร้างมอนสเตอร์ใหม่ทุก ๆ 2 วินาที
    setInterval(createMonster, 2000);

    // ตรวจจับการคลิกเพื่อยิงกระสุน
    canvas.addEventListener('click', () => {
        if (gameOver) {
            // รีเซ็ตค่าเกม
            lives = 3;
            monstersShot = 0;
            gameOver = false;
            monsters = [];
            bullets = [];
            treasureBoxVisible = false;
            document.getElementById('treasureBox').style.display = 'none'; // ซ่อนกล่องสมบัติ
            setInterval(createMonster, 2000); // สร้างมอนสเตอร์ใหม่
        } else {
            shoot(); // ยิงกระสุน
        }
    });

    // เริ่มต้น loop ของเกม
    gameLoop();
</script>
