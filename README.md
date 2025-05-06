<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GEOVISION 4006 - Türkiye İklim Haritası</title>
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(to bottom right, #e0f7fa, #f1f8e9);
      color: #333;
    }
    header {
      background-color: #003049;
      color: white;
      padding: 20px;
      text-align: center;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    }
    h1 {
      margin: 0;
      font-size: 2em;
    }
    #map-container {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-top: 30px;
      overflow: hidden;
    }
    #map {
      transition: transform 0.3s ease;
      cursor: grab;
    }
    .region-info {
      display: none;
      margin: 20px auto;
      max-width: 600px;
      padding: 20px;
      background: white;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    .region-info img {
      max-width: 100%;
      border-radius: 10px;
      margin-top: 10px;
    }
    .region-icon {
      width: 24px;
      vertical-align: middle;
      margin-right: 8px;
    }
    .game-container {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      margin: 40px auto;
      max-width: 800px;
    }
    .card {
      width: 100px;
      height: 100px;
      margin: 10px;
      perspective: 1000px;
      cursor: pointer;
    }
    .card-inner {
      position: relative;
      width: 100%;
      height: 100%;
      transition: transform 0.6s;
      transform-style: preserve-3d;
    }
    .card.flipped .card-inner {
      transform: rotateY(180deg);
    }
    .card-front, .card-back {
      position: absolute;
      width: 100%;
      height: 100%;
      backface-visibility: hidden;
      border-radius: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 24px;
      font-weight: bold;
    }
    .card-front {
      background-color: #90caf9;
      color: white;
    }
    .card-back {
      background-color: #eeeeee;
      transform: rotateY(180deg);
    }
  </style>
</head>
<body>
  <header>
    <h1>GEOVISION 4006 - Türkiye İklim Haritası</h1>
    <p>Haritadan bir bölgeye tıklayarak detaylı bilgiye ulaşabilirsiniz.</p>
  </header>

  <div id="map-container">
    <img id="map" src="turkiye-iklim-haritasi.png" alt="Türkiye Haritası" usemap="#image-map">
  </div>

  <!-- Harici Hafıza Oyunu Linki -->
  <div style="text-align: center; margin-top: 60px;">
    <a href="https://fbkrl0.github.io/Haf-za-Oyunu/" target="_blank" style="text-decoration: none; color: #003049;">
      <img src="https://cdn-icons-png.flaticon.com/512/854/854878.png" alt="Hafıza Oyunu İkonu" width="80" style="border-radius: 16px; box-shadow: 0 4px 12px rgba(0,0,0,0.2); transition: transform 0.3s;" />
      <div style="font-size: 1.3em; font-weight: bold; margin-top: 10px;">Harici Hafıza Oyunu</div>
      <div style="font-size: 0.95em; color: #555;">İklim bilginizi test edin!</div>
    </a>
  </div>

  <div id="info" class="region-info"></div>

  <div class="game-container" id="game"></div>

  <map name="image-map">
    <!-- Harita alanları burada devam ediyor -->
  </map>

  <script>
    const cardsArray = ['A','B','C','D','E','F','A','B','C','D','E','F'];
    let flippedCards = [];
    let matchedPairs = 0;
    const game = document.getElementById('game');

    function shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    }

    function createCard(content) {
      const card = document.createElement('div');
      card.classList.add('card');

      const inner = document.createElement('div');
      inner.classList.add('card-inner');

      const front = document.createElement('div');
      front.classList.add('card-front');
      front.textContent = content;

      const back = document.createElement('div');
      back.classList.add('card-back');
      back.textContent = '?';

      inner.appendChild(front);
      inner.appendChild(back);
      card.appendChild(inner);

      card.addEventListener('click', () => {
        if (card.classList.contains('flipped') || flippedCards.length === 2) return;
        card.classList.add('flipped');
        flippedCards.push({ card, content });

        if (flippedCards.length === 2) {
          const [first, second] = flippedCards;
          if (first.content === second.content) {
            setTimeout(() => {
              first.card.style.visibility = 'hidden';
              second.card.style.visibility = 'hidden';
              matchedPairs++;
              if (matchedPairs === cardsArray.length / 2) {
                alert('Tebrikler! Tüm kartları eşleştirdiniz.');
              }
              flippedCards = [];
            }, 1000);
          } else {
            setTimeout(() => {
              first.card.classList.remove('flipped');
              second.card.classList.remove('flipped');
              flippedCards = [];
            }, 1000);
          }
        }
      });

      return card;
    }

    shuffle(cardsArray).forEach(content => {
      const card = createCard(content);
      game.appendChild(card);
    });

    const map = document.getElementById('map');
    let isDragging = false, startX, startY, currentX = 0, currentY = 0, scale = 1;

    map.addEventListener('mousedown', (e) => {
      isDragging = true;
      startX = e.clientX - currentX;
      startY = e.clientY - currentY;
      map.style.cursor = 'grabbing';
    });

    window.addEventListener('mousemove', (e) => {
      if (!isDragging) return;
      currentX = e.clientX - startX;
      currentY = e.clientY - startY;
      updateTransform();
    });

    window.addEventListener('mouseup', () => {
      isDragging = false;
      map.style.cursor = 'grab';
    });

    window.addEventListener('wheel', (e) => {
      e.preventDefault();
      const delta = e.deltaY > 0 ? -0.1 : 0.1;
      scale = Math.min(Math.max(0.5, scale + delta), 2);
      updateTransform();
    });

    function updateTransform() {
      map.style.transform = `translate(${currentX}px, ${currentY}px) scale(${scale})`;
    }
  </script>
</body>
</html>
