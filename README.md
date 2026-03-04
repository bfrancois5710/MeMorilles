<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title> 🍄 MeMorille 🍄</title>

<style>

/* 🌲 FOND FORET */
body {
    font-family: Arial, sans-serif;
    background-image: url("tout-savoir-la-foret.jpg");
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    text-align: center;
    color: white;
    margin: 0;
}

/* Overlay sombre pour lisibilité */
body::before {
    content: "";
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.4);
    z-index: -1;
}

h1 {
    margin-top: 20px;
}

#moves {
    margin-top: 10px;
    font-size: 18px;
}

/* 🏆 IMAGE FINALE */
#winScreen {
    display: none;
    margin-top: 20px;
}

#winScreen img {
    width: 300px;
    border-radius: 20px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.6);
    animation: pop 0.6s ease forwards;
}

@keyframes pop {
    0% { transform: scale(0); opacity: 0; }
    100% { transform: scale(1); opacity: 1; }
}

/* 🎴 PLATEAU */
.game-board {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
    max-width: 900px;
    margin: 40px auto;
}

/* CARTES */
.card {
    width: 100%;
    aspect-ratio: 2 / 3;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.6s;
    cursor: pointer;
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 8px 20px rgba(0,0,0,0.5);
}

.card.flip {
    transform: rotateY(180deg);
}

.front, .back {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
}

/* 🃏 RECTO UNIQUE */
.front {
    background-image: url("Bbm5.jpg");
    background-size: cover;
    background-position: center;
}

/* 🍄 VERSO */
.back {
    transform: rotateY(180deg);
    background-size: cover;
    background-position: center;
}

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

<h1>Me_morille 🍄</h1>
<div id="moves">Coups : 0</div>

<div class="game-board" id="gameBoard"></div>

<button onclick="restartGame()">Recommencer</button>

<!-- IMAGE FINALE -->
<div id="winScreen">
    <h2>🌟 Maître des morilles 🌟</h2>
    <img src="victoire.jpg" alt="Victoire">
</div>

<script>

const images = [
    "1.jpg",
    "2.jpg",
    "3.jpg",
    "4.jpg"
];

let cardsArray = [...images, ...images];
let firstCard, secondCard;
let lockBoard = false;
let moves = 0;

const board = document.getElementById("gameBoard");
const movesDisplay = document.getElementById("moves");
const winScreen = document.getElementById("winScreen");

function shuffle(array) {
    array.sort(() => 0.5 - Math.random());
}

function createBoard() {
    board.innerHTML = "";
    shuffle(cardsArray);
    cardsArray.forEach(image => {
        const card = document.createElement("div");
        card.classList.add("card");
        card.dataset.image = image;

        const front = document.createElement("div");
        front.classList.add("front");

        const back = document.createElement("div");
        back.classList.add("back");
        back.style.backgroundImage = `url(${image})`;

        card.appendChild(front);
        card.appendChild(back);

        card.addEventListener("click", flipCard);
        board.appendChild(card);
    });
}

function flipCard() {
    if (lockBoard) return;
    if (this === firstCard) return;

    this.classList.add("flip");

    if (!firstCard) {
        firstCard = this;
        return;
    }

    secondCard = this;
    moves++;
    movesDisplay.textContent = "Coups : " + moves;

    checkMatch();
}

function checkMatch() {
    let isMatch = firstCard.dataset.image === secondCard.dataset.image;

    if (isMatch) {
        firstCard.removeEventListener("click", flipCard);
        secondCard.removeEventListener("click", flipCard);
        resetBoard();
        checkWin();
    } else {
        lockBoard = true;
        setTimeout(() => {
            firstCard.classList.remove("flip");
            secondCard.classList.remove("flip");
            resetBoard();
        }, 1000);
    }
}

function resetBoard() {
    [firstCard, secondCard, lockBoard] = [null, null, false];
}

function checkWin() {
    const flippedCards = document.querySelectorAll(".flip");
    if (flippedCards.length === cardsArray.length) {
        winScreen.style.display = "block";
    }
}

function restartGame() {
    moves = 0;
    movesDisplay.textContent = "Coups : 0";
    winScreen.style.display = "none";
    createBoard();
}

createBoard();

</script>

</body>
</html>
