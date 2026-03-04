<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🍄 MeMorille 🍄</title>

<!-- Google Font si tu veux style Impact -->
<link href="https://fonts.googleapis.com/css2?family=Anton&display=swap" rel="stylesheet">

<style>
body {
    font-family: 'Anton', sans-serif;
    background-image: url("tout-savoir-sur-la-foret.jpg");
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    text-align: center;
    color: white;
    margin: 0;
}
body::before {
    content: "";
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.4);
    z-index: -1;
}

h1 { margin-top: 20px; font-family: 'Anton', sans-serif; }
#moves { margin-top: 10px; font-size: 18px; }
#bestScore { margin-top: 5px; font-size:16px; }

.game-board {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
    max-width: 900px;
    margin: 40px auto;
    perspective: 1000px;
}

.card {
    width: 100%;
    aspect-ratio: 2/3;
    position: relative;
    transform-style: preserve-3d;
    cursor: pointer;
}
.card img {
    position: absolute;
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 20px;
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    transition: transform 0.6s;
}
.front-img { transform: rotateY(0deg); z-index:2; }
.back-img { transform: rotateY(180deg); }
.card.flip img.front-img { transform: rotateY(180deg); }
.card.flip img.back-img { transform: rotateY(0deg); }

#winScreen {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.85);
    backdrop-filter: blur(5px);
    justify-content: center;
    align-items: center;
    flex-direction: column;
    z-index: 999;
}
#winScreen img { width:60%; max-width:600px; border-radius:25px; margin-bottom:10px; }

button {
    padding: 10px 20px;
    border: none;
    background: #2e3d2f;
    color: white;
    border-radius: 10px;
    cursor: pointer;
    margin-top: 10px;
}

#topScores {
    margin-top: 20px;
    font-size: 16px;
    text-align: center;
}
</style>
</head>
<body>

<h1>🍄 MeMorille 🍄</h1>
<div>
    Pseudo : <input type="text" id="pseudoInput" placeholder="Ton pseudo">
</div>
<div id="moves">Coups : 0</div>
<div id="bestScore">Meilleur score local : -</div>
<div class="game-board" id="gameBoard"></div>
<button onclick="restartGame()">Recommencer</button>

<!-- Top 10 -->
<div id="topScores">
    <h3>Top 10 joueurs :</h3>
    <ul id="scoreList"></ul>
</div>

<!-- Écran victoire -->
<div id="winScreen">
    <h2>🌟 Maître Morilles 🌟</h2>
    <img src="victoire.jpg" alt="Victoire">
    <button onclick="restartGame()">Rejouer</button>
</div>

<!-- Firebase -->
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
  import { getDatabase, ref, push, set, query, orderByChild, limitToFirst, get } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-database.js";

  const firebaseConfig = {
    apiKey: "AIzaSyDiykIz3KPCpNdQeBW_6lP5mXp_si8vKRo",
    authDomain: "memorilles-game.firebaseapp.com",
    projectId: "memorilles-game",
    storageBucket: "memorilles-game.firebasestorage.app",
    messagingSenderId: "837717883602",
    appId: "1:837717883602:web:f02fb044cafe4fec674b0b",
    measurementId: "G-PP7CES97GC"
  };

  const app = initializeApp(firebaseConfig);
  const database = getDatabase(app);

  const images = ["1.jpg","2.jpg","3.jpg","4.jpg"];
  let cardsArray = [...images, ...images];
  let firstCard, secondCard;
  let lockBoard = false;
  let moves = 0;

  const board = document.getElementById("gameBoard");
  const movesDisplay = document.getElementById("moves");
  const winScreen = document.getElementById("winScreen");
  const scoreList = document.getElementById("scoreList");

  let bestScore = localStorage.getItem("bestScore") || Infinity;
  const bestScoreDiv = document.getElementById("bestScore");
  bestScoreDiv.textContent = bestScore === Infinity ? "Meilleur score local : -" : "Meilleur score local : " + bestScore;

  function shuffle(array){ array.sort(() => 0.5 - Math.random()); }

  function createBoard(){
    board.innerHTML = "";
    shuffle(cardsArray);
    cardsArray.forEach(image=>{
        const card = document.createElement("div");
        card.classList.add("card");
        card.dataset.image = image;

        const frontImg = document.createElement("img");
        frontImg.classList.add("front-img");
        frontImg.src = "bbm7jpg.JPG";

        const backImg = document.createElement("img");
        backImg.classList.add("back-img");
        backImg.src = image;

        card.appendChild(frontImg);
        card.appendChild(backImg);

        card.addEventListener("click", flipCard);
        board.appendChild(card);
    });
  }

  function flipCard(){
    if(lockBoard) return;
    if(this === firstCard) return;
    this.classList.add("flip");
    if(!firstCard){ firstCard = this; return; }
    secondCard = this;
    moves++;
    movesDisplay.textContent = "Coups : " + moves;
    checkMatch();
  }

  function checkMatch(){
    let isMatch = firstCard.dataset.image === secondCard.dataset.image;
    if(isMatch){
        firstCard.removeEventListener("click", flipCard);
        secondCard.removeEventListener("click", flipCard);
        resetBoard();
        checkWin();
    } else {
        lockBoard = true;
        setTimeout(()=>{
            firstCard.classList.remove("flip");
            secondCard.classList.remove("flip");
            resetBoard();
        },1000);
    }
  }

  function resetBoard(){ [firstCard, secondCard, lockBoard] = [null,null,false]; }

  async function checkWin(){
    const flippedCards = document.querySelectorAll(".flip");
    if(flippedCards.length === cardsArray.length){
        winScreen.style.display = "flex";
        const pseudo = document.getElementById("pseudoInput").value || "Anonyme";

        // Enregistrer score sur Firebase
        const scoresRef = ref(database,'scores');
        const newScoreRef = push(scoresRef);
        set(newScoreRef,{
            pseudo: pseudo,
            score: moves,
            date: new Date().toISOString()
        });

        // Enregistrer score local
        if(moves < bestScore){
            bestScore = moves;
            localStorage.setItem("bestScore", bestScore);
            bestScoreDiv.textContent = "Meilleur score local : " + bestScore;
        }

        // Mettre à jour Top 10
        await updateTopScores();
    }
  }

  async function updateTopScores(){
    const scoresRef = query(ref(database,'scores'), orderByChild('score'), limitToFirst(10));
    const snapshot = await get(scoresRef);
    const scores = snapshot.val();
    scoreList.innerHTML = "";
    if(scores){
        Object.values(scores).forEach(s=>{
            const li = document.createElement("li");
            li.textContent = `${s.pseudo} : ${s.score} coups`;
            scoreList.appendChild(li);
        });
    }
  }

  function restartGame(){
    moves = 0;
    movesDisplay.textContent = "Coups : 0";
    winScreen.style.display = "none";
    createBoard();
  }

  createBoard();
  updateTopScores();
</script>

</body>
</html>
