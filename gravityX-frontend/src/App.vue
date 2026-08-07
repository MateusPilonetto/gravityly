<script setup>
import { computed, ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import PwaStatus from './components/PwaStatus.vue';
import { userStore } from './store.js';
import {
  api,
  clearToken,
  getFallbackAvatarUrl,
  getProfileAvatarUrl,
  getToken,
  setUnauthorizedResponseHandler,
} from './services/api.js';

const route = useRoute();
const router = useRouter();
const loggingOut = ref(false);

const isGuestPage = computed(() => Boolean(route.meta.guestOnly));
const showApplicationChrome = computed(() => (
  Boolean(route.meta.requiresAuth) && !route.meta.hideApplicationChrome
));
const navAvatarUrl = computed(() => getProfileAvatarUrl(userStore.currentUser, 50));

let activeSessionRequest = 0;

const redirectToLogin = () => {
  clearToken();
  userStore.clearUser();

  if (!isGuestPage.value) {
    void router.replace('/login');
  }
};

const handleAvatarError = (event) => {
  event.currentTarget.src = getFallbackAvatarUrl(userStore.currentUser, 50);
};

setUnauthorizedResponseHandler(redirectToLogin);

const loadCurrentUser = async () => {
  const requestId = ++activeSessionRequest;

  if (!getToken()) {
    redirectToLogin();
    return;
  }

  try {
    const responsePayload = await api.get('/me');

    if (requestId === activeSessionRequest && responsePayload?.data) {
      userStore.setCurrentUser(responsePayload.data);
    }
  } catch (errorResponse) {
    if (requestId !== activeSessionRequest) return;

    if (errorResponse.status === 401) {
      redirectToLogin();
      return;
    }

    console.error('Failed to load the current session:', errorResponse);
  }
};

watch(
  () => `${Boolean(route.meta.requiresAuth)}:${Boolean(route.meta.guestOnly)}`,
  () => {
    if (!route.meta.requiresAuth) {
      activeSessionRequest += 1;

      if (isGuestPage.value) {
        userStore.clearUser();
      }
      return;
    }

    if (getToken() && userStore.currentUser) {
      return;
    }

    void loadCurrentUser();
  },
  { immediate: true },
);

const handleLogout = async () => {
  if (loggingOut.value) return;

  loggingOut.value = true;

  try {
    if (getToken()) {
      await api.post('/logout');
    }
  } catch (errorResponse) {
    if (errorResponse.status !== 401) {
      console.error('Failed to revoke the current session:', errorResponse);
    }
  } finally {
    loggingOut.value = false;
    redirectToLogin();
  }
};
</script>

<template>
  <header v-if="showApplicationChrome" class="app-header">
    <router-link to="/" class="logo-items" aria-label="Go to GravityX feed">
      <img alt="" class="logo" src="./assets/gravityly-logo-light.svg" width="64" height="64">
      <h1 class="logo-title">GravityX</h1>
    </router-link>

    <button
      type="button"
      class="logout-button glass-effect"
      :disabled="loggingOut"
      :aria-busy="loggingOut"
      aria-label="Log out"
      @click="handleLogout"
    >
      <i :class="loggingOut ? 'fa-solid fa-spinner fa-spin' : 'fa-solid fa-arrow-right-from-bracket'" aria-hidden="true"></i>
    </button>
  </header>

  <main>
    <router-view />
  </main>

  <PwaStatus />

  <div v-if="showApplicationChrome" class="bottom-nav-wrapper">
    <nav class="bottom-nav" aria-label="Main navigation">
      <router-link to="/" class="nav-link" aria-label="Feed">
        <i class="fa-solid fa-house nav-icon" aria-hidden="true"></i>
      </router-link>

      <router-link to="/search" class="nav-link" aria-label="Search">
        <i class="fa-solid fa-magnifying-glass nav-icon" aria-hidden="true"></i>
      </router-link>

      <router-link to="/posts/create" class="nav-link" aria-label="Create post">
        <i class="fa-solid fa-square-plus nav-icon" aria-hidden="true"></i>
      </router-link>

      <router-link to="/profile" class="nav-link" aria-label="Your profile">
        <img
          class="nav-profile-pic"
          loading="lazy"
          :src="navAvatarUrl"
          alt=""
          @error="handleAvatarError"
        >
      </router-link>
    </nav>
  </div>
</template>

<style scoped>
.app-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  width: 100%;
  max-width: 1120px;
  margin: 0 auto;
  padding: max(0.75rem, env(safe-area-inset-top)) clamp(0.85rem, 3vw, 1.5rem) 0.75rem;
  line-height: 1.5;
}

