<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="sidebar-top">
      <div class="logo">
        <h1 class="logo-title">{{ t('nav.companyName') }}</h1>
        <span class="logo-subtitle">{{ t('nav.subtitle') }}</span>
      </div>
      <button
        type="button"
        class="collapse-btn"
        :aria-expanded="String(!collapsed)"
        :aria-label="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        @click="toggleCollapsed"
      >
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <rect x="3" y="4" width="18" height="16" rx="2" />
          <path d="M9 4v16" />
          <path v-if="collapsed" d="M13 9l3 3-3 3" />
          <path v-else d="M16 9l-3 3 3 3" />
        </svg>
      </button>
    </div>

    <nav class="sidebar-nav" aria-label="Primary">
      <router-link to="/" class="nav-link" :title="t('nav.overview')" :aria-label="t('nav.overview')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <rect x="3" y="3" width="7" height="9" rx="1" />
          <rect x="14" y="3" width="7" height="5" rx="1" />
          <rect x="14" y="12" width="7" height="9" rx="1" />
          <rect x="3" y="16" width="7" height="5" rx="1" />
        </svg>
        <span class="nav-text">{{ t('nav.overview') }}</span>
      </router-link>
      <router-link to="/inventory" class="nav-link" :title="t('nav.inventory')" :aria-label="t('nav.inventory')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M21 8l-9-5-9 5 9 5 9-5z" />
          <path d="M3 8v8l9 5 9-5V8" />
          <path d="M12 13v8" />
        </svg>
        <span class="nav-text">{{ t('nav.inventory') }}</span>
      </router-link>
      <router-link to="/orders" class="nav-link" :title="t('nav.orders')" :aria-label="t('nav.orders')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M9 4h6a1 1 0 011 1v1H8V5a1 1 0 011-1z" />
          <path d="M8 6H6a2 2 0 00-2 2v11a2 2 0 002 2h12a2 2 0 002-2V8a2 2 0 00-2-2h-2" />
          <path d="M8 12h8M8 16h5" />
        </svg>
        <span class="nav-text">{{ t('nav.orders') }}</span>
      </router-link>
      <router-link to="/spending" class="nav-link" :title="t('nav.finance')" :aria-label="t('nav.finance')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <rect x="2" y="5" width="20" height="14" rx="2" />
          <path d="M2 10h20" />
          <path d="M6 15h4" />
        </svg>
        <span class="nav-text">{{ t('nav.finance') }}</span>
      </router-link>
      <router-link to="/demand" class="nav-link" :title="t('nav.demandForecast')" :aria-label="t('nav.demandForecast')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M3 17l6-6 4 4 8-8" />
          <path d="M15 7h6v6" />
        </svg>
        <span class="nav-text">{{ t('nav.demandForecast') }}</span>
      </router-link>
      <router-link to="/reports" class="nav-link" :title="t('nav.reports')" :aria-label="t('nav.reports')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M14 3H7a2 2 0 00-2 2v14a2 2 0 002 2h10a2 2 0 002-2V8l-5-5z" />
          <path d="M14 3v5h5" />
          <path d="M9 13h6M9 17h6" />
        </svg>
        <span class="nav-text">{{ t('nav.reports') }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
    </div>
  </aside>
</template>

<script>
import { ref, watch } from 'vue'
import { useI18n } from '../composables/useI18n'
import LanguageSwitcher from './LanguageSwitcher.vue'
import ProfileMenu from './ProfileMenu.vue'

const KEY = 'sidebar-collapsed'
const read = () => { try { return localStorage.getItem(KEY) === '1' } catch { return false } }
const write = (v) => { try { localStorage.setItem(KEY, v ? '1' : '0') } catch { /* ignore */ } }

export default {
  name: 'SidebarNav',
  components: {
    LanguageSwitcher,
    ProfileMenu
  },
  emits: ['collapse-change', 'show-profile-details', 'show-tasks'],
  setup(props, { emit }) {
    const { t } = useI18n()
    const collapsed = ref(read())

    const toggleCollapsed = () => {
      collapsed.value = !collapsed.value
      write(collapsed.value)
    }

    // Tell the shell the current state (including the initial value)
    watch(collapsed, (v) => emit('collapse-change', v), { immediate: true })

    return {
      t,
      collapsed,
      toggleCollapsed
    }
  }
}
</script>

<style scoped>
.sidebar {
  position: sticky;
  top: 0;
  height: 100vh;
  z-index: 50;
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
  min-width: 0;
}

.sidebar-top {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--space-2);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--color-border);
}

.logo {
  min-width: 0;
}

.logo-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--color-text);
  letter-spacing: -0.025em;
  line-height: 1.25;
}

.logo-subtitle {
  display: block;
  margin-top: var(--space-1);
  font-size: 0.75rem;
  color: var(--color-text-muted);
}

.collapse-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 32px;
  height: 32px;
  background: none;
  border: 1px solid transparent;
  border-radius: var(--radius-sm);
  color: var(--color-text-muted);
  cursor: pointer;
}

.collapse-btn:hover {
  background: var(--color-hover);
  color: var(--color-text);
}

.collapse-btn:focus-visible,
.nav-link:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.collapse-btn svg {
  width: 20px;
  height: 20px;
}

/* Only the link list scrolls so footer dropdowns are never clipped */
.sidebar-nav {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-4) var(--space-3);
}

.nav-link {
  position: relative;
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-md);
  color: var(--color-text-muted);
  text-decoration: none;
  font-size: 0.938rem;
  font-weight: 500;
  white-space: nowrap;
  transition: background-color 0.15s ease, color 0.15s ease;
}

.nav-link svg {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.nav-link:hover {
  background: var(--color-hover);
  color: var(--color-text);
}

.nav-link.router-link-exact-active {
  background: var(--color-primary-soft);
  color: var(--color-primary);
}

.nav-link.router-link-exact-active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 6px;
  bottom: 6px;
  width: 3px;
  border-radius: 0 2px 2px 0;
  background: var(--color-primary);
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-3);
  border-top: 1px solid var(--color-border);
}

.sidebar-footer :deep(.language-button),
.sidebar-footer :deep(.profile-button) {
  width: 100%;
}

/* Rail mode */
.sidebar.collapsed .sidebar-top {
  flex-direction: column;
  align-items: center;
  padding: var(--space-4) var(--space-2);
}

.sidebar.collapsed .logo {
  display: none;
}

.sidebar.collapsed .nav-text {
  display: none;
}

.sidebar.collapsed .sidebar-nav {
  padding: var(--space-4) var(--space-2);
}

.sidebar.collapsed .nav-link {
  justify-content: center;
  padding: var(--space-3) 0;
}

.sidebar.collapsed .sidebar-footer {
  padding: var(--space-3) var(--space-2);
  align-items: center;
}

.sidebar.collapsed .sidebar-footer :deep(.language-label),
.sidebar.collapsed .sidebar-footer :deep(.profile-name),
.sidebar.collapsed .sidebar-footer :deep(.chevron) {
  display: none;
}

.sidebar.collapsed .sidebar-footer :deep(.language-button),
.sidebar.collapsed .sidebar-footer :deep(.profile-button) {
  width: auto;
  justify-content: center;
  padding: var(--space-2);
}
</style>
