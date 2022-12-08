<script setup>
import { ref, computed } from "vue";
import image from "@/assets/box.png";
import PusyBeauty from "@/assets/pusy-beauty.svg";
import AppFooter from "./components/AppFooter.vue";
import AppImage from "./components/AppImage.vue";

const params = new URLSearchParams(location.search);

const allGiftsDone = ref(false);
const orderId = ref("");
const attempts = ref("");
const boxes = ref(new Array(9).fill().map(() => ({ id: Math.random() })));
const isDisable = ref(false);
const errorPage = ref(false);
const errorMessage = ref("");

const goToCataloge = () => {
  window.location.replace("https://pusy.beauty/");
};

const getAttempts = async () => {
  try {
    const response = await fetch(
      "https://game.pusy.beauty/api/meta?token=" + params.get("token")
    );

    if (response.status !== 200) throw await response.json();

    const { data } = await response.json();

    attempts.value = data?.attempts || 0;
  } catch (error) {
    errorMessage.value = error.message;
    errorPage.value = true;
    console.error(error);
  }
};

const getAllGifts = async () => {
  try {
    const response = await fetch(
      "https://game.pusy.beauty/api/gifts?token=" + params.get("token")
    );
    const { data } = await response.json();
    boxes.value = data.gifts;
    orderId.value = data.orderId;

    allGiftsDone.value = true;
    isDisable.value = true;
  } catch (error) {
    console.error(error);
  }
};

const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const isNextTry = ref(false);

const getGift = async (index) => {
  try {
    isDisable.value = true;
    const response = await fetch("https://game.pusy.beauty/api/run", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        token: params.get("token"),
        index,
      }),
    });
    const { data } = await response.json();
    attempts.value = data.attempts;
    boxes.value = data.products;
    await delay(4000);
    isNextTry.value = true;
  } catch (error) {
    console.error(error);
  }
};
getAttempts();

const countText = computed(() => {
  const lastNumber = attempts.value.toString().substr(-1);

  if (attempts.value >= 11 && attempts.value <= 14) return "попыток";

  if (lastNumber === "1") return "попытка";

  if (lastNumber >= "2" && lastNumber <= "4") return "попытки";

  return "попыток";
});

const nextTry = () => {
  isNextTry.value = false;
  isDisable.value = false;
  boxes.value = new Array(9).fill().map(() => ({ id: Math.random() }));
};
</script>

<template>
  <main class="main">
    <template v-if="!errorPage">
      <div
        class="game"
        :class="{ 'game-over': allGiftsDone }"
        :style="{ '--overflow': allGiftsDone ? 'initial' : 'overflow' }"
      >
        <PusyBeauty />
        <div>
          <div class="count" v-if="!allGiftsDone">
            У вас <span class="count-number" v-text="attempts" />
            {{ countText }}
          </div>
          <div class="all-gifts-done" v-if="allGiftsDone">
            <div class="thank-you">Спасибо за участие!</div>
            <div class="prize">
              Твои призы за заказ:
              <span class="count-number">{{ orderId }}</span>
            </div>
            <button class="button" v-show="allGiftsDone" @click="goToCataloge">
              В Магазин
            </button>
          </div>
          <div
            class="title"
            :style="{
              visibility: attempts != 0 && !allGiftsDone ? 'visible' : 'hidden',
            }"
            v-text="'Выберите один из PÚSY боксов.'"
          />
        </div>
        <div class="list-of-box__wrapper">
          <div
            class="list-of-box"
            :class="{ 'list-of-box--done': allGiftsDone }"
          >
            <button
              :disabled="isDisable"
              v-for="(box, index) in boxes"
              :key="index"
              class="box"
              @click="getGift(index)"
              :class="{ 'box-gift': box?.gift }"
              :style="{ '--i': box?.gift ? 19 : index }"
            >
              <transition mode="out-in" :name="box?.name ? 'list' : undefined">
                <div v-if="box?.image" class="inside-box">
                  <AppImage :src="box.image" />
                  <div class="box-name">{{ box.name }}</div>
                </div>
                <div v-else class="inside-box">
                  <AppImage :src="image" />
                </div>
              </transition>
            </button>
          </div>
        </div>

        <button
          class="button"
          v-if="attempts !== 0"
          :style="{
            visibility: attempts != 0 && isNextTry ? 'visible' : 'hidden',
          }"
          @click="nextTry"
        >
          ИГРАТЬ ЕЩЁ
        </button>
        <button
          class="button"
          v-if="!allGiftsDone && attempts === 0"
          @click="getAllGifts"
        >
          Забрать подарки
        </button>
      </div>
    </template>
    <template v-else>
      <div class="error">
        <h1 class="error-text">{{ errorMessage }}</h1>
        <button class="button" @click="goToCataloge">в магазин</button>
      </div>
    </template>
  </main>
  <AppFooter />
