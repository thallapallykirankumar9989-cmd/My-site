<!DOCTYPE html>
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
    
    h2 { margin-top: 10px; margin-bottom: 5px; }
    #turn-indicator { font-weight: bold; margin-bottom: 15px; color: #333; }
    
    #chessboard {
      display: grid;
      grid-template-columns: repeat(8, 1fr);
      grid-template-rows: repeat(8, 1fr);
      width: 95vw;
      max-width: 400px;
      height: 95vw;
      max-height: 400px;
      border: 4px solid #333;
    }
    
    .square {
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 8vw;
      cursor: pointer;
      position: relative;
    }

    @media (min-width: 400px) {
      .square { font-size: 35px; }
    }

    .white { background-color: #f0d9b5; }
    .black { background-color: #b58863; }
    .selected { background-color: #baca44; } 

    /* నార్మల్ గా వెళ్లగలిగే దారికి నల్లటి చుక్క */
    .dot::after {
      content: '';
      width: 25%;
      height: 25%;
      background-color: rgba(0, 0, 0, 0.4);
      border-radius: 50%;
      position: absolute;
    }

    /* చంపడానికి (Capture) వీలున్న పావు మీద ఎర్రటి చుక్క */
    .capture-dot::after {
      content: '';
      width: 80%;
      height: 80%;
      border: 4px solid rgba(255, 0, 0, 0.6);
      border-radius: 50%;
      position: absolute;
    }
  </style>
</head>
<body>

  <h2>Vamshi's Chess Game</h2>
  <div id="turn-indicator">Turn: White</div>
  <div id="chessboard"></div>

  <script>
    const boardElement = document.getElementById('chessboard');
    const turnIndicator = document.getElementById('turn-indicator');
    let squares = []; 
    let selectedSquare = null; 
    let currentTurn = 'white'; // ఎవరి టర్న్ అని గుర్తుపెట్టుకోవడానికి

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

    // పావు రంగు కనుక్కోవడానికి ఫంక్షన్
    function getPieceColor(piece) {
      if (['♙', '♖', '♘', '♗', '♕', '♔'].includes(piece)) return 'white';
      if (['♟', '♜', '♞', '♝', '♛', '♚'].includes(piece)) return 'black';
      return null;
    }

    // బోర్డ్ క్రియేట్ చేయడం
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

    function handleSquareClick(row, col) {
      const clickedSquare = squares[row][col];
      const clickedPiece = clickedSquare.innerText;

      // కదిలించడానికి లేదా చంపడానికి క్లిక్ చేస్తే
      if ((clickedSquare.classList.contains('dot') || clickedSquare.classList.contains('capture-dot')) && selectedSquare) {
        
        // రాజును చంపేస్తే గేమ్ ఓవర్ (Checkmate)
        if (clickedPiece === '♚') {
          setTimeout(() => alert("Checkmate! White Wins!"), 100);
        } else if (clickedPiece === '♔') {
          setTimeout(() => alert("Checkmate! Black Wins!"), 100);
        }

        movePiece(selectedSquare.row, selectedSquare.col, row, col);
        clearHighlights();
        selectedSquare = null; 
        
        // టర్న్ మార్చడం
        currentTurn = currentTurn === 'white' ? 'black' : 'white';
        turnIndicator.innerText = `Turn: ${currentTurn.charAt(0).toUpperCase() + currentTurn.slice(1)}`;
        return;
      }

      clearHighlights();
      
      // కేవలం వాళ్ల టర్న్ ఉన్న పావులను మాత్రమే ముట్టుకోనివ్వాలి
      if (clickedPiece !== '' && getPieceColor(clickedPiece) === currentTurn) {
        clickedSquare.classList.add('selected');
        selectedSquare = { row, col, piece: clickedPiece }; 
        showPossibleMoves(clickedPiece, row, col);
      } else {
        selectedSquare = null; 
      }
    }

    function movePiece(fromRow, fromCol, toRow, toCol) {
      const piece = squares[fromRow][fromCol].innerText;
      squares[toRow][toCol].innerText = piece; 
      squares[fromRow][fromCol].innerText = ''; 
    }

    // పావుల కదలికలు మరియు చంపే లాజిక్
    function showPossibleMoves(piece, row, col) {
      const color = getPieceColor(piece);

      // 1. తెలుపు భటుడు
      if (piece === '♙') {
        if (row - 1 >= 0 && squares[row - 1][col].innerText === '') {
          addDot(row - 1, col);
          if (row === 6 && squares[row - 2][col].innerText === '') addDot(row - 2, col);
        }
        // క్రాస్‌గా చంపడం
        if (row - 1 >= 0 && col - 1 >= 0 && getPieceColor(squares[row - 1][col - 1].innerText) === 'black') addCapture(row - 1, col - 1);
        if (row - 1 >= 0 && col + 1 < 8 && getPieceColor(squares[row - 1][col + 1].innerText) === 'black') addCapture(row - 1, col + 1);
      }
      
      // 2. నలుపు భటుడు
      if (piece === '♟') {
        if (row + 1 < 8 && squares[row + 1][col].innerText === '') {
          addDot(row + 1, col);
          if (row === 1 && squares[row + 2][col].innerText === '') addDot(row + 2, col);
        }
        // క్రాస్‌గా చంపడం
        if (row + 1 < 8 && col - 1 >= 0 && getPieceColor(squares[row + 1][col - 1].innerText) === 'white') addCapture(row + 1, col - 1);
        if (row + 1 < 8 && col + 1 < 8 && getPieceColor(squares[row + 1][col + 1].innerText) === 'white') addCapture(row + 1, col + 1);
      }

      // 3. గుర్రం
      if (piece === '♘' || piece === '♞') {
        const knightMoves = [[-2, -1], [-2, 1], [-1, -2], [-1, 2], [1, -2], [1, 2], [2, -1], [2, 1]];
        knightMoves.forEach(move => {
          const r = row + move[0];
          const c = col + move[1];
          if (r >= 0 && r < 8 && c >= 0 && c < 8) {
            const targetPiece = squares[r][c].innerText;
            if (targetPiece === '') addDot(r, c);
            else if (getPieceColor(targetPiece) !== color) addCapture(r, c);
          }
        });
      }

      // 4, 5, 6, 7. ఏనుగు, మంత్రి, రాణి, రాజు లాజిక్ (ఉన్నది ఉన్నట్లుగానే, కొత్త కండిషన్స్ తో)
      if (piece === '♖' || piece === '♜') checkDirections(row, col, color, [[-1, 0], [1, 0], [0, -1], [0, 1]]);
      if (piece === '♗' || piece === '♝') checkDirections(row, col, color, [[-1, -1], [-1, 1], [1, -1], [1, 1]]);
      if (piece === '♕' || piece === '♛') checkDirections(row, col, color, [[-1, 0], [1, 0], [0, -1], [0, 1], [-1, -1], [-1, 1], [1, -1], [1, 1]]);
      
      if (piece === '♔' || piece === '♚') {
        const kingMoves = [[-1, -1], [-1, 0], [-1, 1], [0, -1], [0, 1], [1, -1], [1, 0], [1, 1]];
        kingMoves.forEach(move => {
          const r = row + move[0];
          const c = col + move[1];
          if (r >= 0 && r < 8 && c >= 0 && c < 8) {
            const targetPiece = squares[r][c].innerText;
            if (targetPiece === '') addDot(r, c);
            else if (getPieceColor(targetPiece) !== color) addCapture(r, c);
          }
        });
      }
    }

    // దారిలో వెళ్లేటప్పుడు వేరే పావు వస్తే ఆగిపోవడం లేదా చంపడం
    function checkDirections(startRow, startCol, myColor, directions) {
      directions.forEach(dir => {
        let r = startRow + dir[0];
        let c = startCol + dir[1];
        while (r >= 0 && r < 8 && c >= 0 && c < 8) {
          const targetPiece = squares[r][c].innerText;
          if (targetPiece === '') {
            addDot(r, c);
          } else {
            if (getPieceColor(targetPiece) !== myColor) {
              addCapture(r, c); // శత్రువు పావు అయితే ఎర్ర చుక్క పెట్టు
            }
            break; // మన పావు అయినా, శత్రువు పావు అయినా అక్కడితో దారి ఆగిపోవాలి
          }
          r += dir[0];
          c += dir[1];
        }
      });
    }

    function addDot(row, col) { squares[row][col].classList.add('dot'); }
    function addCapture(row, col) { squares[row][col].classList.add('capture-dot'); }
  </script>

</body>
</html>
