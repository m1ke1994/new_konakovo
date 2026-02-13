<template>
  <div class="services">
    <article v-for="card in cards" :key="card.id" class="card" v-reveal>
      <div class="media">
        <img
          :src="card.image"
          :alt="card.title"
          loading="lazy"
          decoding="async"
          fetchpriority="low"
          width="800"
          height="450"
          class="media__img img-lazy"
          @load="markImageLoaded"
        />
        <div class="media-overlay"></div>
      </div>

      <div class="content">
        <header class="top">
          <h3 class="title">{{ card.title }}</h3>

          <div class="meta">
            <span v-if="card.duration" class="chip">{{ card.duration }}</span>
            <span v-if="card.priceLabel" class="price">{{ card.priceLabel }}</span>
          </div>
        </header>

        <p v-if="card.desc" class="description">
          {{ card.desc }}
        </p>

        <footer class="bottom">
          <button class="btn-outline product-grid__more" type="button" @click="openDetails(card)">
            Подробнее
            <span class="arrow">→</span>
          </button>
        </footer>
      </div>
    </article>
  </div>

  <AppModal
    v-model="isModalOpen"
    :title="activeServiceTitle"
    panel-width="min(1120px, 95vw)"
    panel-max-height="92vh"
  >
    <div v-if="activeService" class="service-modal">
      <section class="service-modal__intro">
        <h2 class="service-modal__h2">{{ activeService.title }}</h2>
        <p class="service-modal__intro-text">{{ activeService.intro }}</p>
      </section>

      <section class="service-modal__section service-modal__section--gallery">
        <h3 class="service-modal__h3">Галерея</h3>

        <div class="service-modal__viewer glass-card">
          <img
            v-if="activeGalleryItem?.type === 'image'"
            :src="activeGalleryItem.src"
            :alt="activeService.title"
            class="service-modal__viewer-image"
            loading="lazy"
            decoding="async"
          />
          <iframe
            v-else
            :src="activeGalleryItem?.src"
            class="service-modal__viewer-video"
            title="Видео услуги"
            loading="lazy"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
            referrerpolicy="strict-origin-when-cross-origin"
            allowfullscreen
          ></iframe>
        </div>

        <div class="service-modal__thumbs">
          <button
            v-for="(item, index) in activeService.gallery"
            :key="`${activeService.id}-${index}`"
            class="service-modal__thumb"
            :class="{ 'service-modal__thumb--active': index === activeMediaIndex }"
            type="button"
            @click="selectGalleryMedia(index)"
          >
            <img
              v-if="item.type === 'image'"
              :src="item.src"
              :alt="`Превью ${index + 1}`"
              loading="lazy"
              decoding="async"
            />
            <span v-else class="service-modal__thumb-video">▶ Видео</span>
          </button>
        </div>
      </section>

      <section class="service-modal__section">
        <h3 class="service-modal__h3">Описание услуги</h3>
        <div
          v-for="(block, blockIndex) in activeService.contentSections"
          :key="`${activeService.id}-content-${blockIndex}`"
          class="service-modal__content-block"
        >
          <h4 class="service-modal__h4">{{ block.title }}</h4>
          <p v-for="(paragraph, pIndex) in block.paragraphs" :key="`${blockIndex}-p-${pIndex}`" class="service-modal__paragraph">
            {{ paragraph }}
          </p>
          <div v-if="block.list?.length" class="service-modal__list-wrap">
            <p v-if="block.listTitle" class="service-modal__list-title">{{ block.listTitle }}</p>
            <ul class="service-modal__list">
              <li v-for="(item, lIndex) in block.list" :key="`${blockIndex}-l-${lIndex}`">{{ item }}</li>
            </ul>
          </div>
        </div>
      </section>

      <section class="service-modal__section">
        <h3 class="service-modal__h3">Тарифы</h3>
        <div class="service-modal__tariffs">
          <article
            v-for="tariff in activeService.tariffs"
            :key="tariff.id"
            class="service-modal__tariff-card"
          >
            <h4 class="service-modal__tariff-title">{{ tariff.title }}</h4>
            <p class="service-modal__tariff-description">{{ tariff.description }}</p>
            <p class="service-modal__tariff-price">{{ formatPrice(tariff.price) }}</p>
            <button class="service-modal__tariff-action btn-secondary" type="button" @click="chooseTariff(tariff.id)">
              Выбрать
            </button>
          </article>
        </div>
      </section>

      <p v-if="activeService.note" class="service-modal__note">
        {{ activeService.note }}
      </p>

      <div class="service-modal__cta">
        <button class="service-modal__btn btn-outline" type="button" @click="closeDetails">Закрыть</button>
      </div>
    </div>
  </AppModal>
