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
      overflow: hidden; 
    }
    
    #welcome-screen {
      position: absolute;
      top: 0; left: 0; width: 100vw; height: 100vh;
      background: linear-gradient(135deg, #1e1e1e, #2c2b29, #4a4a4a);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      z-index: 20; 
      text-align: center;
    }

    #welcome-screen h1 {
      font-size: 36px;
      margin-bottom: 10px;
      color: #ebecd0;
      text-shadow: 2px 2px 4px rgba(0,0,0,0.8);
    }

    #welcome-screen p {
      font-size: 18px;
      color: #baca44;
      margin-bottom: 40px;
      text-shadow: 1px 1px 2px rgba(0,0,0,0.8);
    }

    #play-btn {
      padding: 15px 40px;
      font-size: 20px;
      font-weight: bold;
      color: white;
      background-color: #7fa650;
      border: 2px solid #fff;
      border-radius: 25px;
      cursor: pointer;
      box-shadow: 0px 5px 15px rgba(0,0,0,0.5);
      transition: 0.3s;
    }

    #play-btn:hover { background-color: #6a8c42; transform: scale(1.05); }

    #game-container {
      display: none; 
      flex-direction: column;
      align-items: center;
      width: 100%;
      margin-top: 10px;
    }

    h2 { margin-top: 0px; margin-bottom: 5px; font-size: 22px; }
    
    .top-bar {
      display: flex;
      flex-direction: column;
      align-items: center;
      width: 95vw;
      max-width: 400px;
      margin-bottom: 10px;
      gap: 5px;
    }

    .timer-container {
      display: flex;
      justify-content: space-between;
      width: 100%;
    }

    .player-section {
      display: flex;
      flex-direction: column;
      gap: 3px;
    }

    .timer-box {
      background-color: rgba(0, 0, 0, 0.6);
      padding: 5px 10px;
      border-radius: 5px;
      font-size: 16px;
      font-weight: bold;
      border: 2px solid transparent;
      display: flex;
      align-items: center;
      gap: 5px;
    }

    .timer-active {
      border: 2px solid #7fa650; 
      color: #baca44;
    }

    /* --- కొత్తగా యాడ్ చేసిన గ్రేవ్ యార్డ్ డిజైన్ --- */
    .graveyard {
      display: flex;
      flex-wrap: wrap;
      min-height: 24px; /* పావులు లేకపోయినా ప్లేస్ అలానే ఉండటానికి */
      max-width: 180px;
      font-size: 20px;
      text-shadow: 1px 1px 2px black;
      color: #fff;
    }
    
    .dead-piece {
      margin-right: 2px;
    }
    /* ------------------------------------------- */

    .controls-row {
      display: flex;
      justify-content: space-between;
      width: 100%;
      align-items: center;
    }

    #turn-indicator { 
      font-weight: bold; font-size: 16px; color: #fff; 
      background-color: rgba(0,0,0,0.5); padding: 5px 10px; border-radius: 5px;
    }

    #restart-btn {
      padding: 6px 12px; font-size: 14px; font-weight: bold; color: white;
      background-color: #7fa650; border: none; border-radius: 5px; cursor: pointer;
      box-shadow: 0px 4px 6px rgba(0,0,0,0.3);
    }

    #restart-btn:hover { background-color: #6a8c42; }

    #check-alert {
      color: #ff4c4c; font-size: 18px; font-weight: bold; height: 22px; 
      margin-bottom: 5px; text-shadow: 1px 1px 2px black;
    }
    
    #chessboard {
      display: grid;
      grid-template-columns: repeat(8, 1fr);
      grid-template-rows: repeat(8, 1fr);
      width: 95vw; max-width: 400px; height: 95vw; max-height: 400px;
      border: 4px solid #1e1e1e; border-radius: 4px;
      box-shadow: 0px 10px 20px rgba(0,0,0,0.5); position: relative; 
    }
    
    .square {
      display: flex; justify-content: center; align-items: center; font-size: 8vw;
      cursor: pointer; position: relative; text-shadow: 2px 3px 4px rgba(0, 0, 0, 0.6); 
    }

    @media (min-width: 400px) { .square { font-size: 35px; } }

    .white { background-color: #ebecd0; color: #000; }
    .black { background-color: #739552; color: #000; }
    .selected { background-color: #f6f669 !important; } 

    .dot::after {
      content: ''; width: 25%; height: 25%; background-color: rgba(0, 0, 0, 0.3);
      border-radius: 50%; position: absolute;
    }

    .capture-dot::after {
      content: ''; width: 85%; height: 85%; border: 5px solid rgba(255, 0, 0, 0.7);
      border-radius: 50%; position: absolute;
    }

    .in-check { background-color: #ff6666 !important; box-shadow: inset 0 0 10px red; }

    #promotion-overlay {
      display: none; position: absolute; top: 0; left: 0; width: 100%; height: 100%;
      background-color: rgba(0, 0, 0, 0.8); justify-content: center; align-items: center;
      z-index: 10; border-radius: 4px;
    }

    #promotion-menu {
      background-color: white; padding: 20px; border-radius: 10px;
      display: flex; gap: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.5);
    }

    .promo-piece { font-size: 40px; cursor: pointer; color: black; padding: 5px; }
  </style>
</head>
<body>

  <div id="welcome-screen">
    <div style="font-size: 80px; text-shadow: 2px 4px 6px rgba(0,0,0,0.6);">♞</div>
    <h1>Vamshi's Pro Chess</h1>
    <p>Let's Play a Grand Game!</p>
    <button id="play-btn" onclick="startGame()">Play Game</button>
  </div>

  <div id="game-container">
      <h2>Vamshi's Pro Chess</h2>
      
      <div class="top-bar">
        <div class="timer-container">
          <div class="player-section">
            <div id="white-timer" class="timer-box timer-active">♔ White: 10:00</div>
            <div id="white-graveyard" class="graveyard"></div> </div>
          <div class="player-section" style="align-items: flex-end;">
            <div id="black-timer" class="timer-box">♚ Black: 10:00</div>
            <div id="black-graveyard" class="graveyard"></div> </div>
        </div>
        
        <div class="controls-row">
          <div id="turn-indicator">Turn: White</div>
          <button id="restart-btn" onclick="restartGame()">Restart</button>
        </div>
      </div>
      
      <div id="check-alert"></div> 
      <div id="chessboard">
        <div id="promotion-overlay"><div id="promotion-menu"></div></div>
      </div>
  </div>

  <script>
    let audioCtx = null; 

    function initAudio() {
      try {
        if (!audioCtx) {
          const AudioContext = window.AudioContext || window.webkitAudioContext;
          if (AudioContext) audioCtx = new AudioContext();
        }
        if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
      } catch(e) {}
    }

    function playSound(frequency, duration, type) {
      try {
        if(!audioCtx) return;
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
      } catch(e) {}
    }

    function playMoveSound() { playSound(600, 0.1, 'sine'); } 
    function playCaptureSound() { playSound(200, 0.2, 'triangle'); } 
    function playGameOverSound() { playSound(150, 0.5, 'sawtooth'); } 

    let gameActive = false;
    let whiteTime = 600; 
    let blackTime = 600; 
    let timerInterval = null;

    function formatTime(seconds) {
      const m = Math.floor(seconds / 60);
      const s = seconds % 60;
      return `${m}:${s < 10 ? '0' : ''}${s}`;
    }

    function updateTimersUI() {
      const wTimer = document.getElementById('white-timer');
      const bTimer = document.getElementById('black-timer');
      
      wTimer.innerText = `♔ White: ${formatTime(whiteTime)}`;
      bTimer.innerText = `♚ Black: ${formatTime(blackTime)}`;

      if (currentTurn === 'white') {
        wTimer.classList.add('timer-active');
        bTimer.classList.remove('timer-active');
      } else {
        bTimer.classList.add('timer-active');
        wTimer.classList.remove('timer-active');
      }
    }

    function startTimer() {
      if (timerInterval) clearInterval(timerInterval);
      
      timerInterval = setInterval(() => {
        if (!gameActive || gameOver) return;

        if (currentTurn === 'white') {
          whiteTime--;
          if (whiteTime <= 0) handleTimeOut('Black'); 
        } else {
          blackTime--;
          if (blackTime <= 0) handleTimeOut('White'); 
        }
        updateTimersUI();
      }, 1000); 
    }

    function handleTimeOut(winner) {
      gameOver = true;
      gameActive = false;
      clearInterval(timerInterval);
      playGameOverSound();
      
      setTimeout(() => {
        if (confirm(`⏱️ TIME OUT! 🏆 ${winner} Wins!\n\nటైమ్ అయిపోవడం వల్ల ${winner} గెలిచారు. గేమ్ రీస్టార్ట్ చేయమంటారా?`)) {
          restartGame();
        }
      }, 100);
    }

    function startGame() {
        const welcomeScreen = document.getElementById('welcome-screen');
        const gameContainer = document.getElementById('game-container');
        
        welcomeScreen.style.display = 'none';
        gameContainer.style.display = 'flex';
        document.body.style.overflow = 'auto'; 
        
        initAudio();
        playMoveSound(); 
        
        gameActive = true;
        startTimer();
    }

    const boardElement = document.getElementById('chessboard');
    const turnIndicator = document.getElementById('turn-indicator');
    const checkAlert = document.getElementById('check-alert');
    const promoOverlay = document.getElementById('promotion-overlay');
    const promoMenu = document.getElementById('promotion-menu');
    
    // గ్రేవ్ యార్డ్ ఎలిమెంట్స్
    const whiteGraveyard = document.getElementById('white-graveyard');
    const blackGraveyard = document.getElementById('black-graveyard');
    
    let squares = []; 
    let selectedSquare = null; 
    let currentTurn = 'white'; 
    let gameOver = false; 
    let checkMode = false; 
    let possibleMovesCount = 0; 

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
      initAudio();
      gameOver = false; currentTurn = 'white'; selectedSquare = null;
      turnIndicator.innerText = "Turn: White"; checkAlert.innerText = "";
      promoOverlay.style.display = 'none'; removeCheckHighlight();

      // టైమర్స్ రీసెట్
      whiteTime = 600;
      blackTime = 600;
      updateTimersUI();
      gameActive = true;
      startTimer();

      // గ్రేవ్ యార్డ్స్ రీసెట్
      whiteGraveyard.innerHTML = '';
      blackGraveyard.innerHTML = '';

      for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
          squares[row][col].innerText = initialBoard[row][col];
          squares[row][col].className = `square ${(row + col) % 2 === 0 ? 'white' : 'black'}`;
        }
      }
      playMoveSound(); 
    }

    function handleSquareClick(row, col) {
      initAudio();
      if (gameOver || promoOverlay.style.display === 'flex' || !gameActive) return; 

      const clickedSquare = squares[row][col];
      const clickedPiece = clickedSquare.innerText;

      if ((clickedSquare.classList.contains('dot') || clickedSquare.classList.contains('capture-dot')) && selectedSquare) {
        
        let isCapture = clickedSquare.innerText !== '';
        
        // --- పావును చంపినప్పుడు గ్రేవ్ యార్డ్ లోకి యాడ్ చేయడం ---
        if (isCapture) {
            playCaptureSound();
            const capturedPiece = `<span class="dead-piece">${clickedSquare.innerText}</span>`;
            if (currentTurn === 'white') {
                whiteGraveyard.innerHTML += capturedPiece; // వైట్ చంపిన పావులు వైట్ కింద పడతాయి
            } else {
                blackGraveyard.innerHTML += capturedPiece; // బ్లాక్ చంపినవి బ్లాక్ కింద పడతాయి
            }
        } else {
            playMoveSound();
        }
        // --------------------------------------------------------

        movePiece(selectedSquare.row, selectedSquare.col, row, col);
        clearHighlights(); removeCheckHighlight(); selectedSquare = null; 
        
        const movedPiece = squares[row][col].innerText;
        if ((movedPiece === '♙' && row === 0) || (movedPiece === '♟' && row === 7)) {
            showPromotionMenu(row, col, getPieceColor(movedPiece)); return; 
        }

        switchTurn(); return;
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
        const pieces = color === 'white' ? ['♕', '♖', '♘', '♗'] : ['♛', '♜', '♞', '♝'];
        promoMenu.innerHTML = '';
        pieces.forEach(p => {
            const pieceSpan = document.createElement('span');
            pieceSpan.className = 'promo-piece'; pieceSpan.innerText = p;
            pieceSpan.onclick = () => {
                squares[row][col].innerText = p; promoOverlay.style.display = 'none';
                playMoveSound(); switchTurn();
            };
            promoMenu.appendChild(pieceSpan);
        });
        promoOverlay.style.display = 'flex';
    }

    function isSafeMove(fromRow, fromCol, toRow, toCol, color) {
      const originalTargetPiece = squares[toRow][toCol].innerText;
      const movingPiece = squares[fromRow][fromCol].innerText;

      squares[toRow][toCol].innerText = movingPiece; squares[fromRow][fromCol].innerText = '';
      const inCheck = isCheck(color); 
      squares[fromRow][fromCol].innerText = movingPiece; squares[toRow][toCol].innerText = originalTargetPiece;

      return !inCheck; 
    }

    function tryAddDot(fromRow, fromCol, toRow, toCol, color) {
      if (isSafeMove(fromRow, fromCol, toRow, toCol, color)) {
        if (checkMode) possibleMovesCount++; else squares[toRow][toCol].classList.add('dot');
        return true;
      }
      return false;
    }

    function tryAddCapture(fromRow, fromCol, toRow, toCol, color) {
      if (isSafeMove(fromRow, fromCol, toRow, toCol, color)) {
        if (checkMode) possibleMovesCount++; else squares[toRow][toCol].classList.add('capture-dot');
        return true;
      }
      return false;
    }

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
        knightMoves.forEach(m => {
          const r = row + m[0], c = col + m[1];
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
        kingMoves.forEach(m => {
          const r = row + m[0], c = col + m[1];
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
          if (squares[r][c].innerText === kingPiece) { kingRow = r; kingCol = c; break; }
        }
      }
      if (kingRow ===
