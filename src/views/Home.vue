<template>
  <div>
    <div
      class="category-filters grid grid-3-columns@md grid-gap justify-center pb-8"
    >
      <button
        v-for="filterCat in categories"
        :key="filterCat.value"
        @click="setCategory(filterCat.value)"
        class="category-filter block p-3 rounded border border-solid border-transparent"
        :class="{ 'category-filter--selected': filterCat.value === category }"
      >
        <div class="category-filter__content">
          <img :src="filterCat.icon" class="category-filter__icon" />
          <span>{{ filterCat.label }}</span>
        </div>
      </button>
    </div>

    <form v-on:submit.prevent class="mb-3">
      <input
        v-bind:value="search"
        v-on:input="search = $event.target.value"
        placeholder="Filter articles by text"
        class="p-2 rounded border-gray-400"
      />
    </form>

    <h3 v-if="!filtered.length" class="text-center">No articles.</h3>

    <RouterLink
      v-for="article in filtered"
      :key="article.id"
      :to="{ name: 'article', params: { articleId: article._id } }"
    >
      <article class="md:flex justify-between mb-3 p-3 bg-gray-300 rounded">
        <h3 class="pr-3">{{ article.title }}</h3>
        <small class="text-right">Published: {{ article._createdOn }} </small>
      </article>
    </RouterLink>
  </div>
</template>

<script>
import config from "@/config";
import http from "@/utils/http";

export default {
  props: {
    search: String
  },

  data: () => ({
    articles: [],
    categories: config.categories,
    category: ""
  }),

  computed: {
    filtered() {
      const { search, category } = this;
      return this.articles.filter(article => {
        if (search) {
          const text = (article.title + article.content).toLowerCase();
          if (!text.includes(search.toLowerCase())) {
            return false;
          }
        }
        if (category) {
          const categories = article.categories || [];
          return categories.includes(category);
        }
        return true;
      });
    }
  },

  async mounted() {
    this.category = this.$route.query.category;

    if (this.isOffline) {
      this.articles = Object.values(this.$store.state.downloads);
      return;
    }

    this.$store.commit("addLoader");
    this.articles = await http.get(config.db.articles);
    this.$store.commit("removeLoader");
  },

  methods: {
    setCategory(category) {
      const currentCategory = this.category;
      category = currentCategory === category ? undefined : category;
      this.category = category;
      this.$router.push({ name: "home", query: { category } });
    }
  }
};
</script>

<style lang="scss">
.category-filters {
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
}

.category-filter {
  &__content {
    display: grid;
    grid-template-columns: 50px 1fr;
    align-items: center;
  }

  &:hover {
    background: #eee5cc;
  }

  .grid &--selected,
  .grid &--selected:hover {
    border-color: #bd514a;
  }

  &__icon {
    height: 50px;
    width: 50px;
  }
}
</style>