</template>

<style>
#app {
  overflow: var(--overflow) !important;
}
.inside-box {
  position: absolute;
  top: 0;
  left: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  padding: 5px;
  width: 100%;
  height: 100%;
}
.all-gifts-done {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 60px;
}
.error {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.error-text {
  text-align: center;
  margin-top: 20px;
}
.box-name {
  font-family: "Montserrat", sans-serif;
  color: #333333;
  margin-top: 3px;
  font-size: 10px;
  word-break: break-word;
}

.list-enter-active {
  transition: opacity 1s ease;
}
.list-leave-active {
  transition: opacity 1s ease;
  transition-delay: calc(0.1s * var(--i));
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
}

.prize {
  font-family: "Nunito Sans", sans-serif;
  text-align: center;
  font-size: 1rem;
}
.thank-you {
  text-align: center;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: #333333;
  font-size: 30px;
  margin-top: 30px;
  font-weight: 700;
  margin-bottom: 10px;
}
.image {
  object-fit: contain;
  min-height: 0;
  max-width: 100%;
}

.game {
  padding: 15px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  justify-content: space-around;
}

.game-over {
  max-height: initial;
  height: auto;
  background-image: url("./assets/bg-mobile.png");
  background-repeat: no-repeat;
  background-size: contain;
}

.header {
  height: 517px;
  display: flex;
  position: relative;
  display: none;
}

.title {
  text-align: center;
  color: #333333;
  line-height: 1.2;
  font-family: "Nunito Sans", sans-serif;
}

.list-of-box {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  justify-content: center;
  width: 100%;
}
.list-of-box--loading {
  position: relative;
}
.list-of-box--loading::before {
  content: "";
  position: fixed;
  top: 0;
  left: 0;
  backdrop-filter: blur(5px);
  -webkit-backdrop-filter: blur(5px);
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.1);
  z-index: 1;
}

.loader {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 2;
  display: flex;
  place-items: center;
  justify-content: center;
}
.loader::before {
  content: "";
  display: inline-block;
  border: 8px solid #f3f3f3;
  border-top: 8px solid #3498db;
  border-radius: 50%;
  width: 90px;
  height: 90px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.box::before {
  float: left;
  padding-top: 100%;
  content: "";
}

.box::after {
  display: block;
  content: "";
  clear: both;
}

.box {
  border: none;
  border-radius: 15px;
  padding: 0;
  margin: 0;
  cursor: pointer;
  overflow: hidden;
  background-color: #f3f3f3;
  outline: none;
  backface-visibility: hidden;
  position: relative;
  aspect-ratio: 1 / 1;
}
.box-gift {
  box-shadow: 0px 4px 27px rgba(0, 0, 0, 0.2);
  background-color: white;
  border: 3px solid #ff0084;
}

.count {
  text-align: center;
  font-size: 1.5rem;
  line-height: 1.3;
  margin-bottom: 7px;
  text-transform: uppercase;
}
.count-number {
  color: #ff0084;
  text-transform: uppercase;
  font-family: inherit;
}
.button {
  font-family: "Syncopate", sans-serif;
  background-color: #ff0084;
  width: 100%;
  max-width: 310px;
  height: 55px;
  padding: 0;
  text-transform: uppercase;
  border-radius: 3px;
  font-size: 1rem;
  border: none;
  align-self: center;
  color: #fff;
  cursor: pointer;
  letter-spacing: 0.1em;
}

.main {
  display: grid;
  justify-content: center;
}

.list-of-box__wrapper {
  width: 100%;
}

@media (min-width: 1025px) {
  .title {
    font-size: 1.25rem;
  }
  .count {
    font-size: 2.25rem;
  }
  .game-over {
    background-image: none;
  }
  .box-name {
    font-size: 14px;
  }
  .game {
    height: auto;
  }
  .error {
    width: 50vw;
  }
  .list-of-box {
    gap: 20px;
    flex-grow: 1;
    width: auto;
    aspect-ratio: 1 / 1;
    margin: 45px 0px 25px;
  }
  .list-of-box--done {
    aspect-ratio: initial;
    grid-template-columns: repeat(4, 1fr);
    width: 100%;
  }
  .list-of-box__wrapper {
    flex-grow: 1;
  }
}
</style>
