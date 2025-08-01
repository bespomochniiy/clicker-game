# clicker-game
My first clicker
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Кликер с фигурами, летящими через кнопку</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      margin: 0;
      padding: 50px;
      color: #000;
      height: 100vh;
      overflow: hidden;
      position: relative;

      /* Твой фон */
      background-image: url('https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=1350&q=80');
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
    }
    button {
      font-size: 20px;
      padding: 10px 20px;
      margin: 10px;
      cursor: pointer;
      position: relative;
      z-index: 10;
    }
    #points, #coins {
      font-size: 36px;
      margin: 20px;
      color: white;
      text-shadow: 0 0 5px black;
    }
    #upgradeBtn {
      background-color: gold;
      border: none;
      border-radius: 5px;
    }
    #upgradeBtn:disabled {
      background-color: #ccc;
      cursor: not-allowed;
    }
    #exchangeBtn {
      background-color: #4CAF50;
      border: none;
      border-radius: 5px;
      color: white;
    }
    #purpleSquare, #brownTriangle {
      position: absolute;
      width: 50px;
      height: 50px;
      cursor: pointer;
      display: none;
      z-index: 1000;
    }
    #purpleSquare {
      background-color: purple;
      border-radius: 5px;
    }
    #brownTriangle {
      width: 0;
      height: 0;
      border-left: 25px solid transparent;
      border-right: 25px solid transparent;
      border-bottom: 50px solid brown;
      background-color: transparent;
      border-radius: 0;
    }
  </style>
</head>
<body>

  <h1 style="color:white; text-shadow: 0 0 7px black;">Кликер</h1>

  <div>
    <div>Очки: <span id="points">0</span></div>
    <div>Монеты: <span id="coins">0</span></div>
  </div>

  <button id="clickBtn">Кликни меня!</button><br>

  <button id="exchangeBtn" disabled>Обменять очки на монеты</button><br>

  <button id="upgradeBtn" disabled>Улучшить силу клика (стоимость: <span id="upgradeCost">100</span> монет)</button>

  <div id="purpleSquare"></div>
  <div id="brownTriangle"></div>

  <script>
    let points = 0;
    let coins = 0;

    let baseClickPower = 1;      // Постоянная сила клика (улучшения)
    let tempClickMultiplier = 1; // Временный множитель (от квадрата)

    let upgradeCost = 100;

    const pointsDisplay = document.getElementById('points');
    const coinsDisplay = document.getElementById('coins');
    const clickBtn = document.getElementById('clickBtn');
    const upgradeBtn = document.getElementById('upgradeBtn');
    const upgradeCostDisplay = document.getElementById('upgradeCost');
    const exchangeBtn = document.getElementById('exchangeBtn');
    const purpleSquare = document.getElementById('purpleSquare');
    const brownTriangle = document.getElementById('brownTriangle');

    function getClickPower() {
      return baseClickPower * tempClickMultiplier;
    }

    function updateDisplays() {
      pointsDisplay.textContent = Math.floor(points);
      coinsDisplay.textContent = coins;
      upgradeCostDisplay.textContent = upgradeCost;

      upgradeBtn.disabled = coins < upgradeCost;
      exchangeBtn.disabled = points === 0;
    }

    clickBtn.onclick = () => {
      points += getClickPower();
      updateDisplays();
    };

    exchangeBtn.onclick = () => {
      if(points > 0) {
        coins += Math.floor(points);
        points = 0;
        updateDisplays();
      }
    };

    // Получить случайную сторону экрана
    function getRandomEdge() {
      const edges = ['top', 'right', 'bottom', 'left'];
      return edges[Math.floor(Math.random() * edges.length)];
    }

    // Координаты старта фигуры по краю
    function getStartPosition(edge, size) {
      const w = window.innerWidth;
      const h = window.innerHeight;
      switch(edge) {
        case 'top':    return { x: Math.random() * (w - size), y: -size };
        case 'right':  return { x: w, y: Math.random() * (h - size) };
        case 'bottom': return { x: Math.random() * (w - size), y: h };
        case 'left':   return { x: -size, y: Math.random() * (h - size) };
      }
    }

    // Запуск анимации фигуры, летящей через кнопку upgradeBtn
    function launchShape(shapeElem, onClickEffect) {
      const size = 50;
      const w = window.innerWidth;
      const h = window.innerHeight;

      const upgradeRect = upgradeBtn.getBoundingClientRect();
      const targetX = upgradeRect.left + upgradeRect.width/2 - size/2;
      const targetY = upgradeRect.top + upgradeRect.height/2 - size/2;

      const distToTop = upgradeRect.top;
      const distToBottom = h - upgradeRect.bottom;
      const distToLeft = upgradeRect.left;
      const distToRight = w - upgradeRect.right;

      const distances = [
        { side: 'top', dist: distToTop },
        { side: 'bottom', dist: distToBottom },
        { side: 'left', dist: distToLeft },
        { side: 'right', dist: distToRight }
      ].sort((a,b) => a.dist - b.dist);

      const allowedSides = distances.slice(1).map(d => d.side);

      const startEdge = allowedSides[Math.floor(Math.random() * allowedSides.length)];

      const pos = getStartPosition(startEdge, size);

      shapeElem.style.left = pos.x + 'px';
      shapeElem.style.top = pos.y + 'px';
      shapeElem.style.display = 'block';

      let dx = targetX - pos.x;
      let dy = targetY - pos.y;
      const dist = Math.sqrt(dx*dx + dy*dy);

      dx /= dist;
      dy /= dist;

      const speed = 4;

      let animationFrameId;

      function move() {
        pos.x += dx * speed;
        pos.y += dy * speed;

        shapeElem.style.left = pos.x + 'px';
        shapeElem.style.top = pos.y + 'px';

        if (
          pos.x < -size*2 || pos.x > w + size*2 ||
          pos.y < -size*2 || pos.y > h + size*2
        ) {
          shapeElem.style.display = 'none';
          cancelAnimationFrame(animationFrameId);
          return;
        }

        animationFrameId = requestAnimationFrame(move);
      }

      animationFrameId = requestAnimationFrame(move);

      shapeElem.onclick = () => {
        shapeElem.style.display = 'none';
        cancelAnimationFrame(animationFrameId);
        onClickEffect();
      };
    }

    setInterval(() => {
      launchShape(purpleSquare, () => {
        if (tempClickMultiplier === 1) {
          tempClickMultiplier = 5;
          alert('Сила клика увеличена в 5 раз на 5 секунд!');
          setTimeout(() => {
            tempClickMultiplier = 1;
            alert('Временное улучшение закончилось.');
          }, 5000);
        }
      });
    }, 20000);

    setInterval(() => {
      launchShape(brownTriangle, () => {
        points *= 0.9;
        baseClickPower *= 0.9;
        alert('Очки и сила клика уменьшены на 10%!');
        updateDisplays();
      });
    }, 30000);

    upgradeBtn.onclick = () => {
      if (coins >= upgradeCost) {
        coins -= upgradeCost;
        baseClickPower *= 10;
        upgradeCost *= 20;
        updateDisplays();
        alert('Улучшение куплено! Сила клика увеличена в 10 раз.');
      }
    };

    updateDisplays();
  </script>

</body>
</html>
