<template>
  <aside class="sidebar">
    <div class="sidebar-logo">
      <h1>{{ t('nav.companyName') }}</h1>
      <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
    </div>

    <nav class="sidebar-nav">
      <router-link to="/" :class="{ active: $route.path === '/' }">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M2.5 2.5H8V8H2.5V2.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M10 2.5H15.5V6.5H10V2.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M10 8.5H15.5V15.5H10V8.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M2.5 10H8V15.5H2.5V10Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
        </svg>
        {{ t('nav.overview') }}
      </router-link>

      <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M2.5 5.5L9 2.5L15.5 5.5L9 8.5L2.5 5.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M2.5 5.5V12.5L9 15.5L15.5 12.5V5.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M9 8.5V15.5" stroke="currentColor" stroke-width="1.5"/>
        </svg>
        {{ t('nav.inventory') }}
      </router-link>

      <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M2.5 4.5H15.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M2.5 9H15.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M2.5 13.5H15.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <circle cx="5" cy="4.5" r="1" fill="currentColor"/>
          <circle cx="5" cy="9" r="1" fill="currentColor"/>
          <circle cx="5" cy="13.5" r="1" fill="currentColor"/>
        </svg>
        {{ t('nav.orders') }}
      </router-link>

      <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <circle cx="9" cy="9" r="6.5" stroke="currentColor" stroke-width="1.5"/>
          <path d="M9 5.5V12.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M11 7C11 6.17157 10.1046 5.5 9 5.5C7.89543 5.5 7 6.17157 7 7C7 7.82843 7.89543 8.5 9 8.5C10.1046 8.5 11 9.17157 11 10C11 10.8284 10.1046 11.5 9 11.5C7.89543 11.5 7 10.8284 7 10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        {{ t('nav.finance') }}
      </router-link>

      <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M2.5 14L6.5 9.5L9.5 12L15.5 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M11.5 5H15.5V9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        {{ t('nav.demandForecast') }}
      </router-link>

      <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M5 2.5H11L14 5.5V15C14 15.5523 13.5523 16 13 16H5C4.44772 16 4 15.5523 4 15V3.5C4 2.94772 4.44772 2.5 5 2.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M11 2.5V5.5H14" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M6.5 9.5H11.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M6.5 12.5H11.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        {{ t('nav.reports') }}
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="emit('show-profile-details')"
        @show-tasks="emit('show-tasks')"
      />
    </div>
  </aside>
</template>

<script setup>
import { useI18n } from '../composables/useI18n'
import LanguageSwitcher from './LanguageSwitcher.vue'
import ProfileMenu from './ProfileMenu.vue'

const { t } = useI18n()

const emit = defineEmits(['show-profile-details', 'show-tasks'])
</script>

<style scoped>
.sidebar {
  width: 256px;
  height: 100vh;
  position: sticky;
  top: 0;
  background: white;
  border-right: 1px solid #e2e8f0;
  display: flex;
  flex-direction: column;
  padding: 1.5rem 1.25rem;
}

.sidebar-logo {
  padding: 0 0.625rem 1.5rem;
  margin-bottom: 0.5rem;
  border-bottom: 1px solid #e2e8f0;
}

.sidebar-logo h1 {
  font-size: 1.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
  margin-bottom: 0.375rem;
}

.sidebar-subtitle {
  font-size: 0.813rem;
  color: #64748b;
  font-weight: 400;
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  margin-top: 1.25rem;
  overflow-y: auto;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 0.625rem;
  color: #64748b;
  text-decoration: none;
  font-weight: 500;
  font-size: 0.875rem;
  border-radius: 8px;
  border-left: 3px solid transparent;
  transition: all 0.2s ease;
}

.sidebar-nav a svg {
  flex-shrink: 0;
  color: #64748b;
  transition: color 0.2s ease;
}

.sidebar-nav a:hover {
  color: #0f172a;
  background: #f1f5f9;
}

.sidebar-nav a:hover svg {
  color: #0f172a;
}

.sidebar-nav a.active {
  color: #2563eb;
  background: #eff6ff;
  border-left-color: #2563eb;
}

.sidebar-nav a.active svg {
  color: #2563eb;
}

.sidebar-footer {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: 0.625rem;
  padding-top: 1.25rem;
  border-top: 1px solid #e2e8f0;
}

.sidebar-footer :deep(.profile-button),
.sidebar-footer :deep(.language-button) {
  width: 100%;
}
</style>
