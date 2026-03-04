<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🍄 Me_morilles 🍄</title>

<style>
/* BODY & FOND */
body {
    font-family: IMPACT, sans-serif;
    background: linear-gradient(180deg, #2e3d2f, #151a16); /* tout-savoir-sur-la-foret.jpg *
    color: white;
    text-align: center;
    margin: 0;
    padding: 0;
}

/* TITRE & PARAGRAPHE */
h1 {
    margin-top: 20px;
    letter-spacing: 2px;
}

p {
    margin-bottom: 10px;
}

/* COMPTEUR */
#moves {
    margin-top: 10px;
    font-size: 18px;
}

/* MESSAGE DE VICTOIRE */
#winMessage {
    display: none;
    font-size: 28px;
    margin-top: 20px;
    color: #ffd700;
    animation: pop 0.5s ease forwards;
}

@keyframes pop {
    0% { transform: scale(0); opacity: 0; }
    50% { transform: scale(1.2); opacity: 1; }
    100% { transform: scale(1); opacity: 1; }
}

/* PLATEAU */
.game-board {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 15px;
    max-width: 900px;
    margin: 30px auto;
}

/* CARTES */
.card {
    width: 100%;
    aspect-ratio: 2 / 3; /* cartes verticales */
    cursor: pointer;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.5s;
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 8px 20px rgba(0,0,0,0.4);
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

/* RECTO : image dos de carte */
.front {
    background-image: url("mm5.jpg"); /* ton image Canva pour le recto */
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
}

/* VERSO : morille */
.back {
    transform: rotateY(180deg);
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
}

/* BOUTON */
button {
    margin-top: 20px;
    padding: 10px 20px;
    border: none;
    background: #3e4e1f;
    color: white;
    border-radius: 10px;
    cursor: pointer;
    font-size: 16px;
}

</style>
</head>

<body>

<h1>Me_morille 🍄</h1>
<p>Retrouve les paires de morilles !</p>

<div id="moves">Coups : 0</div>

<!-- MESSAGE DE VICTOIRE -->
<div id="winMessage"></div>

<div class="game-board" id="gameBoard"></div>

<button onclick="restartGame()">Recommencer</button>

<script>
// 🎨 IMAGES DES MORILLES (JPG que tu as uploadé)
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
const winMessage = document.getElementById("winMessage");

// Mélanger les cartes
function shuffle(array) {
    array.sort(() => 0.5 - Math.random());
}

// Créer le plateau
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

// Retourner la carte
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

// Vérifier si c'est une paire
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

// Réinitialiser le plateau après une paire
function resetBoard() {
    [firstCard, secondCard, lockBoard] = [null, null, false];
}

// Vérifier victoire
function checkWin() {
    const flippedCards = document.querySelectorAll(".flip");
    if (flippedCards.length === cardsArray.length) {
        winMessage.textContent = "🎉 Bravo ! Tu as trouvé toutes les morilles ! 🍄✨";
        winMessage.style.display = "block";
    }
}

// Recommencer le jeu
function restartGame() {
    moves = 0;
    movesDisplay.textContent = "Coups : 0";
    winMessage.style.display = "none";
    createBoard();
}

// Démarrer le jeu
createBoard();

</script>

</body>
</html>

    transform-style: preserve-3d;
    transition: transform 0.5s;
