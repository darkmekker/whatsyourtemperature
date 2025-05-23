<template>
  <nav class="fixed z-50 top-0 inset-x-0" aria-label="Main Menu">
    <div class="menu-container" :class="{ 'menu-open': isMenuOpen }">
      <div class="hamburger" @click="toggleMenu">
        <div class="line rounded"></div>
        <div class="line rounded"></div>
        <div class="line rounded"></div>
      </div>
      <div class="menu-panel relative">
        <ul class="menu-items text-lg" @click="toggleMenu">
          <li class="flex-1">
            <nuxt-link class="block text-white p-2" to="/">Home</nuxt-link>
          </li>
          <li class="flex-1">
            <nuxt-link class="block text-white p-2" to="/about">About Promeos</nuxt-link>
          </li>
          <li class="flex-1">
            <nuxt-link class="block text-white p-2" to="/vision">Vision</nuxt-link>
          </li>
          <li class="flex-1">
            <nuxt-link class="block text-white p-2" to="/futureproof">Future Proof</nuxt-link>
          </li>
          <li class="flex-1">
            <nuxt-link class="block text-white p-2" to="/contact">Contact</nuxt-link>
          </li>
        </ul>
        <div class="absolute bottom-5 right-0 w-full flex text-xs px-5 justify-end gap-4">
          <a href="https://promeos.com/legal-notice/?lang=en" target="_blank">Imprint</a>

          <a href="https://promeos.com/data-protection-notice/?lang=en" target="_blank">Privacy</a>
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  name: 'Header',
  data() {
    return {
      isMenuOpen: false,
    }
  },
  methods: {
    toggleMenu() {
      this.isMenuOpen = !this.isMenuOpen
    },
  },
  beforeMount() {
    // Your JavaScript snippet here
    console.log('Page loaded!');
    const currentHash = window.location.hash;
    if (currentHash.includes("invite_token")) {
      window.location.href = "/admin/" + currentHash;
    }
  } 
}
</script>
  const currentHash = window.location.hash;
  if (currentHash.includes("_token")) {
    window.location.href = "/admin/" + currentHash;
  }

<style lang="postcss" scoped>
.scrim-bg {
  &::before {
    content: '';
    z-index: -1;
    background-color: var(--bg);
    @apply absolute bottom-0 inset-x-0 h-12 mb-4 transition-colors duration-200 ease-in-out;
  }
  &::after {
    content: '';
    z-index: -1;
    opacity: 1;
    animation: fadeIn1 500ms ease-in-out;
    @apply pointer-events-none absolute bottom-0 inset-x-0 h-16 -mb-12;
    background: linear-gradient(to bottom, #111827, cubic-bezier(0.15, 0, 0.45, 1), transparent);
  }
}

.menu-container {
  position: relative;
  height: 100%;
  transition: transform 0.3s ease-in-out;
  transform: translateX(0);
}

.menu-open .menu-container {
  transform: translateX(-100%);
}

.hamburger {
  width: 30px;
  height: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  cursor: pointer;
  position: absolute;
  top: 30px;
  right: 30px;
  z-index: 10;
}

.line {
  width: 100%;
  height: 4px;
  background-color: #fff;
}

.menu-panel {
  position: absolute;
  top: 0;
  right: 0;
  width: 300px;
  height: 100vh;
  background-color: var(--bg);
  border-left: 4px solid var(--color-primary);
  padding: 60px 25px 20px;
  transform: translateX(100%);
  transition: transform 0.3s ease-in-out;
}

.menu-open .menu-panel {
  transform: translateX(0);
}

.menu-items {
  list-style: none;
  padding: 0;
  margin: 0;
}

.menu-items li {
  margin-bottom: 10px;
  font-family: var(--font-heading);
}

.nuxt-link-exact-active {
  @apply text-primary border-gray-400 rounded bg-gray-600 bg-opacity-25 cursor-default;
}

.light {
  & .scrim-bg {
    &::after {
      animation-name: fadeIn2;
      background: linear-gradient(to bottom, #e5e7eb, cubic-bezier(0.15, 0, 0.45, 1), transparent);
    }
  }
  & .nuxt-link-exact-active {
    @apply text-primary-700 border-gray-600 bg-gray-100;
  }
}

/* Need two because of smoother switching between color modes */
@keyframes fadeIn1 {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
@keyframes fadeIn2 {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
