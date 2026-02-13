<template>
  <div class="floating-contact" aria-label="Быстрая связь">
    <a class="floating-contact__btn btn-primary" href="tel:+79990000000">Позвонить</a>
    <a
      class="floating-contact__btn btn-secondary"
      href="https://t.me/username"
      target="_blank"
      rel="noopener"
    >
      Написать
    </a>
  </div>
</template>

<style scoped>
.floating-contact {
  position: fixed;
  right: 16px;
  bottom: max(16px, env(safe-area-inset-bottom));
  z-index: 55;
  display: grid;
  gap: 10px;
}

.floating-contact__btn {
  min-width: 132px;
  box-shadow: 0 10px 20px var(--shadow);
}

@media (max-width: 720px) {
  .floating-contact {
    right: 12px;
    gap: 8px;
  }

  .floating-contact__btn {
    min-width: 118px;
    padding: 9px 12px;
    font-size: 12px;
  }
}
</style>