.logo-items {
  display: flex;
  min-width: 0;
  align-items: center;
  gap: clamp(0.55rem, 2vw, 1rem);
  color: inherit;
}

.logo {
  display: block;
  width: clamp(2.75rem, 8vw, 4rem);
  height: clamp(2.75rem, 8vw, 4rem);
  flex: 0 0 auto;
  border-radius: 1.25rem;
}

.logo-title {
  margin: 0;
  color: #FFC857;
  font-size: clamp(1.45rem, 5vw, 2rem);
  line-height: 1;
}

.logout-button {
  display: grid;
  width: clamp(2.75rem, 8vw, 3.5rem);
  height: clamp(2.75rem, 8vw, 3.5rem);
  flex: 0 0 auto;
  place-items: center;
  border-radius: 50%;
  color: #ff7b7b;
  cursor: pointer;
  font-size: 1.1rem;
  transition: transform 180ms ease, border-color 180ms ease, background-color 180ms ease;
}

.logout-button:hover:not(:disabled) {
  border-color: rgba(255, 93, 93, 0.45);
  background: rgba(255, 93, 93, 0.12);
  transform: translateY(-1px);
}

.logout-button:disabled {
  cursor: wait;
  opacity: 0.65;
}

.bottom-nav-wrapper {
  position: fixed;
  z-index: 50;
  right: 0;
  bottom: max(0.75rem, env(safe-area-inset-bottom));
  left: 0;
  display: flex;
  justify-content: center;
  padding: 0 max(0.65rem, env(safe-area-inset-left)) 0 max(0.65rem, env(safe-area-inset-right));
  pointer-events: none;
}

.bottom-nav {
  display: flex;
  width: min(100%, 25rem);
  align-items: center;
  justify-content: space-around;
  gap: clamp(0.25rem, 3vw, 1.25rem);
  padding: 0.55rem clamp(0.7rem, 4vw, 1.35rem);
  border: 1px solid rgba(111, 92, 255, 0.3);
  border-radius: 9999px;
  background-color: rgba(33, 25, 52, 0.95);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.24);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  pointer-events: auto;
}

.nav-link {
  display: grid;
  width: clamp(2.65rem, 12vw, 3.25rem);
  height: clamp(2.65rem, 12vw, 3.25rem);
  place-items: center;
  border-radius: 50%;
  color: #6F5CFF;
  transition: background-color 180ms ease, color 180ms ease, transform 180ms ease;
}

.nav-icon {
  color: currentColor;
  font-size: clamp(1.55rem, 7vw, 2.15rem);
}

.nav-link.router-link-active,
.nav-link:hover {
  background: rgba(201, 194, 232, 0.09);
  color: #C9C2E8;
  transform: translateY(-2px);
}

.nav-profile-pic {
  width: clamp(2rem, 9vw, 2.35rem);
  height: clamp(2rem, 9vw, 2.35rem);
  border: 2px solid #FFC857;
  border-radius: 50%;
  object-fit: cover;
}

.nav-link.router-link-active .nav-profile-pic {
  box-shadow: 0 0 0 3px rgba(255, 200, 87, 0.15);
}

@media (max-width: 390px) {
  .app-header {
    padding-right: 0.75rem;
    padding-left: 0.75rem;
  }

  .bottom-nav {
    gap: 0.1rem;
    padding-right: 0.55rem;
    padding-left: 0.55rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  .logout-button,
  .nav-link {
    transition: none;
  }
}
</style>
