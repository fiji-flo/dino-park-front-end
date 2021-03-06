<template>
  <article class="list-container">
    <header class="list-container__control-bar">
      <SearchForm
        class="search-container"
        id="access-group-list-search"
        v-on:clear-query="clearSearchHandler"
        :searchFormHandler="searchFormHandler"
        :searchFormLabel="fluent('access-group_list', 'search')"
        :searchFormKeyUpHandler="searchFormKeyUpHandler"
        :initialQuery="$route.query.group"
      ></SearchForm>
      <Select
        class="options--chevron options--large group-select"
        label="Sort"
        id="member-list-sort"
        v-model="selectedSort"
        :options="sortOptions"
        :nonOption="defaultSort"
      ></Select>
    </header>
    <ul class="list-container__list">
      <li
        v-for="(group, idx) in groupList"
        :key="idx"
        class="list-item-container"
      >
        <AccessGroupListItem :group="group" />
      </li>
    </ul>
    <footer class="list-container__actions" v-if="canShowMore">
      <Button class="show-more" @click="handleShowMoreClicked">
        <Icon id="chevron-down" :width="24" :height="24"></Icon>
        {{ fluent('access-group_list', 'show-more') }}
      </Button>
    </footer>
  </article>
</template>

<script>
import EditButton from '@/components/ui/EditButton.vue';
import Button from '@/components/ui/Button.vue';
import AccessGroupListItem from '@/components/access_group/AccessGroupListItem.vue';
import SearchForm from '@/components/ui/SearchForm.vue';
import Select from '@/components/ui/Select.vue';
import Tooltip from '@/components/ui/Tooltip.vue';
import Icon from '@/components/ui/Icon.vue';
import { AbbGroupViewModel } from '@/view_models/AccessGroupViewModel';

const resultsStep = 20;
const defaultListOptions = {
  search: '',
  sort: 'member-count-desc',
  numResults: resultsStep,
  next: null,
};

export default {
  name: 'AccessGroupList',
  components: {
    EditButton,
    Button,
    Icon,
    SearchForm,
    Select,
    Tooltip,
    AccessGroupListItem,
  },
  props: {
    title: String,
  },
  methods: {
    async fetchList(options = {}) {
      try {
        const data = await this.accessGroupApi.execute({
          path: 'groups/get',
          endpointArguments: [options],
        });

        const localFetchList = data.groups.map(
          (group) => new AbbGroupViewModel(group),
        );
        // if we have requested new groups, concat them to the already existing ones
        if (options.hasOwnProperty('next') && options.next) {
          this.groupList = this.groupList.concat(localFetchList);
        } else {
          this.groupList = localFetchList;
        }

        this.next = data.next;
      } catch (e) {
        console.error('Propagating error during fetchList()', e);
        throw new Error(e);
      }
    },
    // eslint-disable-next-line
    async searchFormHandler(searchQuery, scope) {
      if (this.nextClicked) {
        this.resetOptions();
      }
      this.listOptions.search = searchQuery;
      if (this.$route.query.group !== searchQuery) {
        if (this.$route.query.group && searchQuery === '') {
          this.$router.replace({ query: {} });
        } else {
          this.$router.replace({ query: { group: searchQuery } });
        }
      }
      await this.fetchList(this.listOptions);
    },
    searchFormKeyUpHandler(searchQuery, scope) {
      this.searchFormHandler(searchQuery, scope);
    },
    async clearSearchHandler() {
      this.listOptions.search = '';
      await this.fetchList(this.listOptions);
    },
    async handleShowMoreClicked() {
      this.nextClicked = true;
      this.listOptions.next = this.next;
      await this.fetchList(this.listOptions);
    },
    resetOptions() {
      this.listOptions = defaultListOptions;
      this.nextClicked = false;
      this.next = null;
      // fix `next` not getting overwritten to a falsey value besides `defaultListOptions.next` is null.
      this.listOptions.next = null;
    },
  },
  computed: {
    canShowMore() {
      return this.next !== null;
    },
  },
  watch: {
    selectedSort(value) {
      if (this.nextClicked) {
        this.resetOptions();
      }
      this.listOptions.sort = value;
      this.fetchList(this.listOptions);
    },
  },
  data() {
    return {
      nextClicked: false,
      groupList: [],
      listOptions: {
        ...defaultListOptions,
        search: this.$route.query.group || '',
      },
      selectedSort: defaultListOptions.sort,
      next: null,
      sortOptions: [
        { value: 'name-asc', label: 'Name A-Z' },
        { value: 'name-desc', label: 'Name Z-A' },
        { value: 'member-count-desc', label: 'Most members' },
        { value: 'member-count-asc', label: 'Fewest members' },
      ],
      defaultSort: {
        value: '',
        label: 'Sort',
      },
    };
  },
  async created() {
    await this.fetchList(this.listOptions);
  },
};
</script>