</template>

<script setup>
import { computed, ref, watch } from "vue";
import { services } from "../../data/services";
import AppModal from "../ui/AppModal.vue";

const isModalOpen = ref(false);
const activeService = ref(null);
const activeMediaIndex = ref(0);

const formatPrice = (price) => `${Number(price || 0).toLocaleString("ru-RU")} ₽`;

const cards = computed(() =>
  services.map((service) => {
    const minPrice = Math.min(...service.tariffs.map((tariff) => tariff.price));
    return {
      ...service,
      desc: service.description,
      duration: service.durationLabel,
      priceLabel: `от ${formatPrice(minPrice)}`,
    };
  })
);

const activeServiceTitle = computed(() => activeService.value?.fullTitle || activeService.value?.title || "Подробнее");

const activeGalleryItem = computed(() => {
  if (!activeService.value?.gallery?.length) return null;
  return activeService.value.gallery[activeMediaIndex.value] || activeService.value.gallery[0];
});

const selectGalleryMedia = (index) => {
  activeMediaIndex.value = index;
};

const openDetails = (service) => {
  activeService.value = service;
  activeMediaIndex.value = 0;
  isModalOpen.value = true;
};

const closeDetails = () => {
  isModalOpen.value = false;
};

const chooseTariff = (tariffId) => {
  if (typeof window !== "undefined") {
    window.localStorage.setItem("preferredTariffId", String(tariffId));
    const feedbackSection = document.getElementById("feedback");
    feedbackSection?.scrollIntoView({ behavior: "smooth", block: "start" });
  }
  closeDetails();
};

watch(isModalOpen, (value) => {
  if (!value) {
    activeService.value = null;
    activeMediaIndex.value = 0;
  }
});

const markImageLoaded = (event) => {
  event.target.classList.add("is-loaded");
};
</script>

<style scoped>
.services {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 32px;
}

.card {
  position: relative;
  overflow: hidden;
  border-radius: 22px;
  background: var(--card);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  border: 1px solid var(--border);
  box-shadow: 0 18px 40px var(--shadow), inset 0 1px 0 rgba(255, 255, 255, 0.35);
  transition: transform 280ms ease, box-shadow 280ms ease, border-color 280ms ease;
  display: grid;
  grid-template-rows: 180px 1fr;
  min-height: 360px;
}

.card:hover {
  transform: translateY(-4px);
  border-color: var(--border);
  box-shadow: 0 28px 60px var(--shadow), inset 0 1px 0 rgba(255, 255, 255, 0.4);
}

.media {
  position: relative;
  width: 100%;
  height: 180px;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.06);
}

.media__img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transform: scale(1.02);
  transition: transform 350ms ease;
  display: block;
}

.card:hover .media__img {
  transform: scale(1.06);
}

.media-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to bottom, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.1) 55%, rgba(0, 0, 0, 0.18) 100%);
}

