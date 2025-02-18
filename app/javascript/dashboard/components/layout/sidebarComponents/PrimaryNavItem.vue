<script>
import { OnClickOutside } from '@vueuse/components';
import { HELP_CENTER_MENU_ITEMS } from 'dashboard/helper/portalHelper';

import DropdownMenu from 'dashboard/components-next/dropdown-menu/DropdownMenu.vue';

export default {
  components: {
    DropdownMenu,
    OnClickOutside,
  },
  props: {
    to: {
      type: String,
      default: '',
    },
    id: {
      type: String,
      default: '',
    },
    name: {
      type: String,
      default: '',
    },
    icon: {
      type: String,
      default: '',
    },
    count: {
      type: String,
      default: '',
    },
    isChildMenuActive: {
      type: Boolean,
      default: false,
    },
    openInNewPage: {
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      helpCenterMenu: HELP_CENTER_MENU_ITEMS,
      showHelpCenterMenu: false,
    };
  },
  computed: {
    helpCenterMenuItems() {
      return this.helpCenterMenu.map(item => ({
        ...item,
        isSelected: this.isSelectedMenuItem(item),
      }));
    },
    isHelpCenter() {
      return this.id === 'helpcenter';
    },
    isHelpCenterSelected() {
      const routes = [
        'portals_new',
        'portals_index',
        'portals_articles_index',
        'portals_articles_new',
        'portals_articles_edit',
        'portals_categories_index',
        'portals_categories_articles_index',
        'portals_categories_articles_edit',
        'portals_locales_index',
        'portals_settings_index',
      ];
      return routes.includes(this.$route.name);
    },
  },
  methods: {
    isSelectedMenuItem(menuItem) {
      return menuItem.value.includes(this.$route.name);
    },
    toggleHelpCenterMenu() {
      this.showHelpCenterMenu = !this.showHelpCenterMenu;
    },
    handleHelpCenterAction({ action }) {
      this.$router.push({
        name: 'portals_index',
        params: {
          navigationPath: action,
        },
      });
    },
  },
};
</script>

<template>
  <OnClickOutside v-if="isHelpCenter" @trigger="showHelpCenterMenu = false">
    <button
      v-tooltip.top="$t(`SIDEBAR.${name}`)"
      class="relative flex items-center w-full px-4 py-3 text-slate-400 hover:text-slate-100 hover:bg-slate-800/50 transition-colors duration-200 rounded-lg group"
      :class="{
        'bg-slate-800 text-slate-100': isHelpCenterSelected,
      }"
      @click="toggleHelpCenterMenu"
    >
      <fluent-icon
        :icon="icon"
        :class="{
          'text-woot-500': isHelpCenterSelected,
        }"
      />
      <DropdownMenu
        v-if="showHelpCenterMenu && isHelpCenter"
        :menu-items="helpCenterMenuItems"
        class="ltr:left-10 rtl:right-10 w-36 z-[100] top-0 overflow-y-auto max-h-52"
        @action="handleHelpCenterAction"
      />
    </button>
  </OnClickOutside>

  <router-link v-else v-slot="{ href, isActive, navigate }" :to="to" custom>
    <a
      v-tooltip.right="$t(`SIDEBAR.${name}`)"
      :href="href"
      class="relative flex items-center w-full px-4 py-3 text-slate-400 hover:text-slate-100 hover:bg-slate-800/50 transition-colors duration-200 rounded-lg group"
      :class="{
        'bg-slate-800 text-slate-100': isActive || isChildMenuActive,
      }"
      :rel="openInNewPage ? 'noopener noreferrer nofollow' : undefined"
      :target="openInNewPage ? '_blank' : undefined"
      @click="navigate"
    >
      <fluent-icon
        :icon="icon"
        class="w-5 h-5 mr-3"
        :class="{
          'text-slate-100': isActive || isChildMenuActive,
        }"
      />
      <span class="text-sm font-medium">{{ $t(`SIDEBAR.${name}`) }}</span>
      <span
        v-if="count"
        class="flex items-center justify-center px-2 py-0.5 text-xs font-medium bg-violet-500 text-white rounded-full ml-auto"
      >
        {{ count }}
      </span>
    </a>
  </router-link>
</template>
