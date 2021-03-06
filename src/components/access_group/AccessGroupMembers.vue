<template>
  <article class="members-container">
    <header class="members-container__top-bar">
      <SearchForm
        class="top-bar__search"
        id="access-group-members-search"
        v-on:clear-query="clearSearchHandler"
        :searchFormHandler="searchFormHandler"
        :searchFormLabel="fluent('access-group_members', 'search')"
      ></SearchForm>
    </header>
    <nav class="members-container__tabs">
      <ul class="tabs-container">
        <li
          :class="{ 'tabs-container__item': true, active: isTabActive(tab) }"
          v-for="(tab, idx) in tabList"
          :key="idx"
          @click="handleTabClick(tab)"
          tabindex="0"
        >
          {{ tab.label }}
        </li>
      </ul>
    </nav>
    <template v-if="membersList.length > 0">
      <ul class="members-container__list">
        <li v-for="(member, idx) in membersList" :key="idx">
          <AccessGroupMemberItem class="list-item-container" :member="member" />
        </li>
      </ul>
      <div class="members-container__load-more">
        <Button class="load-more" v-if="showLoadMore" @click="loadMoreHandler">
          <Icon id="chevron-down" />
          {{ fluent('access-group_members', 'members-container__load-more') }}
        </Button>
      </div>
    </template>
    <template v-else>
      <p class="members-container__empty">
        {{ fluent('access-group_members', 'empty-line1') }}
        <br />
        {{ fluent('access-group_members', 'empty-line2') }}
        <RouterLink
          :to="{
            name: 'Edit Access Group',
            query: {
              section: 'invitations',
            },
          }"
          >{{ fluent('access-group_members', 'empty-action') }}</RouterLink
        >?
      </p>
    </template>
  </article>
</template>

<script>
import EditButton from '@/components/ui/EditButton.vue';
import Button from '@/components/ui/Button.vue';
import AccessGroupMemberItem from '@/components/access_group/AccessGroupMemberItem.vue';
import SearchForm from '@/components/ui/SearchForm.vue';
import Select from '@/components/ui/Select.vue';
import Tooltip from '@/components/ui/Tooltip.vue';
import Icon from '@/components/ui/Icon.vue';
import MembersListMixin from '@/components/_mixins/MembersListMixin.vue';
import { getTwoColumnGridArraySplitFromArray } from '@/assets/js/component-utils';

const defaultTab = 'all';
export default {
  name: 'AccessGroupMembers',
  components: {
    EditButton,
    Button,
    Icon,
    SearchForm,
    Select,
    Tooltip,
    AccessGroupMemberItem,
  },
  mixins: [MembersListMixin],
  props: {
    groupInformation: Object,
  },
  methods: {
    searchFormHandler(searchQuery) {
      this.updateSearch(searchQuery);
    },
    handleTabClick(tab) {
      this.updateRole(tab.key);
      this.activeTab = tab.key;
    },
    clearSearchHandler() {
      this.clearSearch();
    },
    isTabActive(tab) {
      return this.activeTab === tab.key;
    },
    loadMoreHandler() {
      this.loadMoreMembers();
    },
  },
  computed: {
    showLoadMore() {
      return this.membersNext !== null;
    },
  },
  data() {
    return {
      tabList: [
        {
          key: 'all',
          label: this.fluent(
            'access-group_members',
            'tabs-container__item-all',
          ),
        },
        {
          key: 'curators',
          label: this.fluent(
            'access-group_members',
            'tabs-container__item-curators',
          ),
        },
        {
          key: 'members',
          label: this.fluent(
            'access-group_members',
            'tabs-container__item-members',
          ),
        },
      ],
      activeTab: defaultTab,
    };
  },
};
</script>

<style>
.members-container__top-bar {
  background: var(--white);
  box-shadow: var(--shadowCard);
  margin: 0;
  position: relative;
  padding: 1.5em;
  display: flex;
  justify-content: space-between;
}

@media (min-width: 57.5em) {
  .members-container__top-bar {
    justify-content: initial;
  }
}

.members-container__top-bar .members-container__tooltip {
  display: flex;
  position: absolute;
  right: 3%;
  top: 50%;
  height: 2em;
  transform: translate(0, -50%);
}

@media (min-width: 35em) {
  .members-container__top-bar .members-container__tooltip {
    right: 2em;
  }
}

.members-container__top-bar .members-container__tooltip .show-more__button {
  margin: 0;
}

.members-container__tabs {
  min-width: 25em;
  width: 50%;
  border: 1px solid var(--gray-40);
  border-radius: 5em;
  margin: 1em auto 2em auto;
}

.members-container__tabs > ul {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

.members-container__tabs .tabs-container {
  display: flex;
  flex-direction: row;
}

.tabs-container .tabs-container__item {
  flex: 1;
  border-right: 1px solid var(--gray-40);
  text-align: center;
  padding: 1em;
  cursor: pointer;
  background: var(--white);
}

.tabs-container .tabs-container__item.active {
  background: var(--blue-60);
  color: var(--white);
}

.tabs-container .tabs-container__item:first-child {
  padding: 1em;
  border-top-left-radius: 5em;
  border-bottom-left-radius: 5em;
}

.tabs-container .tabs-container__item:nth-child(2) {
  padding: 1em 2em;
}

.tabs-container .tabs-container__item:last-child {
  border-right: none;
  padding-right: 2em;
  padding-left: 1em;
  border-top-right-radius: 5em;
  border-bottom-right-radius: 5em;
}

.members-container__list {
  list-style-type: none;
  margin: 0;
  padding: 0;
  width: 100%;
}

@media (min-width: 57.5em) {
  .members-container__list {
    display: grid;
    grid-template-columns: calc(50% - 0.5em) calc(50% - 0.5em);
    grid-row-gap: 1em;
    grid-column-gap: 1em;
    grid-auto-rows: min-content;
  }
}

.members-container__list .list-item-container {
  width: 100%;
  display: flex;
  margin-top: 1em;
}

.members-container__list-column {
  margin: 0;
  padding: 0;
}

@media (min-width: 57.5em) {
  .members-container__list-column {
    display: flex;
    flex-direction: column;
  }

  .members-container__list .list-item-container:first-child {
    margin-top: 0;
  }
}

.members-container__list .list-item:first-child {
  margin-left: 0;
}

.members-container__top-bar .top-bar__search {
  margin: 0 auto;
  width: 15em;
}

@media (min-width: 57.5em) {
  .members-container__top-bar .top-bar__search {
    width: 25em;
  }
}

.members-container .members-container__empty {
  text-align: center;
  margin: 3em auto;
}

.members-container__load-more {
  display: flex;
  flex-direction: row;
  justify-content: center;
}

.members-container__load-more .load-more {
  margin: 1em auto;
  background-color: transparent;
  color: var(--black);
}
</style>
