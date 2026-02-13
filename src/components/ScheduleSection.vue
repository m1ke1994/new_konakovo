<template>
  <div class="month-schedule">

    <!-- HEADER -->
    <div class="month-schedule__top">
      <h2>Расписание программ</h2>

      <div class="month-schedule__days">
        <button
          v-for="day in days"
          :key="day.dateKey"
          class="month-schedule__day"
          :class="{
            active: activeDate === day.dateKey,
            disabled: day.isPast
          }"
          :disabled="day.isPast"
          @click="activeDate = day.dateKey"
        >
          <span class="month-schedule__day-label">
            {{ day.label }}
          </span>
          <span class="month-schedule__day-number">
            {{ day.day }}
          </span>
        </button>
      </div>
    </div>

    <!-- EVENTS -->
    <div class="month-schedule__list">

      <div
        v-for="event in filteredEvents"
        :key="event.id"
        class="program-card"
        @click="openModal(event)"
      >
        <div
          class="program-card__bar"
          :style="{ background: event.color }"
        ></div>

        <div class="program-card__content">
          <div class="program-card__time">
            {{ event.start }} — {{ event.end }}
          </div>

          <div class="program-card__title">
            {{ event.title }}
          </div>

          <div class="program-card__meta">
            {{ event.category }}
          </div>
        </div>

        <div class="program-card__price">
          {{ formatPrice(event.price) }}
        </div>
      </div>

      <div
        v-if="!filteredEvents.length"
        class="month-schedule__empty"
      >
        В этот день мероприятий нет
      </div>

    </div>

    <!-- MODAL -->
    <div
      v-if="activeEvent"
      class="program-modal"
      @click.self="activeEvent = null"
    >
      <div class="program-modal__card">

        <button
          class="program-modal__close"
          @click="activeEvent = null"
        >
          ✕
        </button>

        <h3>{{ activeEvent.title }}</h3>

        <p class="program-modal__time">
          {{ activeEvent.start }} — {{ activeEvent.end }}
        </p>

        <img
          :src="activeEvent.image"
          class="program-modal__image"
        />

        <p class="program-modal__desc">
          {{ activeEvent.description }}
        </p>

      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed } from "vue";

/* ======= ДНИ МЕСЯЦА ======= */

const today = new Date();
const currentMonth = today.getMonth();
const currentYear = today.getFullYear();

const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate();

const days = [];

for (let i = 1; i <= daysInMonth; i++) {
  const dateObj = new Date(currentYear, currentMonth, i);

  const dateKey = dateObj.toISOString().split("T")[0];

  const isPast =
    dateObj.setHours(0, 0, 0, 0) <
    new Date().setHours(0, 0, 0, 0);

  days.push({
    day: i,
    dateKey,
    isPast,
    label:
      i === today.getDate()
        ? "Сегодня"
        : i === today.getDate() + 1
        ? "Завтра"
        : dateObj.toLocaleDateString("ru-RU", { weekday: "short" })
  });
}

const activeDate = ref(
  today.toISOString().split("T")[0]
);

/* ======= MOCK EVENTS ======= */

const events = ref([
  {
    id: 1,
    date: today.toISOString().split("T")[0],
    title: "Экскурсия в Братство Лосей",
    category: "Экскурсия",
    start: "10:00",
    end: "13:00",
    price: 3500,
    image: "/1.jpeg",
    description: "Погружение в атмосферу заповедника.",
    color: "#E9B949"
  },
  {
    id: 2,
    date: today.toISOString().split("T")[0],
    title: "Мастер-класс по йоге",
    category: "Мастер-класс",
    start: "14:00",
    end: "15:30",
    price: 2200,
    image: "/2.jpeg",
    description: "Практика для начинающих.",
    color: "#6BA368"
  },
  {
    id: 3,
    date: days[2]?.dateKey,
    title: "Беговая встреча",
    category: "Спорт",
    start: "09:00",
    end: "10:00",
    price: 1500,
    image: "/3.jpeg",
    description: "Лёгкая пробежка и техника бега.",
    color: "#C88B3A"
  }
]);

const filteredEvents = computed(() =>
  events.value.filter(e => e.date === activeDate.value)
);

const activeEvent = ref(null);

const openModal = (event) => {
  activeEvent.value = event;
};

const formatPrice = (v) =>
  `${Number(v).toLocaleString("ru-RU")} ₽`;
</script>

<style scoped>
.month-schedule {
  padding: 40px 0;
}

/* HEADER */

.month-schedule__top {
  margin-bottom: 24px;
}

.month-schedule__days {
  display: flex;
  gap: 6px;
  overflow-x: auto;
  padding-bottom: 6px;
}

.month-schedule__day {
  min-width: 64px;
  padding: 6px 8px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: var(--bg-elevated);
  font-size: 11px;
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
}

.month-schedule__day.active {
  background: var(--primary);
  color: var(--color-dark-deep);
  border-color: var(--primary);
}

.month-schedule__day.disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

/* CARDS */

.program-card {
  display: grid;
  grid-template-columns: 6px 1fr auto;
  align-items: center;
  gap: 14px;
  padding: 12px 14px;
  border-radius: 10px;
  border: 1px solid var(--border);
  background: var(--card);
  margin-bottom: 10px;
  cursor: pointer;
}

.program-card__bar {
  height: 100%;
  border-radius: 4px;
}

.program-card__title {
  font-weight: 600;
  margin: 2px 0;
}

.program-card__meta {
  font-size: 12px;
  color: var(--muted);
}

.program-card__price {
  font-weight: 600;
}

/* MODAL */

.program-modal {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.55);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.program-modal__card {
  width: 420px;
  background: var(--card);
  border-radius: 14px;
  padding: 24px;
  position: relative;
}

.program-modal__close {
  position: absolute;
  right: 16px;
  top: 16px;
  border: none;
  background: none;
  cursor: pointer;
}

.program-modal__image {
  width: 100%;
  border-radius: 10px;
  margin: 12px 0;
}
</style>