.content {
  padding: 16px 16px 18px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.top {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.title {
  margin: 0;
  font-size: 18px;
  line-height: 1.2;
  color: var(--text-strong);
  letter-spacing: 0.2px;
}

.meta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}

.chip {
  display: inline-flex;
  align-items: center;
  padding: 7px 10px;
  border-radius: 999px;
  font-size: 12.5px;
  color: var(--text);
  background: var(--bg-elevated);
  border: 1px solid var(--border);
}

.price {
  font-size: 14px;
  font-weight: 600;
  color: var(--text-strong);
  padding: 7px 10px;
  border-radius: 999px;
  background: var(--primary-soft);
  border: 1px solid var(--border);
}

.description {
  margin: 0;
  font-size: 14px;
  line-height: 1.55;
  color: var(--muted);
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.bottom {
  margin-top: auto;
  display: flex;
  align-items: center;
}

.product-grid__more {
  font-size: 14px;
}

.arrow {
  margin-left: 8px;
}

.service-modal {
  display: grid;
  gap: 22px;
  color: var(--text);
}

.service-modal__section {
  padding-top: 18px;
  border-top: 1px solid color-mix(in srgb, var(--border) 85%, transparent);
}

.service-modal__section--gallery {
  display: grid;
  gap: 14px;
}

.service-modal__h2 {
  margin: 0;
  font-size: clamp(24px, 4vw, 34px);
  line-height: 1.15;
  color: var(--text-strong);
}

.service-modal__h3 {
  margin: 0 0 12px;
  font-size: 18px;
  color: var(--text-strong);
}

.service-modal__h4 {
  margin: 0 0 8px;
  font-size: 16px;
  color: var(--text-strong);
}

.service-modal__intro-text {
  margin: 10px 0 0;
  color: var(--muted);
  line-height: 1.7;
  font-size: 15px;
}

.service-modal__viewer {
  border-radius: 18px;
  overflow: hidden;
}

.service-modal__viewer-image,
.service-modal__viewer-video {
  width: 100%;
  aspect-ratio: 16 / 9;
  border: 0;
  display: block;
  object-fit: cover;
}

.service-modal__thumbs {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 10px;
}

.service-modal__thumb {
  border: 1px solid color-mix(in srgb, var(--border) 80%, transparent);
  border-radius: 12px;
  padding: 0;
  overflow: hidden;
  background: var(--bg-elevated);
  cursor: pointer;
  min-height: 72px;
}

.service-modal__thumb img {
  width: 100%;
  height: 100%;
  min-height: 72px;
  object-fit: cover;
  display: block;
}

.service-modal__thumb-video {
  width: 100%;
  height: 100%;
  min-height: 72px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: var(--text-strong);
  font-size: 13px;
  background: color-mix(in srgb, var(--bg-elevated) 85%, transparent);
}

.service-modal__thumb--active {
  border-color: color-mix(in srgb, var(--primary) 55%, var(--border));
  box-shadow: 0 8px 18px var(--shadow);
}

.service-modal__content-block + .service-modal__content-block {
  margin-top: 16px;
}

.service-modal__paragraph {
  margin: 0 0 10px;
  font-size: 15px;
  line-height: 1.72;
  color: var(--text);
}

.service-modal__list-wrap {
  margin-top: 8px;
}

.service-modal__list-title {
  margin: 0 0 6px;
  font-size: 14px;
  color: var(--text-strong);
}

.service-modal__list {
  margin: 0;
  padding-left: 18px;
  display: grid;
  gap: 6px;
  color: var(--muted);
  font-size: 14px;
  line-height: 1.6;
}

.service-modal__tariffs {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
}

.service-modal__tariff-card {
  background: color-mix(in srgb, var(--bg-elevated) 84%, transparent);
  border: 1px solid color-mix(in srgb, var(--border) 80%, transparent);
  border-radius: 14px;
  padding: 14px;
}

.service-modal__tariff-title {
  margin: 0;
  font-size: 16px;
  color: var(--text-strong);
}

.service-modal__tariff-description {
  margin: 8px 0 10px;
  font-size: 14px;
  line-height: 1.55;
  color: var(--muted);
}

.service-modal__tariff-price {
  margin: 0 0 10px;
  font-size: 18px;
  color: var(--color-dark-deep);
  font-weight: 700;
}

.service-modal__tariff-action {
  width: 100%;
}

.service-modal__note {
  margin: 0;
  padding: 12px 14px;
  border-radius: 12px;
  background: var(--primary-soft);
  border: 1px solid var(--border);
  color: var(--text-strong);
  font-size: 14px;
  line-height: 1.6;
}

.service-modal__cta {
  display: flex;
  justify-content: flex-end;
}

.service-modal__btn {
  font-size: 14px;
}

@media (max-width: 980px) {
  .services {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .service-modal__thumbs {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}

@media (max-width: 780px) {
  .service-modal__tariffs {
    grid-template-columns: 1fr;
  }

  .service-modal__thumbs {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 620px) {
  .services {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  .card {
    grid-template-rows: 160px 1fr;
    min-height: 330px;
    border-radius: 18px;
  }

  .media {
    height: 160px;
  }

  .title {
    font-size: 17px;
  }

  .description {
    font-size: 13.5px;
  }
}
</style>
