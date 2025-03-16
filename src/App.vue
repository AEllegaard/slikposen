<script setup>
import { ref, onMounted } from 'vue';

// Reaktive variabler
const data = ref([]);
const selectedItem = ref(null); // Holder styr på det valgte element
const showPopup = ref(false); // Styrer, om popup'en skal vises
const showInitialPopup = ref(true); // Styrer, om den første popup skal vises

// Seeded random generator
function seededRandom(seed) {
  let x = Math.sin(seed++) * 10000;
  return x - Math.floor(x);
}

// Shuffle-funktion med seed
function shuffleArray(array, seed) {
  let shuffled = [...array];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(seededRandom(seed) * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    seed++;
  }
  return shuffled;
}

// Generer tilfældige positioner
function generateRandomPositions(array, seed) {
  const positions = []; // Holder styr på allerede brugte positioner

  return array.map((item) => {
    let randomTop, randomLeft;
    let isOverlapping;

    do {
      // Generer tilfældige positioner
      randomTop = Math.floor(seededRandom(seed) * 80); // Tilfældig top (0-80%)
      randomLeft = Math.floor(seededRandom(seed + 1) * 80); // Tilfældig left (0-80%)
      seed += 2; // Opdater seed

      // Tjek for overlap med eksisterende positioner
      isOverlapping = positions.some(
        (pos) =>
          Math.abs(pos.top - randomTop) < 15 && // Justér 15 for minimum afstand
          Math.abs(pos.left - randomLeft) < 15
      );
    } while (isOverlapping);

    // Gem den nye position
    positions.push({ top: randomTop, left: randomLeft });

    return { ...item, top: `${randomTop}%`, left: `${randomLeft}%` };
  });
}

onMounted(async () => {
  try {
    const response = await fetch('/data.json'); // Hent JSON-filen fra /public
    if (!response.ok) {
      throw new Error('Failed to fetch data.json');
    }
    const fetchedData = await response.json();
    const seed = 61; // Brug et fast seed for at få samme rækkefølge hver gang
    const shuffledData = shuffleArray(fetchedData, seed); // Shuffle dataene
    data.value = generateRandomPositions(shuffledData, seed); // Tilføj tilfældige positioner
    console.log(data.value); // Log dataene for at sikre, at de er korrekt hentet og shufflet
  } catch (error) {
    console.error(error);
  }
});

// Funktion til at håndtere klik på et billede
function handleClick(item) {
  selectedItem.value = item; // Sæt det valgte element
  showPopup.value = true; // Vis popup'en
}

// Funktion til at lukke popup'en
function closePopup() {
  showPopup.value = false; // Skjul popup'en
  selectedItem.value = null; // Nulstil det valgte element
}
</script>

<template>
  <div class="relative h-screen w-screen overflow-hidden left-20 bottom-5">
    <div
      v-for="(item, index) in data"
      :key="index"
      class="absolute item-container"
      :style="{ top: item.top, left: item.left }"
      @click="handleClick(item)"
    >
      <img
        :width="item.bredde"
        :src="item.slikimg"
        :alt="item.sliknavn"
        class="img-shad hover:rotate-15 hover:scale-110"
      />
    </div>
    <img src="/media/logo.png" alt="" class="absolute w-30 right-25 bottom-0">

    <!-- Popup til at vise detaljer om det valgte element -->
    <div v-if="showPopup" class="popup">
      <div class="popup-content font-candy w-100">
        <h2 class="text-2xl text-pink">{{ selectedItem.navn }}</h2>
        <p><span class="text-orange">{{ selectedItem.sliknavn }}:</span> <br>{{ selectedItem.beskrivelse }}</p>
        <button @click="closePopup">Luk</button>
      </div>
    </div>
  </div>

  <transition name="popup-fade">
  <div v-if="showInitialPopup" class="popup">
    <div class="popup-content font-candy w-150">
      <h2 class="text-xl text-pink">Kæreste Karsten</h2>
      <p class="text-left"> <br>
      Vi er så taknemmlige for at du har turde at tage springet med os i denne digitale verden. 
    vi værdsætter det arbejde du har lagt i os og den støtte du har viser os, når vi synes det er svært. <br><br>
    Vi har lavet en helt særlig pose slik til dig. Den er fyldt med det vi elsker allermest.
    <br> <br>
    De sukkersødeste hilsner <br>
    CD2 </p>
      <button @click="showInitialPopup = false">Luk</button>
    </div>
  </div>
</transition>

</template>

<style scoped>
/* Popup animation */
.popup-fade-enter-active,
.popup-fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease-out;
}

.popup-fade-enter-from,
.popup-fade-leave-to {
  opacity: 0;
  transform: scale(0.8); /* Starter lidt mindre */
}

.popup-fade-enter-to,
.popup-fade-leave-from {
  opacity: 1;
  transform: scale(1); /* Vokser til normal størrelse */
}

/* Resten af din CSS */
.img-shad {
  filter: drop-shadow(5px 5px 10px rgba(0, 0, 0, 0.2)); /* Tilføjer en skygge */
}

.item-container {
  transition: transform 0.3s ease; /* Tilføjer en smooth hover-effekt */
  transform: translateY(-20px); /* Rykker billederne 20px op */
}

.popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  padding: 20px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.3);
  z-index: 1000;
  border-radius: 15px;
}

.popup-content {
  text-align: center;
  animation: fadeIn 0.3s ease-out;
}

.popup-content h2 {
  margin-bottom: 10px;
}

.popup-content p {
  margin-bottom: 20px;
}

.popup-content button {
  padding: 8px 25px;
  background-color: #e321ac;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 20px;
  transition: 0.3s ease;
}

.popup-content button:hover {
  background-color: #ffb33f;
}

/* Ekstra blød fade-in */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
</style>