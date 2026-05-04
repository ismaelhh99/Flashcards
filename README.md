# Flashcards
Flashcards English - Spanish vocabulary 
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Flashcards Inglés ↔ Español</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
      max-width: 500px;
      margin: auto;
    }

    h1 { margin-bottom: 10px; }

    input, select {
      width: 90%;
      padding: 10px;
      margin: 5px 0;
      font-size: 16px;
    }

    button {
      padding: 10px 15px;
      margin: 5px;
      font-size: 15px;
      cursor: pointer;
    }

    #card {
      padding: 30px;
      border: 2px solid #333;
      margin: 20px auto;
      width: 90%;
      min-height: 120px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 24px;
      cursor: pointer;
      background: #f9f9f9;
      border-radius: 10px;
    }

    .small {
      font-size: 14px;
      color: gray;
    }

    #list {
      margin-top: 20px;
      text-align: left;
      font-size: 14px;
    }

    .row {
      border-bottom: 1px solid #ddd;
      padding: 6px 0;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .row button {
      font-size: 12px;
      padding: 5px 8px;
    }
  </style>
</head>
<body>

<h1>📚 Flashcards Inglés ↔ Español</h1>
<p class="small">Con carpetas (decks) ilimitadas</p>

<hr>

<h3>📁 Crear carpeta (deck)</h3>
<input type="text" id="deckName" placeholder="Nombre de carpeta (ej: Phrasal Verbs)">
<button onclick="createDeck()">Crear carpeta</button>

<hr>

<h3>📌 Seleccionar carpeta</h3>
<select id="deckSelect" onchange="changeDeck()"></select>
<button onclick="deleteDeck()">Eliminar carpeta</button>

<hr>

<h3>➕ Añadir tarjeta</h3>
<input type="text" id="wordEn" placeholder="Palabra en inglés">
<input type="text" id="wordEs" placeholder="Palabra en español">
<button onclick="addCard()">Añadir tarjeta</button>

<hr>

<h3>🎴 Estudio</h3>
<div id="card" onclick="flipCard()">Añade una carpeta y palabras</div>

<button onclick="prevCard()">⬅ Anterior</button>
<button onclick="nextCard()">Siguiente ➡</button>
<br>
<button onclick="shuffleDeck()">🔀 Mezclar</button>
<button onclick="toggleDirection()">🔁 Cambiar modo</button>

<p class="small" id="modeText"></p>

<hr>

<h3>📜 Lista de tarjetas</h3>
<div id="list"></div>

<script>
let data = JSON.parse(localStorage.getItem("flashcardsApp")) || {
  decks: {},
  currentDeck: null,
  mode: "EN_TO_ES"
};

let currentIndex = 0;
let showing = "front";

// Guardar
function save() {
  localStorage.setItem("flashcardsApp", JSON.stringify(data));
}

// Actualizar selector de carpetas
function renderDecks() {
  const select = document.getElementById("deckSelect");
  select.innerHTML = "";

  const deckNames = Object.keys(data.decks);

  if (deckNames.length === 0) {
    select.innerHTML = `<option value="">(No hay carpetas)</option>`;
    data.currentDeck = null;
    save();
    showCard();
    renderList();
    return;
  }

  deckNames.forEach(name => {
    const opt = document.createElement("option");
    opt.value = name;
    opt.textContent = name;
    select.appendChild(opt);
  });

  if (!data.currentDeck || !data.decks[data.currentDeck]) {
    data.currentDeck = deckNames[0];
  }

  select.value = data.currentDeck;
  save();
}

// Crear carpeta
function createDeck() {
  const name = document.getElementById("deckName").value.trim();

  if (!name) {
    alert("Pon un nombre para la carpeta.");
    return;
  }

  if (data.decks[name]) {
    alert("Ya existe una carpeta con ese nombre.");
    return;
  }

  data.decks[name] = [];
  data.currentDeck = name;
  document.getElementById("deckName").value = "";

  save();
  renderDecks();
  currentIndex = 0;
  showing = "front";
  showCard();
  renderList();
}

// Cambiar carpeta
function changeDeck() {
  const select = document.getElementById("deckSelect");
  data.currentDeck = select.value;
  currentIndex = 0;
  showing = "front";
  save();
  showCard();
  renderList();
}

// Eliminar carpeta
function deleteDeck() {
  if (!data.currentDeck) return;

  if (!confirm("¿Seguro que quieres eliminar esta carpeta y todas sus tarjetas?")) return;

  delete data.decks[data.currentDeck];
  data.currentDeck = null;
  currentIndex = 0;
  showing = "front";

  save();
  renderDecks();
  showCard();
  renderList();
}

// Añadir tarjeta
function addCard() {
  if (!data.currentDeck) {
    alert("Primero crea o selecciona una carpeta.");
    return;
  }

  const en = document.getElementById("wordEn").value.trim();
  const es = document.getElementById("wordEs").value.trim();

  if (!en || !es) {
    alert("Rellena inglés y español.");
    return;
  }

  data.decks[data.currentDeck].push({ en, es });

  document.getElementById("wordEn").value = "";
  document.getElementById("wordEs").value = "";

  save();
  renderList();
  showCard();
}

// Mostrar tarjeta actual
function showCard() {
  const cardDiv = document.getElementById("card");
  const modeText = document.getElementById("modeText");

  if (!data.currentDeck) {
    cardDiv.textContent = "Crea una carpeta para empezar";
    modeText.textContent = "";
    return;
  }

  const deck = data.decks[data.currentDeck];

  if (!deck || deck.length === 0) {
    cardDiv.textContent = "Añade palabras a esta carpeta";
    modeText.textContent = "";
    return;
  }

  if (currentIndex < 0) currentIndex = deck.length - 1;
  if (currentIndex >= deck.length) currentIndex = 0;

  const card = deck[currentIndex];

  let front, back;

  if (data.mode === "EN_TO_ES") {
    front = card.en;
    back = card.es;
    modeText.textContent = "Modo: Inglés → Español";
  } else {
    front = card.es;
    back = card.en;
    modeText.textContent = "Modo: Español → Inglés";
  }

  cardDiv.textContent = (showing === "front") ? front : back;
}

// Girar tarjeta
function flipCard() {
  if (!data.currentDeck) return;
  const deck = data.decks[data.currentDeck];
  if (!deck || deck.length === 0) return;

  showing = (showing === "front") ? "back" : "front";
  showCard();
}

// Siguiente
function nextCard() {
  if (!data.currentDeck) return;
  const deck = data.decks[data.currentDeck];
  if (!deck || deck.length === 0) return;

  currentIndex++;
  showing = "front";
  showCard();
}

// Anterior
function prevCard() {
  if (!data.currentDeck) return;
  const deck = data.decks[data.currentDeck];
  if (!deck || deck.length === 0) return;

  currentIndex--;
  showing = "front";
  showCard();
}

// Mezclar deck
function shuffleDeck() {
  if (!data.currentDeck) return;

  const deck = data.decks[data.currentDeck];
  if (!deck || deck.length < 2) return;

  for (let i = deck.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [deck[i], deck[j]] = [deck[j], deck[i]];
  }

  currentIndex = 0;
  showing = "front";
  save();
  showCard();
  renderList();
}

// Cambiar modo
function toggleDirection() {
  data.mode = (data.mode === "EN_TO_ES") ? "ES_TO_EN" : "EN_TO_ES";
  showing = "front";
  save();
  showCard();
}

// Lista de tarjetas
function renderList() {
  const listDiv = document.getElementById("list");

  if (!data.currentDeck) {
    listDiv.innerHTML = "<i>No hay carpeta seleccionada.</i>";
    return;
  }

  const deck = data.decks[data.currentDeck];

  if (!deck || deck.length === 0) {
    listDiv.innerHTML = "<i>No hay tarjetas todavía.</i>";
    return;
  }

  listDiv.innerHTML = "";

  deck.forEach((c, i) => {
    const row = document.createElement("div");
    row.className = "row";

    const text = document.createElement("span");
    text.textContent = `${i + 1}. ${c.en} — ${c.es}`;

    const btn = document.createElement("button");
    btn.textContent = "Eliminar";
    btn.onclick = () => deleteCard(i);

    row.appendChild(text);
    row.appendChild(btn);
    listDiv.appendChild(row);
  });
}

// Eliminar tarjeta
function deleteCard(index) {
  const deck = data.decks[data.currentDeck];
  deck.splice(index, 1);

  if (currentIndex >= deck.length) currentIndex = 0;

  save();
  renderList();
  showCard();
}

// Inicializar
renderDecks();
showCard();
renderList();
</script>

</body>
</html>
