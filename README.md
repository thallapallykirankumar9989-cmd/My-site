<!index.md>
<html lang="te">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>Vamshi's Chess Game</title>
  <style>
    body {
      margin: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      background-color: #f4f4f4;
      font-family: Arial, sans-serif;
    }
    
    h2 { margin-top: 20px; }
    
    /* మొబైల్‌కి తగ్గట్టుగా ఆటోమేటిక్ సైజు తీసుకునే బోర్డ్ */
    #chessboard {
      display: grid;
      grid-template-columns: repeat(8, 1fr);
      grid-template-rows: repeat(8, 1fr);
      width: 95vw; /* ఫోన్ వెడల్పులో 95% తీసుకుంటుంది */
      max-width: 400px; /* కంప్యూటర్ లో మరీ పెద్దగా కాకుండా లిమిట్ */
      height: 95vw;
      max-height: 400px;
      border: 4px solid #333;
    }
    
    .square {
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 8vw; /* పావుల సైజు కూడా స్క్రీన్‌ని బట్టి మారుతుంది */
      cursor: pointer;
      position: relative;
    }

    /* కంప్యూటర్ స్క్రీన్ కి పావుల సైజు లిమిట్ */
    @media (min-width: 400px) {
      .square { font-size: 35px; }
    }

    .white { background-color: #f0d9b5; }
    .black { background-color: #b58863; }
    .selected { background-color: #baca44; } 

    /* వెళ్లగలిగే దారిని చూపే డాట్ డిజైన్ */
    .dot::after {
      content: '';
      width: 25%;
      height: 25%;
      background-color: rgba(0, 0, 0, 0.4);
      border-radius: 50%;
      position: absolute;
    }
  </style>
</head>
<body>

  <h2>Vamshi's Chess Game</h2>
  <div id="chessboard"></div>

  <script>
    const boardElement = document.getElementById('chessboard');
    let squares = []; 

    const initialBoard = [
      ['♜', '♞', '♝', '♛', '♚', '♝', '♞', '♜'],
      ['♟', '♟', '♟', '♟', '♟', '♟', '♟', '♟'],
      ['', '', '', '', '', '', '', ''],
      ['', '', '', '', '', '', '', ''],
      ['', '', '', '', '', '', '', ''],
      ['', '', '', '', '', '', '', ''],
      ['♙', '♙', '♙', '♙', '♙', '♙', '♙', '♙'],
      ['♖', '♘', '♗', '♕', '♔', '♗', '♘', '♖']
    ];

    // బోర్డ్ క్రియేట్ చేయడం
    for (let row = 0; row < 8; row++) {
      squares[row] = [];
      for (let col = 0; col < 8; col++) {
        const square = document.createElement('div');
        square.className = `square ${(row + col) % 2 === 0 ? 'white' : 'black'}`;
        square.innerText = initialBoard[row][col];
        
        // గడిని క్లిక్ చేసినప్పుడు యాక్షన్
        square.addEventListener('click', () => handleSquareClick(row, col));
        
        boardElement.appendChild(square);
        squares[row][col] = square; 
      }
    }

    function clearHighlights() {
      for (let r = 0; r < 8; r++) {
        for (let c = 0; c < 8; c++) {
          squares[r][c].classList.remove('selected', 'dot');
        }
      }
    }

    function handleSquareClick(row, col) {
      clearHighlights();
      const piece = squares[row][col].innerText;
      
      if (piece !== '') {
        squares[row][col].classList.add('selected');
        showPossibleMoves(piece, row, col);
      }
    }

    // పావులు ఎక్కడికి వెళ్తాయో డాట్స్ చూపే లాజిక్
    function showPossibleMoves(piece, row, col) {
      // తెలుపు భటుడు
      if (piece === '♙') {
        if (row - 1 >= 0 && squares[row - 1][col].innerText === '') addDot(row - 1, col);
        if (row === 6 && squares[row - 2][col].innerText === '') addDot(row - 2, col);
      }
      // నలుపు భటుడు
      if (piece === '♟') {
        if (row + 1 < 8 && squares[row + 1][col].innerText === '') addDot(row + 1, col);
        if (row === 1 && squares[row + 2][col].innerText === '') addDot(row + 2, col);
      }
      // గుర్రం
      if (piece === '♘' || piece === '♞') {
        const knightMoves = [
          [-2, -1], [-2, 1], [-1, -2], [-1, 2],
          [1, -2], [1, 2], [2, -1], [2, 1]
        ];
        knightMoves.forEach(move => {
          const newRow = row + move[0];
          const newCol = col + move[1];
          if (newRow >= 0 && newRow < 8 && newCol >= 0 && newCol < 8) {
            addDot(newRow, newCol);
          }
        });
      }
    }

    function addDot(row, col) {
      squares[row][col].classList.add('dot');
    }
  </script>

</body>
</html>
