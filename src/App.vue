<script setup lang="ts">
import { ref, watch } from "vue";
import GameFigure from "./components/GameFigure.vue";
import GameHeader from "./components/GameHeader.vue";
import GameNotification from "./components/GameNotification.vue";
import GamePopUp from "./components/GamePopUp.vue";
import GameWord from "./components/GameWord.vue";
import GameWrongLetters from "./components/GameWrongLetters.vue";
import { useRandomWord } from "./composables/useRandomWord";
import { useLetters } from "./composables/useLetters";
import { useNotification } from "./composables/useNotification";

const { word, getRandomWord } = useRandomWord();

const { letters, correctLetters, wrongLetters, isLose, isWin, addLetters, resetLetters } =
  useLetters(word);
const { notification, showNotification } = useNotification();
const popup = ref<InstanceType<typeof GamePopUp> | null>(null);
const restart = async () => {
  await getRandomWord();
  resetLetters();
  popup.value?.close();
};

watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open("win");
  }
});

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open("lose");
  }
});

window.addEventListener("keydown", ({ key }) => {
  if (isLose.value || isWin.value) {
    if (key === "Enter") {
      restart();
    }
    return;
  }

  if (letters.value.includes(key)) {
    showNotification();
    return;
  }

  addLetters(key);
});
</script>

<template>
  <GameHeader />
  <div class="game-container">
    <GameFigure :wrong-letters-count="wrongLetters.length" />

    <GameWrongLetters :wrong-letters="wrongLetters" />

    <GameWord :word="word" :correct-letters="correctLetters" />
  </div>

  <!-- Container for final message -->
  <GamePopUp ref="popup" :word="word" @restart="restart" />

  <!-- Notification -->
  <GameNotification ref="notification" />
</template>
