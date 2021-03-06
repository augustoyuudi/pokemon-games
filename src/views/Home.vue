<template>
  <main class='home view'>
    <section
      class='home__cards'
      ref='cards'
    >
      <router-link
        v-for='game in games'
        :key='game.id'
        :to='{name: "game", params: { gameName: game.name }}'
        class='home__card'
      >
        <game-card
          v-if="game"
          :game='game'
        >
        </game-card>
      </router-link>
      <scroll-observer @isIntersecting='handleIntersection'></scroll-observer>
    </section>
  </main>
</template>

<script>
import GamesAPI from '@/services/http/Games';
import GameCard from '@/components/GameCard.vue';
import ScrollObserver from '@/components/shared/ScrollObserver.vue';

export default {
  name: 'Home',

  components: {
    GameCard,
    ScrollObserver,
  },

  created() {
    this.fetchGames();
    this.setReloadScrollBehavior();
  },

  destroyed() {
    window.removeEventListener('beforeunload', this.scrollViewToTop);
  },

  computed: {
    gamesPerPage() {
      return 8;
    },
  },

  data() {
    return {
      games: [],
      currentPage: 1,
      canFetchNewGames: true,
    };
  },

  methods: {
    createGamePromises() {
      const initialIndex = (this.gamesPerPage * (this.currentPage - 1)) + 1;
      const lastIndex = this.currentPage * this.gamesPerPage;
      const promises = [];

      for (let i = initialIndex; i <= lastIndex; i += 1) {
        promises.push(GamesAPI.getByID(i));
      }

      return promises;
    },
    async fetchGames() {
      const promises = this.createGamePromises();
      this.currentPage += 1;
      const rawGames = await Promise.allSettled(promises);
      const newGames = rawGames
        .filter((game) => game.value)
        .map((game) => game.value.data);
      this.games = [
        ...this.games,
        ...newGames,
      ];
      this.canFetchNewGames = !(newGames.length !== rawGames.length);
    },
    setReloadScrollBehavior() {
      window.addEventListener('beforeunload', this.scrollViewToTop);
    },
    scrollViewToTop() {
      window.scrollTo(0, 0);
    },
    handleIntersection() {
      if (!this.canFetchNewGames) {
        return;
      }

      this.fetchGames();
    },
  },
};
</script>
