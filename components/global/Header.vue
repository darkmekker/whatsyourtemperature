<template>
  <nav class="fixed z-40 top-0 inset-x-0" aria-label="Main Menu">
    <div class="menu-container" :class="{ 'menu-open': isMenuOpen }">
      <div class="hamburger" @click="toggleMenu">
        <div class="line"></div>
        <div class="line"></div>
        <div class="line"></div>
      </div>
      <div class="menu-panel">
        <div class="hamburger" @click="toggleMenu">
          <div class="line"></div>
          <div class="line"></div>
          <div class="line"></div>
        </div>
        <ul class="menu-items" @click="toggleMenu">
          <li class="flex-1">
            <nuxt-link class="btn block" to="/">Home</nuxt-link>
          </li>
          <li class="flex-1 ml-2">
            <nuxt-link class="btn block" to="/about">About</nuxt-link>
          </li>
          <li class="flex-1 ml-2">
            <nuxt-link class="btn block" to="/history">History</nuxt-link>
          </li>
          <li class="flex-1 ml-2">
            <nuxt-link class="btn block" to="/research">Research</nuxt-link>
          </li>
          <li class="flex-1 ml-2">
            <nuxt-link class="btn block" to="/contact">Contact</nuxt-link>
          </li>
        </ul>
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
}
</script>

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
  top: 20px;
  right: 20px;
}

.line {
  width: 100%;
  height: 2px;
  background-color: #fff;
}

.menu-panel {
  position: absolute;
  top: 0;
  right: 0;
  width: 200px;
  height: 100%;
  background-color: #111827;
  padding: 20px;
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
}

.nuxt-link-exact-active {
  @apply text-gray-200 border-gray-400 bg-gray-800 bg-opacity-25 cursor-default;
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
