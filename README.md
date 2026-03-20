<!DOCTYPE html>
<html lang="te">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>Vamshi's Pro Chess</title>
  <style>
    body {
      margin: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      background-color: #2c2b29;
      font-family: Arial, sans-serif;
      color: white;
    }
    
    h2 { margin-top: 15px; margin-bottom: 10px; }
    
    .top-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 95vw;
      max-width: 400px;
      margin-bottom: 10px;
    }

    #turn-indicator { 
      font-weight: bold; 
      font-size: 18px; 
      color: #fff; 
      background-color: rgba(0,0,0,0.5);
      padding: 5px 10px;
      border-radius: 5px;
    }

    #restart-btn {
      padding: 8px 15px;
      font-size: 14px;
      font-weight: bold;
      color: white;
      background-color: #7fa650;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      box-shadow: 0px 4px 6px rgba(0,0,0,0.3);
      transition: 0.2s;
    }

    #restart-btn:hover { background-color: #6a8c42; }
    
    #check-alert {
      color: #ff4c4c;
      font-size: 20px;
      font-weight: bold;
      height: 25px; 
      margin-bottom: 10px;
      text-shadow: 1px 1px 2px black;
    }
    
    #chessboard {
      display: grid;
      grid-template-columns: repeat(8, 1fr);
      grid-template-rows: repeat(8, 1fr);
      width: 95vw;
      max-width: 400px;
      height: 95vw;
      max-height: 400px;
      border: 4px solid #1e1e1e;
      border-radius: 4px;
      box-shadow: 0px 10px 20px rgba(0,0,0,0.5);
    }
    
    .square {
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 8vw;
      cursor: pointer;
      position: relative;
      text-shadow: 2px 3px 4px rgba(0, 0, 0, 0.6); 
    }

    @media (min-width: 400px) {
      .square { font-size: 35px; }
    }

    .white { background-color: #ebecd0; color: #000; }
    .black { background-color: #739552; color: #000; }
    .selected { background-color: #f6f669 !important; } 

    .dot::after {
      content: '';
      width: 25%;
      height: 25%;
      background-color: rgba(0, 0, 0, 0.3);
      border-radius: 50%;
      position: absolute;
    }

    .capture-dot::after {
      content: '';
      width: 85%;
      height: 85%;
      border: 5px solid rgba(255, 0, 0, 0.7);
      border-radius: 50%;
      position: absolute;
    }

    .in-check { background-color: #ff6666 !important; box-shadow: inset 0 0 10px red; }
  </style>
</head>
<body>

  <h2>Vamshi's Pro Chess</h2>
  
  <div class="top-bar">
    <div id="turn-indicator">Turn: White</div>
    <button id="restart-btn" onclick="restartGame()">Restart</button>
  </div>
  
  <div id="check-alert"></div> 
  <div id="chessboard"></div>

  <script>
    const boardElement = document.getElementById('chessboard');
    const turnIndicator = document.getElementById('turn-indicator');
    const checkAlert = document.getElementById('check-alert');
    let squares = []; 
    let selectedSquare = null; 
    let currentTurn = 'white'; 
    let gameOver = false; 

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

    function getPieceColor(piece) {
      if (['♙', '♖', '♘', '♗', '♕', '♔'].includes(piece)) return 'white';
      if (['♟', '♜', '♞', '♝', '♛', '♚'].includes(piece)) return 'black';
      return null;
    }

    for (let row = 0; row < 8; row++) {
      squares[row] = [];
      for (let col = 0; col < 8; col++) {
        const square = document.createElement('div');
        square.className = `square ${(row + col) % 2 === 0 ? 'white' : 'black'}`;
        square.innerText = initialBoard[row][col];
        square.addEventListener('click', () => handleSquareClick(row, col));
        boardElement.appendChild(square);
        squares[row][col] = square; 
      }
    }

    function clearHighlights() {
      for (let r = 0; r < 8; r++) {
        for (let c = 0; c < 8; c++) {
          squares[r][c].classList.remove('selected', 'dot', 'capture-dot');
        }
      }
    }

    function removeCheckHighlight() {
      for (let r = 0; r < 8; r++) {
        for (let c = 0; c < 8; c++) {
          squares[r][c].classList.remove('in-check');
        }
      }
    }

    function restartGame() {
      gameOver = false;
      currentTurn = 'white';
      selectedSquare = null;
      turnIndicator.innerText = "Turn: White";
      checkAlert.innerText = "";
      checkAlert.style.color = "#ff4c4c";

      for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
          squares[row][col].innerText = initialBoard[row][col];
          squares[row][col].className = `square ${(row + col) % 2 === 0 ? 'white' : 'black'}`;
        }
      }
    }

    function handleSquareClick(row, col) {
      if (gameOver) return; 

      const clickedSquare = squares[row][col];
      const clickedPiece = clickedSquare.innerText;

      if ((clickedSquare.classList.contains('dot') || clickedSquare.classList.contains('capture-dot')) && selectedSquare) {
        
        let isCheckmate = false;
        let winnerName = "";

        if (clickedPiece === '♚') {
          isCheckmate = true;
          winnerName = "White";
        } else if (clickedPiece === '♔') {
          isCheckmate = true;
          winnerName = "Black";
        }

        movePiece(selectedSquare.row, selectedSquare.col, row, col);
        clearHighlights();
        removeCheckHighlight(); 
        selectedSquare = null; 
        
        if (isCheckmate) {
          // రాజు చనిపోయిన ఇమేజ్ స్క్రీన్ మీద కనిపించాక అలర్ట్ రావడానికి 100 మిల్లీసెకన్ల డిలే ఇస్తున్నాం
          setTimeout(() => {
            alert(`🏆 CHECKMATE! ${winnerName} Wins! 🏆\n\nప్లేయర్ 'OK' నొక్కగానే గేమ్ రీస్టార్ట్ అవుతుంది.`);
            restartGame(); // అలర్ట్ క్లోజ్ అవ్వగానే ఆటోమేటిక్ గా గేమ్ రీస్టార్ట్ అవుతుంది
          }, 100);
          return; 
        }

        currentTurn = currentTurn === 'white' ? 'black' : 'white';
        turnIndicator.innerText = `Turn: ${currentTurn.charAt(0).toUpperCase() + currentTurn.slice(1)}`;

        if (isCheck(currentTurn)) {
          checkAlert.style.color = "#ff4c4c";
          checkAlert.innerText = "⚠️ CHECK! ⚠️";
          highlightKing(currentTurn); 
        } else {
          checkAlert.innerText = "";
        }
        return;
      }

      clearHighlights();
      
      if (clickedPiece !== '' && getPieceColor(clickedPiece) === currentTurn) {
        clickedSquare.classList.add('selected');
        selectedSquare = { row, col, piece: clickedPiece }; 
        showPossibleMoves(clickedPiece, row, col);
      } else {
        selectedSquare = null; 
      }
    }

    function movePiece(fromRow, fromCol, toRow, toCol) {
      squares[toRow][toCol].innerText = squares[fromRow][fromCol].innerText; 
      squares[fromRow][fromCol].innerText = ''; 
    }

    function showPossibleMoves(piece, row, col) {
      const color = getPieceColor(piece);

      if (piece === '♙') {
        if (row - 1 >= 0 && squares[row - 1][col].innerText === '') {
          addDot(row - 1, col);
          if (row === 6 && squares[row - 2][col].innerText === '') addDot(row - 2, col);
        }
        if (row - 1 >= 0 && col - 1 >= 0 && getPieceColor(squares[row - 1][col - 1].innerText) === 'black') addCapture(row - 1, col - 1);
        if (row - 1 >= 0 && col + 1 < 8 && getPieceColor(squares[row - 1][col + 1].innerText) === 'black') addCapture(row - 1, col + 1);
      }
      
      if (piece === '♟') {
        if (row + 1 < 8 && squares[row + 1][col].innerText === '') {
          addDot(row + 1, col);
          if (row === 1 && squares[row + 2][col].innerText === '') addDot(row + 2, col);
        }
        if (row + 1 < 8 && col - 1 >= 0 && getPieceColor(squares[row + 1][col - 1].innerText) === 'white') addCapture(row + 1, col - 1);
        if (row + 1 < 8 && col + 1 < 8 && getPieceColor(squares[row + 1][col + 1].innerText) === 'white') addCapture(row + 1, col + 1);
      }

      if (piece === '♘' || piece === '♞') {
        const knightMoves = [[-2, -1], [-2, 1], [-1, -2], [-1, 2], [1, -2], [1, 2], [2, -1], [2, 1]];
        knightMoves.forEach(move => {
          const r = row + move[0], c = col + move[1];
          if (r >= 0 && r < 8 && c >= 0 && c < 8) {
            const targetPiece = squares[r][c].innerText;
            if (targetPiece === '') addDot(r, c);
            else if (getPieceColor(targetPiece) !== color) addCapture(r, c);
          }
        });
      }

      if (piece === '♖' || piece === '♜') checkDirections(row, col, color, [[-1, 0], [1, 0], [0, -1], [0, 1]]);
      if (piece === '♗' || piece === '♝') checkDirections(row, col, color, [[-1, -1], [-1, 1], [1, -1], [1, 1]]);
      if (piece === '♕' || piece === '♛') checkDirections(row, col, color, [[-1, 0], [1, 0], [0, -1], [0, 1], [-1, -1], [-1, 1], [1, -1], [1, 1]]);
      
      if (piece === '♔' || piece === '♚') {
        const kingMoves = [[-1, -1], [-1, 0], [-1, 1], [0, -1], [0, 1], [1, -1], [1, 0], [1, 1]];
        kingMoves.forEach(move => {
          const r = row + move[0], c = col + move[1];
          if (r >= 0 && r < 8 && c >= 0 && c < 8) {
            const targetPiece = squares[r][c].innerText;
            if (targetPiece === '') addDot(r, c);
            else if (getPieceColor(targetPiece) !== color) addCapture(r, c);
          }
        });
      }
    }

    function checkDirections(startRow, startCol, myColor, directions) {
      directions.forEach(dir => {
        let r = startRow + dir[0], c = startCol + dir[1];
        while (r >= 0 && r < 8 && c >= 0 && c < 8) {
          const targetPiece = squares[r][c].innerText;
          if (targetPiece === '') addDot(r, c);
          else {
            if (getPieceColor(targetPiece) !== myColor) addCapture(r, c);
            break; 
          }
          r += dir[0], c += dir[1];
        }
      });
    }

    function addDot(row, col) { squares[row][col].classList.add('dot'); }
    function addCapture(row, col) { squares[row][col].classList.add('capture-dot'); }

    function isCheck(kingColor) {
      let kingRow = -1, kingCol = -1;
      const kingPiece = kingColor === 'white' ? '♔' : '♚';
      
      for (let r = 0; r < 8; r++) {
        for (let c = 0; c < 8; c++) {
          if (squares[r][c].innerText === kingPiece) {
            kingRow = r; kingCol = c; break;
          }
        }
      }
      if (kingRow === -1) return false; 

      const attackerColor = kingColor === 'white' ? 'black' : 'white';

      const knightMoves = [[-2, -1], [-2, 1], [-1, -2], [-1, 2], [1, -2], [1, 2], [2, -1], [2, 1]];
      for (let move of knightMoves) {
        const r = kingRow + move[0], c = kingCol + move[1];
        if (r >= 0 && r < 8 && c >= 0 && c < 8) {
          const piece = squares[r][c].innerText;
          if (getPieceColor(piece) === attackerColor && (piece === '♘' || piece === '♞')) return true;
        }
      }

      const straightDirs = [[-1, 0], [1, 0], [0, -1], [0, 1]];
      if (checkLineOfSight(kingRow, kingCol, attackerColor, straightDirs, ['♖', '♜', '♕', '♛'])) return true;

      const diagDirs = [[-1, -1], [-1, 1], [1, -1], [1, 1]];
      if (checkLineOfSight(kingRow, kingCol, attackerColor, diagDirs, ['♗', '♝', '♕', '♛'])) return true;

      if (kingColor === 'white') {
        if (kingRow - 1 >= 0 && kingCol - 1 >= 0 && squares[kingRow - 1][kingCol - 1].innerText === '♟') return true;
        if (kingRow - 1 >= 0 && kingCol + 1 < 8 && squares[kingRow - 1][kingCol + 1].innerText === '♟') return true;
      } else {
        if (kingRow + 1 < 8 && kingCol - 1 >= 0 && squares[kingRow + 1][kingCol - 1].innerText === '♙') return true;
        if (kingRow + 1 < 8 && kingCol + 1 < 8 && squares[kingRow + 1][kingCol + 1].innerText === '♙') return true;
      }

      return false;
    }

    function checkLineOfSight(startRow, startCol, attackerColor, directions, attackerPieces) {
      for (let dir of directions) {
        let r = startRow + dir[0], c = startCol + dir[1];
        while (r >= 0 && r < 8 && c >= 0 && c < 8) {
          const piece = squares[r][c].innerText;
          if (piece !== '') {
            if (getPieceColor(piece) === attackerColor && attackerPieces.includes(piece)) return true;
            break; 
          }
          r += dir[0], c += dir[1];
        }
      }
      return false;
    }

    function highlightKing(color) {
      const kingPiece = color === 'white' ? '♔' : '♚';
      for (let r = 0; r < 8; r++) {
        for (let c = 0; c < 8; c++) {
          if (squares[r][c].innerText === kingPiece) {
            squares[r][c].classList.add('in-check');
            return;
          }
        }
      }
    }
  </script>

</body>
</html>
