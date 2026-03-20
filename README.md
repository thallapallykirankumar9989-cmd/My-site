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
      position: relative; /* ప్రమోషన్ ఓవర్లే కోసం */
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

    /* పాన్ ప్రమోషన్ పాపప్ (Modal) డిజైన్ */
    #promotion-overlay {
      display: none; /* మొదట దాచిపెట్టాలి */
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

  <h2>Vamshi's Pro Chess</h2>
  
  <div class="top-bar">
    <div id="turn-indicator">Turn: White</div>
    <button id="restart-btn" onclick="restartGame()">Restart</button>
  </div>
  
  <div id="check-alert"></div> 
  
  <div id="chessboard">
    <div id="promotion-overlay">
        <div id="promotion-menu">
            </div>
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
    
    // సౌండ్ ఎఫెక్ట్స్ కోసం ఆడియో కంటెక్స్ట్ (కోడింగ్ ద్వారా సౌండ్ పుట్టించడానికి)
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

    // --- కొత్తగా యాడ్ చేసిన సౌండ్ ఫంక్షన్స్ ---
    function playSound(frequency, duration, type) {
      const oscillator = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();
      oscillator.type = type; // 'sine', 'square', 'sawtooth', 'triangle'
      oscillator.frequency.setValueAtTime(frequency, audioCtx.currentTime);
      gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime); // వాల్యూమ్ తక్కువగా
      gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
      oscillator.connect(gainNode);
      gainNode.connect(audioCtx.destination);
      oscillator.start();
      oscillator.stop(audioCtx.currentTime + duration);
    }

    function playMoveSound() { playSound(600, 0.1, 'sine'); } // చిన్న 'టిక్' సౌండ్
    function playCaptureSound() { playSound(200, 0.2, 'triangle'); } // కొంచెం బరువైన సౌండ్
    // ------------------------------------------

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
      // ఆడియో కంటెక్స్ట్ కొన్నిసార్లు సస్పెండ్ అవుతుంది, రీస్టార్ట్ అప్పుడు రిజ్యూమ్ చేద్దాం
      if (audioCtx.state === 'suspended') audioCtx.resume();
      
      gameOver = false;
      currentTurn = 'white';
      selectedSquare = null;
      turnIndicator.innerText = "Turn: White";
      checkAlert.innerText = "";
      checkAlert.style.color = "#ff4c4c";
      promoOverlay.style.display = 'none'; // పాపప్ దాచేయి

      for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
          squares[row][col].innerText = initialBoard[row][col];
          squares[row][col].className = `square ${(row + col) % 2 === 0 ? 'white' : 'black'}`;
        }
      }
      playMoveSound(); // గేమ్ స్టార్ట్ అయినట్లు సౌండ్
    }

    function handleSquareClick(row, col) {
      // ఆడియో కంటెక్స్ట్ యూజర్ క్లిక్ ద్వారా యాక్టివేట్ అవ్వాలి
      if (audioCtx.state === 'suspended') audioCtx.resume();
      
      if (gameOver || promoOverlay.style.display === 'flex') return; // ప్రమోషన్ మెనూ ఉంటే క్లిక్ అవ్వద్దు

      const clickedSquare = squares[row][col];
      const clickedPiece = clickedSquare.innerText;

      // పావును కదిలించడానికి లేదా చంపడానికి క్లిక్ చేస్తే
      if ((clickedSquare.classList.contains('dot') || clickedSquare.classList.contains('capture-dot')) && selectedSquare) {
        
        let isCapture = clickedSquare.innerText !== '';
        let isCheckmate = false;
        let winnerName = "";

        if (clickedPiece === '♚') { isCheckmate = true; winnerName = "White"; }
        else if (clickedPiece === '♔') { isCheckmate = true; winnerName = "Black"; }

        // సౌండ్ ప్లే చేయడం
        if (isCapture) playCaptureSound();
        else playMoveSound();

        // పావును మూవ్ చేయడం
        movePiece(selectedSquare.row, selectedSquare.col, row, col);
        clearHighlights();
        removeCheckHighlight(); 
        selectedSquare = null; 
        
        if (isCheckmate) {
          gameOver = true;
          setTimeout(() => {
            alert(`Game Over! 🏆 ${winnerName} Wins! \n\nగేమ్ మళ్ళీ మొదటి నుండి స్టార్ట్ అవుతుంది.`);
            restartGame(); 
          }, 100);
          return; 
        }

        // --- కొత్తగా యాడ్ చేసిన ప్రమోషన్ చెక్ ---
        const movedPiece = squares[row][col].innerText;
        // తెలుపు భటుడు 0 లైన్ కి లేదా నలుపు భటుడు 7 లైన్ కి వెళ్తే
        if ((movedPiece === '♙' && row === 0) || (movedPiece === '♟' && row === 7)) {
            showPromotionMenu(row, col, getPieceColor(movedPiece));
            return; // టర్న్ మార్చొద్దు, ప్రమోషన్ అయ్యాక మారుస్తాం
        }
        // ------------------------------------

        // మామూలు మూవ్ అయితే టర్న్ మార్చు
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

    // --- కొత్తగా యాడ్ చేసిన ప్రమోషన్ మెనూ లాజిక్ ---
    function showPromotionMenu(row, col, color) {
        // ప్రమోషన్ పావులు (రంగును బట్టి)
        const whitePieces = ['♕', '♖', '♘', '♗'];
        const blackPieces = ['♛', '♜', '♞', '♝'];
        const pieces = color === 'white' ? whitePieces : blackPieces;

        // మెనూ క్లియర్ చేసి కొత్త పావులు పెట్టడం
        promoMenu.innerHTML = '';
        pieces.forEach(p => {
            const pieceSpan = document.createElement('span');
            pieceSpan.className = 'promo-piece';
            pieceSpan.innerText = p;
            pieceSpan.onclick = () => selectPromotion(row, col, p);
            promoMenu.appendChild(pieceSpan);
        });

        // పాపప్ చూపించు
        promoOverlay.style.display = 'flex';
    }

    function selectPromotion(row, col, piece) {
        // ఎంచుకున్న పావుని బోర్డ్ మీద పెట్టు
        squares[row][col].innerText = piece;
        // పాపప్ దాచేయి
        promoOverlay.style.display = 'none';
        playMoveSound(); // ప్రమోషన్ అయినట్లు సౌండ్
        // ప్రమోషన్ అయ్యాక టర్న్ మార్చు
        switchTurn();
    }
    // ----------------------------------------------

    // టర్న్ మార్చే లాజిక్ ని ఒక ఫంక్షన్ లో పెట్టాను
    function switchTurn() {
        currentTurn = currentTurn === 'white' ? 'black' : 'white';
        turnIndicator.innerText = `Turn: ${currentTurn.charAt(0).toUpperCase() + currentTurn.slice(1)}`;

        if (isCheck(currentTurn)) {
          checkAlert.style.color = "#ff4c4c";
          checkAlert.innerText = "⚠️ CHECK! ⚠️";
          highlightKing(currentTurn); 
        } else {
          checkAlert.innerText = "";
        }
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
