<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <!-- <div
      class="fixed w-100 h-100 opacity-80 bg-purple-800 inset-0 z-50 flex items-center justify-center"
    >
      <svg
        class="animate-spin -ml-1 mr-3 h-12 w-12 text-white"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>
    </div> -->
    <div class="container">
      <section>
        <!-- TODO: заставить работать v-model="ticker" -->
        <InputField
          :value="ticker"
          @search="ticker=$event"
          :coins="getCoinsList"
          @addTicker = "addTicker"
          @addCoin="addCoin"
        />
        <!-- TODO: заставить это работать корректно, если такое возможно-->
        <AddButton @add="addTicker">Добавить</AddButton>
      </section>
      <template v-if="tickers.length">
        <!-- <Filter TODO: сделать так, чтобы работало
          :filter = "filter"
          :page = "page"
          :hasNextPage = "hasNextPage"  
        /> -->
        <div>
          Фильтр :<input
            v-model="filter"
            type="text"
            class="block w-52 pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
          />
        </div>
        <div class="flex justify-items-end">
          <button
            v-if="page > 1"
            @click="page = page - 1"
            type="button"
            class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
          >
            Назад
          </button>
          <button
            v-if="hasNextPage"
            @click="page = page + 1"
            type="button"
            class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
          >
            Вперед
          </button>
        </div>

        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <!-- <ticker/> -->
          <Tickers 
            :tickers="filteredTickers()" 
            :sell="sell" 
            @select="select" 
            @remove="deleteTicker"
          />
          <!-- <ticker /> -->
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>

      <!-- <graph /> -->
      <section v-if="sell" class="relative">
        <Graph :sell = "sell" :column="normalizeGraphHeight()" @close="sell=null" />
      </section>
      <!-- <graph /> -->
    </div>
  </div>
</template>

<script>
import InputField from "@/components/input-field.vue";
import AddButton from "@/components/add-button.vue";
import Tickers from "@/components/ticker.vue";
import Graph from "@/components/graph.vue";
// import Filter from "@/components/filter.vue";

export default {
  name: "App",
  components: { InputField, Tickers, Graph, AddButton },
  data() {
    return {
      ticker: "",
      tickers: [],
      sell: null,
      graph: [],
      coinsList: [],
      filter: "",
      page: 1,
      hasNextPage: true,
    };
  },

  async created() {
    const res = await fetch(
      `https://min-api.cryptocompare.com/data/all/coinlist?summary=true`,
    );
    const coins = await res.json();
    this.coinsList.push(...Object.values(coins.Data));
    this.coinsList = this.coinsList.map((coin) => coin.Symbol);
  },

  mounted() {
    const tickersData = localStorage.getItem("crypto-list");

    if (tickersData) {
      this.tickers = JSON.parse(tickersData);

      this.tickers.forEach((ticker) => this.updatePrice(ticker.name));
    }
  },

  watch: {
    filter() {
      this.page = 1;
    }
  },

  methods: {
    updatePrice(tickerName) {
      setInterval(async () => {
        const res = await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=a702990e9a088e4f6c2093af65a63ce15b25ddc1a07548dc445cff1b566fcd8f`,
        );
        const data = await res.json();

        this.tickers.find((el) => el.name === tickerName).price = data.USD;

        if (this.sell?.name === tickerName) {
          this.graph.push(data.USD);
        }
      }, 3000);
    },

    addTicker() {
      const currentTicker = {
        name: this.ticker,
        price: "-",
      };

      for (let addedTicker of this.tickers) {
        if (addedTicker.name === this.ticker) return;
      }

      this.updatePrice(currentTicker.name);

      this.tickers.push(currentTicker);
      this.filter = "";

      localStorage.setItem("crypto-list", JSON.stringify(this.tickers));

      this.ticker = "";
      
    },

    deleteTicker(currentTicker) {
      this.tickers = this.tickers.filter((el) => el !== currentTicker);
    },

    normalizeGraphHeight() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);

      return this.graph.map(
        (price) => 5 + ((price - minValue) * 95) / (maxValue - minValue),
      );
    },

    select(ticker) {
      this.sell = ticker;
      this.graph = [];
    },

    getCoinsList(ticker) {
      ticker = ticker.toLowerCase();

      return this.coinsList
        .filter((el) => el.toLowerCase().includes(ticker))
        .slice(0, 4);
    },

    addCoin(coin) {
      this.ticker = coin;
      this.addTicker();
    },

    checkDublicate() {
      return this.tickers.includes(this.ticker);
    },

    filteredTickers() {
      const start = (this.page - 1) * 6;
      const end = this.page * 6;

      const filteredTickers = this.tickers.filter((ticker) =>
        ticker.name.includes(this.filter),
      );
      this.hasNextPage = filteredTickers.length > end;

      return filteredTickers.slice(start, end);
    },
  },
};
</script>
