<!DOCTYPE html>
<html>
<head>
  <title>Chess Game by Vamshi</title>
  <style>
    /* చెస్ బోర్డ్ సైజు మరియు డిజైన్ (CSS) */
    #chessboard {
      display: grid;
      grid-template-columns: repeat(8, 60px);
      grid-template-rows: repeat(8, 60px);
      width: 480px;
      border: 4px solid #333;
      margin: 20px auto;
    }
    .white { background-color: #f0d9b5; } /* తెలుపు గడుల రంగు */
    .black { background-color: #b58863; } /* నలుపు గడుల రంగు */
  </style>
</head>
<body>

  <h2 style="text-align: center;">Vamshi's Chess Game</h2>
  <div id="chessboard"></div>

  <script>
    // చెస్ బోర్డ్‌లో 64 గడులను క్రియేట్ చేసే లాజిక్ (JavaScript)
    const board = document.getElementById('chessboard');
    
    for (let row = 0; row < 8; row++) {
      for (let col = 0; col < 8; col++) {
        const square = document.createElement('div');
        
        // సరి బేసి సంఖ్యల ఆధారంగా నలుపు, తెలుపు రంగులు ఇవ్వడం
        if ((row + col) % 2 === 0) {
          square.className = 'white';
        } else {
          square.className = 'black';
        }
        
        board.appendChild(square);
      }
    }
  </script>

</body>
</html>
