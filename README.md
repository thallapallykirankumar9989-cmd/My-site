<!DOCTYPE html>
<html lang="te">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>Pro Chess Game</title>
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
      position: relative; 
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

    #promotion-overlay {
      display: none; 
      position: absolute;
      top: 0; left: 0; width: 100%; height: 100%;
      background-color: rgba(0, 0, 0, 0.8);
      justify-content: center;
      align-items: center;
      z-index: 10;
      border-radius: 4px;
    }

    #promotion-menu {
      background-color: white;
      padding: 20px;
      border-radius: 10px;
      display: flex;
      gap: 15px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.5);
    }

    .promo-piece {
      font-size: 40px;
      cursor: pointer;
      color: black;
      transition: 0.2s;
      padding: 5px;
    }
    .promo-piece:hover { transform: scale(1.2); background-color: #ddd; border-radius: 5px; }

  </style>
</head>
<body>

  <h2>Pro Chess Game</h2>
  
  <div class="top-bar">
    <div id="turn-indicator">Turn: White</div>
    <button id="restart-btn" onclick="restartGame()">Restart</button>
  </div>
  
  <div id="check-alert"></div> 
  
  <div id="chessboard">
    <div id="promotion-overlay">
        <div id="promotion-menu"></div>
    </div>
  </div>

  <script>
    const boardElement = document.getElementById('chessboard');
    const turnIndicator = document.getElementById('turn-indicator');
    const checkAlert = document.getElementById('check-alert');
    const promoOverlay = document.getElementById('promotion-overlay');
    const promoMenu = document.getElementById('promotion-menu');
    
    let squares = []; 
    let selectedSquare = null; 
    let currentTurn = 'white'; 
    let gameOver = false; 
    
    let checkMode = false; // చెక్ మేట్ కనుక్కోవడానికి వాడే మోడ్
    let possibleMovesCount = 0; 
    
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

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

    function playSound(frequency, duration, type) {
      if(audioCtx.state === 'suspended') return;
      const oscillator = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();
      oscillator.type = type; 
      oscillator.frequency.setValueAtTime(frequency, audioCtx.currentTime);
      gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime); 
      gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
      oscillator.connect(gainNode);
      gainNode.connect(audioCtx.destination);
      oscillator.start();
      oscillator.stop(audioCtx.currentTime + duration);
    }

    function playMoveSound() { playSound(600, 0.1, 'sine'); } 
    function playCaptureSound() { playSound(200, 0.2, 'triangle'); } 

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
      if (audioCtx.state === 'suspended') audioCtx.resume();
      
      gameOver = false;
      currentTurn = 'white';
      selectedSquare = null;
      turnIndicator.innerText = "Turn: White";
      checkAlert.innerText = "";
      checkAlert.style.color = "#ff4c4c";
      promoOverlay.style.display = 'none'; 
      removeCheckHighlight();

      for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
          squares[row][col].innerText = initialBoard[row][col];
          squares[row][col].className = `square ${(row + col) % 2 === 0 ? 'white' : 'black'}`;
        }
      }
      playMoveSound(); 
    }

    function handleSquareClick(row, col) {
      if (audioCtx.state === 'suspended') audioCtx.resume();
      if (gameOver || promoOverlay.style.display === 'flex') return; 

      const clickedSquare = squares[row][col];
      const clickedPiece = clickedSquare.innerText;

      if ((clickedSquare.classList.contains('dot') || clickedSquare.classList.contains('capture-dot')) && selectedSquare) {
        
        let isCapture = clickedSquare.innerText !== '';
        if (isCapture) playCaptureSound();
        else playMoveSound();

        movePiece(selectedSquare.row, selectedSquare.col, row, col);
        clearHighlights();
        removeCheckHighlight(); 
        selectedSquare = null; 
        
        const movedPiece = squares[row][col].innerText;
        if ((movedPiece === '♙' && row === 0) || (movedPiece === '♟' && row === 7)) {
            showPromotionMenu(row, col, getPieceColor(movedPiece));
            return; 
        }

        switchTurn();
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

    function showPromotionMenu(row, col, color) {
        const whitePieces = ['♕', '♖', '♘', '♗'];
        const blackPieces = ['♛', '♜', '♞', '♝'];
        const pieces = color === 'white' ? whitePieces : blackPieces;

        promoMenu.innerHTML = '';
        pieces.forEach(p => {
            const pieceSpan = document.createElement('span');
            pieceSpan.className = 'promo-piece';
            pieceSpan.innerText = p;
            pieceSpan.onclick = () => selectPromotion(row, col, p);
            promoMenu.appendChild(pieceSpan);
        });
        promoOverlay.style.display = 'flex';
    }

    function selectPromotion(row, col, piece) {
        squares[row][col].innerText = piece;
        promoOverlay.style.display = 'none';
        playMoveSound(); 
        switchTurn();
    }

    // --- కొత్త ఫంక్షన్: ఏదైనా పావు కదిపితే రాజు సేఫ్ గా ఉంటాడా అని ముందుగానే చూడటం ---
    function isSafeMove(fromRow, fromCol, toRow, toCol, color) {
      const originalTargetPiece = squares[toRow][toCol].innerText;
      const movingPiece = squares[fromRow][fromCol].innerText;

      // తత్కాలికంగా కదిపి చూద్దాం (Simulate Move)
      squares[toRow][toCol].innerText = movingPiece;
      squares[fromRow][fromCol].innerText = '';

      const inCheck = isCheck(color); // రాజు సేఫ్ గా ఉన్నాడా?

      // కదిపిన పావును మళ్ళీ వెనక్కి తెద్దాం (Revert Move)
      squares[fromRow][fromCol].innerText = movingPiece;
      squares[toRow][toCol].innerText = originalTargetPiece;

      return !inCheck; // చెక్ లేకపోతే అది సేఫ్
    }

    function tryAddDot(fromRow, fromCol, toRow, toCol, color) {
      if (isSafeMove(fromRow, fromCol, toRow, toCol, color)) {
        if (checkMode) possibleMovesCount++;
        else squares[toRow][toCol].classList.add('dot');
        return true;
      }
      return false;
    }

    function tryAddCapture(fromRow, fromCol, toRow, toCol, color) {
      if (isSafeMove(fromRow, fromCol, toRow, toCol, color)) {
        if (checkMode) possibleMovesCount++;
        else squares[toRow][toCol].classList.add('capture-dot');
        return true;
      }
      return false;
    }
    // ----------------------------------------------------------------------------------

    function showPossibleMoves(piece, row, col) {
      const color = getPieceColor(piece);

      if (piece === '♙') {
        if (row - 1 >= 0 && squares[row - 1][col].innerText === '') {
          tryAddDot(row, col, row - 1, col, color);
          if (row === 6 && squares[row - 2][col].innerText === '') tryAddDot(row, col, row - 2, col, color);
        }
        if (row - 1 >= 0 && col - 1 >= 0 && getPieceColor(squares[row - 1][col - 1].innerText) === 'black') tryAddCapture(row, col, row - 1, col - 1, color);
        if (row - 1 >= 0 && col + 1 < 8 && getPieceColor(squares[row - 1][col + 1].innerText) === 'black') tryAddCapture(row, col, row - 1, col + 1, color);
      }
      
      if (piece === '♟') {
        if (row + 1 < 8 && squares[row + 1][col].innerText === '') {
          tryAddDot(row, col, row + 1, col, color);
          if (row === 1 && squares[row + 2][col].innerText === '') tryAddDot(row, col, row + 2, col, color);
        }
        if (row + 1 < 8 && col - 1 >= 0 && getPieceColor(squares[row + 1][col - 1].innerText) === 'white') tryAddCapture(row, col, row + 1, col - 1, color);
        if (row + 1 < 8 && col + 1 < 8 && getPieceColor(squares[row + 1][col + 1].innerText) === 'white') tryAddCapture(row, col, row + 1, col + 1, color);
      }

      if (piece === '♘' || piece === '♞') {
        const knightMoves = [[-2, -1], [-2, 1], [-1, -2], [-1, 2], [1, -2], [1, 2], [2, -1], [2, 1]];
        knightMoves.forEach(move => {
          const r = row + move[0], c = col + move[1];
          if (r >= 0 && r < 8 && c >= 0 && c < 8) {
            const targetPiece = squares[r][c].innerText;
            if (targetPiece === '') tryAddDot(row, col, r, c, color);
            else if (getPieceColor(targetPiece) !== color) tryAddCapture(row, col, r, c, color);
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
            if (targetPiece === '') tryAddDot(row, col, r, c, color);
            else if (getPieceColor(targetPiece) !== color) tryAddCapture(row, col, r, c, color);
          }
        });
      }
    }

    function checkDirections(startRow, startCol, myColor, directions) {
      directions.forEach(dir => {
        let r = startRow + dir[0], c = startCol + dir[1];
        while (r >= 0 && r < 8 && c >= 0 && c < 8) {
          const targetPiece = squares[r][c].innerText;
          if (targetPiece === '') tryAddDot(startRow, startCol, r, c, myColor);
          else {
            if (getPieceColor(targetPiece) !== myColor) tryAddCapture(startRow, startCol, r, c, myColor);
            break; 
          }
          r += dir[0], c += dir[1];
        }
      });
    }

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

    // --- కొత్త ఫంక్షన్: ఎవరైనా గెలిచారా (చెక్ మేట్) అని కనుక్కోవడం ---
    function isCheckmateOrStalemate(color) {
        possibleMovesCount = 0;
        checkMode = true; // ఈ మోడ్ ఆన్ చేస్తే డాట్స్ పడవు, కేవలం లెక్కపెడుతుంది
        for (let r = 0; r < 8; r++) {
            for (let c = 0; c < 8; c++) {
                const piece = squares[r][c].innerText;
                if (getPieceColor(piece) === color) {
                    showPossibleMoves(piece, r, c);
                }
            }
        }
        checkMode = false;
        return possibleMovesCount === 0; // కదలడానికి ఛాన్స్ లేకపోతే 0 అవుతుంది
    }

    function switchTurn() {
        currentTurn = currentTurn === 'white' ? 'black' : 'white';
        turnIndicator.innerText = `Turn: ${currentTurn.charAt(0).toUpperCase() + currentTurn.slice(1)}`;

        const inCheck = isCheck(currentTurn);

        if (inCheck) {
          checkAlert.style.color = "#ff4c4c";
          checkAlert.innerText = "⚠️ CHECK! ⚠️";
          highlightKing(currentTurn); 
        } else {
          checkAlert.innerText = "";
        }

        // ఎవరి టర్న్ ఉందో వాళ్ళకి మూవ్ చేయడానికి దారి ఉందో లేదో చూడటం
        if (isCheckmateOrStalemate(currentTurn)) {
            gameOver = true;
            let msg = "";
            if (inCheck) {
                let winner = currentTurn === 'white' ? 'Black' : 'White';
                msg = `🏆 CHECKMATE! ${winner} Wins! 🏆\n\nగేమ్ మళ్ళీ మొదటి నుండి స్టార్ట్ చేయమంటారా? (OK నొక్కండి)`;
            } else {
                msg = `🤝 STALEMATE! (మ్యాచ్ డ్రా అయింది)\n\nగేమ్ మళ్ళీ మొదటి నుండి స్టార్ట్ చేయమంటారా? (OK నొక్కండి)`;
            }

            // పావు కదిలిన కొంచెం సేపటికి పాపప్ వచ్చేలా డిలే
            setTimeout(() => {
                // కన్ఫర్మ్ పాపప్ (OK లేదా Cancel ఆప్షన్స్ ఉంటాయి)
                if (confirm(msg)) {
                    restartGame();
                }
            }, 100);
        }
    }
  </script>

</body>
</html>