<style>
.list-container {
  width: 100%;
}

@media (min-width: 57.5em) {
  .list-container {
    margin: 0;
    box-shadow: none;
    width: auto;
  }
}

.list-container__control-bar {
  margin: 0;
  position: relative;
  padding: 1.5em;
  display: flex;
  flex-direction: row;
  justify-content: center;
  background: var(--white);
  box-shadow: var(--shadowCard);
  margin: 0 1em 2em 1em;
}

.list-container__control-bar .search-form__input {
  font-size: 1em;
  padding-top: 0.7em;
  padding-bottom: 0.7em;
  padding-left: 4em;
}

.list-container__control-bar .search-form__input::placeholder {
  text-align: left;
}

@media (min-width: 57.5em) {
  .list-container__control-bar {
    background: var(--white);
    margin: 0 0 2em 0;
  }

  .list-container__control-bar .search-form__input {
    font-size: initial;
  }
}

.list-container__control-bar .group-select {
  align-self: center;
  margin-left: 1em;
}

.list-container__control-bar .group-select .options__toggle {
  border: 1px solid var(--gray-30);
  padding: 0.75em 2.5em 0.75em 1.75em;
}

.list-container__control-bar .group-select .options__toggle:hover {
  border-color: var(--blue-60);
}

.search-container {
  margin: 0 auto;
}

@media (min-width: 57.5em) {
  .list-container__control-bar {
    justify-content: initial;
    position: relative;
  }

  .list-container__control-bar .group-select {
    position: absolute;
    top: 1.7em;
    right: 1em;
  }
}

.list-container__control-bar .list-container__tooltip {
  display: flex;
  position: absolute;
  right: 3%;
  top: 50%;
  height: 2em;
  transform: translate(0, -50%);
}

@media (min-width: 35em) {
  .list-container__control-bar .list-container__tooltip {
    right: 2em;
  }
}

.list-container__control-bar .list-container__tooltip .show-more__button {
  margin: 0;
}

.list-container__list {
  list-style-type: none;
  margin: 0;
  padding: 0;
  width: 100%;
}

@media (min-width: 57.5em) {
  .list-container__list {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-row-gap: 1em;
    grid-column-gap: 1em;
    grid-auto-rows: min-content;
  }
}

.list-container__list .list-item-container {
  width: 100%;
  display: flex;
  background: var(--white);
  box-shadow: var(--shadowCard);
  margin: 1em 0;
  word-break: break-all;
  height: min-content;
}

@media (min-width: 57.5em) {
  .list-container__list .list-item-container {
    background: var(--white);
    box-shadow: none;
    margin: 0;
  }
  .list-container__list .list-item-container:nth-child(even) {
    background: var(--white);
  }
}

.list-container__list-column {
  margin: 0;
  padding: 0;
}

@media (min-width: 57.5em) {
  .list-container__list-column {
    display: flex;
    flex-direction: column;
  }
}

.list-container__top-bar .top-bar__search {
  margin: 0 auto;
  width: 15em;
}

@media (min-width: 57.5em) {
  .list-container__top-bar .top-bar__search {
    width: 25em;
  }
}

.list-container .list-container__empty {
  text-align: center;
  margin: 3em auto;
}

.list-container .list-container__actions {
  background: var(--gray-20);
  display: flex;
  justify-content: center;
  padding: 1em;
}

.list-container .list-container__actions .show-more {
  background: transparent;
  color: var(--gray-50);
}

.list-container .list-container__actions .show-more:hover {
  border: 1px solid transparent;
}
</style>
