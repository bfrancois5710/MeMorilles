<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🍄 MeMorille 🍄</title>

<style>
/* FOND FORET */
body {
    font-family: Arial, sans-serif;
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

/* TITRE & COMPTEUR */
h1 { margin-top: 20px; }
#moves { margin-top: 10px; font-size: 18px; }

/* ÉCRAN DE VICTOIRE */
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
#winScreen img {
    width: 60%;
    max-width: 600px;
    border-radius: 25px;
    box-shadow: 0 20px 50px rgba(0,0,0,0.8);
    animation: zoomIn 0.6s ease forwards;
}
@keyframes zoomIn {
    from { transform: scale(0.7); opacity: 0; }
    to { transform: scale(1); opacity: 1; }
}

/* PLATEAU & CARTES */
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

/* Effet pop lors du flip */
@keyframes popCard {
    0% { transform: scale(0.8); }
    50% { transform: scale(1.1); }
    100% { transform: scale(1); }
}
.card.flip { animation: popCard 0.3s ease; }

/* BOUTON */
button {
    padding: 10px 20px;
    border: none;
    background: #2e3d2f;
    color: white;
    border-radius: 10px;
    cursor: pointer;
    margin-top: 20px;
}
</style>
</head>

<body>

<h1>🍄 MeMorille 🍄</h1>
<div id="moves">Coups : 0</div>

<div class="game-board" id="gameBoard"></div>

<button onclick="restartGame()">Recommencer</button>

<!-- ÉCRAN DE VICTOIRE -->
<div id="winScreen">
    <h2>🌟 Maître Morilles 🌟</h2>
    <img src="victoire.jpg" alt="Victoire">
    <button onclick="restartGame()">Rejouer</button>
</div>

<script>
// Fichiers images
const images = ["1.jpg","2.jpg","3.jpg","4.jpg"];
let cardsArray = [...images, ...images];
let firstCard, secondCard;
let lockBoard = false;
let moves = 0;

// Sélecteurs
const board = document.getElementById("gameBoard");
const movesDisplay = document.getElementById("moves");
const winScreen = document.getElementById("winScreen");

// Son succès
const successSound = new Audio("success.mp3");

// Mélanger
function shuffle(array){ array.sort(() => 0.5 - Math.random()); }

// Créer le plateau
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

// Flip carte
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

// Vérifier paire
function checkMatch(){
    let isMatch = firstCard.dataset.image === secondCard.dataset.image;
    if(isMatch){
        successSound.currentTime = 0;
        successSound.play();
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

// Réinitialiser board
function resetBoard(){ [firstCard, secondCard, lockBoard] = [null,null,false]; }

// Vérifier victoire
function checkWin(){
    const flippedCards = document.querySelectorAll(".flip");
    if(flippedCards.length === cardsArray.length){
        winScreen.style.display = "flex";
    }
}

// Recommencer
function restartGame(){
    moves = 0;
    movesDisplay.textContent = "Coups : 0";
    winScreen.style.display = "none";
    createBoard();
}

// Démarrer
createBoard();
</script>

</body>
</html>
