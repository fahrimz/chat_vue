<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <span class="green">CHAT APP</span>
    <nav>
      <RouterLink to="/">Home</RouterLink>
      <RouterLink to="/about">About</RouterLink>

      <RouterLink class="button" to="/public-chat">Chat Now</RouterLink>
    </nav>
  </header>

  <RouterView />
</template>

<style scoped>
header {
  padding: 0;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 3rem;
}

header span {
  font-weight: bold;
  padding: 0;
}

nav {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
  padding: 0 1rem 0 0;
}

nav .button {
  margin-left: 0.5rem;
}

@media (max-width: 767px) {
  header {
    margin-bottom: 2rem;
  }
}
</style>
