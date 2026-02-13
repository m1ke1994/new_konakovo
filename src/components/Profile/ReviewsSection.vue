<template>
  <div
    ref="reviewsRef"
    class="reviews"
    v-reveal
    @mouseenter="pause"
    @mouseleave="resume"
    @focusin="pause"
    @focusout="resume"
  >
    <div class="reviews__card-wrap">
      <Transition name="fade-rise" mode="out-in">
        <article
          class="reviews__card glass-card"
          :key="activeReview.id"
          tabindex="0"
        >
          <div class="reviews__avatar">
            <img
              :src="activeReview.avatarUrl"
              :alt="`Фото — ${activeReview.name}`"
              loading="lazy"
              decoding="async"
              fetchpriority="low"
              width="64"
              height="64"
              class="img-lazy"
              @load="markImageLoaded"
            />
          </div>

          <div class="reviews__stars" aria-label="Рейтинг">
            <span
              v-for="n in 5"
              :key="n"
              :class="['star', { muted: n > activeReview.rating }]"
              aria-hidden="true"
            >
              ★
            </span>
          </div>

          <p class="reviews__text">«{{ activeReview.text }}»</p>

          <div class="reviews__name">{{ activeReview.name }}</div>
          <div class="reviews__meta">{{ activeReview.service }} · {{ activeReview.date }}</div>
        </article>
      </Transition>
    </div>

    <div class="reviews__nav">
      <button class="nav-btn" type="button" aria-label="Предыдущий отзыв" @click="prev">
        ‹
      </button>

      <div class="dots" role="tablist" aria-label="Слайды">
        <button
          v-for="(review, idx) in reviews"
          :key="review.id"
          class="dot"
          :class="{ active: idx === activeIndex }"
          type="button"
          @click="goTo(idx)"
          :aria-label="`Слайд ${idx + 1}`"
        ></button>
      </div>

      <button class="nav-btn" type="button" aria-label="Следующий отзыв" @click="next">
        ›
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref } from "vue"

const reviews = [
  {
    id: 1,
    name: "Мария К.",
    text:
      "Экскурсия в «Братство лосей» — очень тёплый и спокойный опыт. Всё организовано бережно, без спешки, с вниманием к деталям.",
    rating: 5,
    service: "Экскурсия",
    date: "15 октября 2024",
    avatarUrl: "/avatar.png",
  },
  {
    id: 2,
    name: "Екатерина Д.",
    text:
      "Утренние практики помогли перезагрузиться и настроиться на день. После занятий появляется ясность и спокойствие — очень рекомендую.",
    rating: 5,
    service: "Утренние практики",
    date: "28 сентября 2024",
    avatarUrl: "/avatar.png",
  },
  {
    id: 3,
    name: "Алина Р.",
    text:
      "Чайная церемония — как маленький ритуал тишины и уюта. Атмосфера мягкая, уютная, хочется возвращаться снова.",
    rating: 5,
    service: "Чайная церемония",
    date: "7 ноября 2024",
    avatarUrl: "/avatar.png",
  },
  {
    id: 4,
    name: "Игорь С.",
    text:
      "Мастер-класс прошёл легко и интересно. Понравился подход: спокойно, дружелюбно и при этом очень содержательно.",
    rating: 4,
    service: "Мастер-класс",
    date: "22 августа 2024",
    avatarUrl: "/avatar.png",
  },
  {
    id: 5,
    name: "Ольга Н.",
    text:
      "Игра «Лила» оказалась глубже, чем я ожидала. Без давления, с уважением и поддержкой — очень ценный опыт.",
    rating: 5,
    service: "Игра «Лила»",
    date: "2 июля 2024",
    avatarUrl: "/avatar.png",
  },
]

const activeIndex = ref(0)
const isPaused = ref(false)
const reviewsRef = ref(null)
let timerId = null
let visibilityObserver = null

const activeReview = computed(() => reviews[activeIndex.value])

const next = () => {
  activeIndex.value = (activeIndex.value + 1) % reviews.length
  restart()
}

const prev = () => {
  activeIndex.value = (activeIndex.value - 1 + reviews.length) % reviews.length
  restart()
}

const goTo = (idx) => {
  activeIndex.value = idx
  restart()
}

const start = () => {
  stop()
  timerId = setInterval(() => {
    if (!isPaused.value) {
      activeIndex.value = (activeIndex.value + 1) % reviews.length
    }
  }, 4500)
}

const stop = () => {
  if (timerId) {
    clearInterval(timerId)
    timerId = null
  }
}

const pause = () => {
  isPaused.value = true
}

const resume = () => {
  isPaused.value = false
}

const restart = () => {
  stop()
  start()
}

const markImageLoaded = (event) => {
  event.target.classList.add("is-loaded")
}

onMounted(() => {
  if (typeof window === "undefined") return
  if (!("IntersectionObserver" in window) || !reviewsRef.value) {
    start()
    return
  }
  visibilityObserver = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting) {
        start()
      } else {
        stop()
      }
    },
    { rootMargin: "200px 0px", threshold: 0.01 }
  )
  visibilityObserver.observe(reviewsRef.value)
})

onUnmounted(() => {
  stop()
  visibilityObserver?.disconnect()
})
</script>

<style scoped>
.reviews {
  width: 100%;
}

.reviews__card-wrap {
  display: flex;
  justify-content: center;
}

.reviews__card {
  width: min(900px, 100%);
  padding: 28px 30px;
  display: grid;
  justify-items: center;
  gap: 14px;
  text-align: center;
}

.glass-card {
  border-radius: 26px;
}

.reviews__avatar {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background: var(--card);
  border: 1px solid var(--border);
  box-shadow: 0 10px 20px var(--shadow);
  overflow: hidden;
}

.reviews__avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.reviews__stars {
  display: inline-flex;
  gap: 6px;
  font-size: 16px;
  color: var(--primary);
}

.star.muted {
  color: rgba(200, 169, 107, 0.35);
}

.reviews__text {
  font-size: 16px;
  font-style: italic;
  color: var(--text);
  margin: 0;
  line-height: 1.6;
}

.reviews__name {
  font-weight: 600;
  color: var(--text-strong);
}

.reviews__meta {
  font-size: 13px;
  color: var(--muted);
}

.reviews__nav {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-top: 24px;
}

.nav-btn {
  width: 42px;
  height: 42px;
  border-radius: 50%;
  border: 1px solid var(--border);
  background: var(--card);
  backdrop-filter: blur(12px);
  cursor: pointer;
  color: var(--text);
  box-shadow: 0 10px 20px var(--shadow);
  transition: transform 200ms ease, box-shadow 200ms ease;
  font-size: 20px;
  line-height: 1;
}

.nav-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 12px 24px var(--shadow);
}

.dots {
  display: inline-flex;
  gap: 8px;
}

.dot {
  width: 8px;
  height: 8px;
  border-radius: 999px;
  border: none;
  background: rgba(200, 169, 107, 0.35);
  cursor: pointer;
  transition: width 200ms ease, background 200ms ease;
}

.dot.active {
  width: 18px;
  background: rgba(200, 169, 107, 0.9);
}

.fade-rise-enter-active,
.fade-rise-leave-active {
  transition: opacity 350ms ease, transform 350ms ease;
}

.fade-rise-enter-from,
.fade-rise-leave-to {
  opacity: 0;
  transform: translateY(10px) scale(0.98);
}

@media (max-width: 720px) {
  .reviews__card {
    padding: 22px 18px;
  }
}
</style>
