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
        <InputField
          :ticker = "ticker"
          :coins = "getCoinsList"
          @addTicker="addTicker"
          @addCoin = "addCoin"
         />
        <!-- TODO: заставить это работать корректно, если такое возможно-->
        <AddButton :addTicker = "addTicker">Добавить</AddButton> 
      </section>
      <template v-if="tickers.length">
        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <!-- <ticker/> -->
          <Tickers
            :tickers = "tickers"
            :sell = "sell"
            @remove = "deleteTicker"
            @select = "select"
          />
          <!-- <ticker /> -->
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>

      <!-- <graph /> -->
      <section v-if="sell" class="relative">
        <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
          {{ sell.name }} - USD
        </h3>
        <div class="flex items-end border-gray-600 border-b border-l h-64">
          <div
            v-for="(bar, idx) in normalizeGraphHeight()"
            :key="idx"
            :style="{ height: `${bar}%` }"
            class="bg-purple-800 border w-10"
          ></div>
        </div>
        <button
          @click="sell = null"
          type="button"
          class="absolute top-0 right-0"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:svgjs="http://svgjs.com/svgjs"
            version="1.1"
            width="30"
            height="30"
            x="0"
            y="0"
            viewBox="0 0 511.76 511.76"
            style="enable-background: new 0 0 512 512"
            xml:space="preserve"
          >
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                fill="#718096"
                data-original="#000000"
              ></path>
            </g>
          </svg>
        </button>
      </section>
      <!-- <graph /> -->
    </div>
  </div>
</template>

<script>
import InputField from "@/components/input-field.vue";
import AddButton from "@/components/add-button.vue";
import Tickers from "@/components/ticker.vue";

export default {
  name: "App",
  components: {InputField, AddButton, Tickers},
  data () {
    return {
      ticker: "",
      tickers: [],
      sell: null,
      graph: [],
      coinsList: []
    }
  },
  
  async created() {
    const res = await fetch(
          `https://min-api.cryptocompare.com/data/all/coinlist?summary=true`,
        );
    const coins = await res.json();
    this.coinsList.push(...Object.values(coins.Data))
    this.coinsList = this.coinsList.map(el => el.Symbol)
  },

  methods: {
    addTicker() {
      const currentTicker = {
        name: this.ticker,
        price: "-",
      };
      
      for(let addedTicker of this.tickers) {
        if(addedTicker.name === this.ticker) return;
      }

      this.tickers.push(currentTicker);

      setInterval(async () => {
        const res = await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${currentTicker.name}&tsyms=USD&api_key=a702990e9a088e4f6c2093af65a63ce15b25ddc1a07548dc445cff1b566fcd8f`,
        );
        const data = await res.json();

        this.tickers.find((el) => el.name === currentTicker.name).price = data.USD;

        if (this.sell?.name === currentTicker.name) {
          this.graph.push(data.USD);
        }

      }, 3000);

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

      return this.coinsList.filter(el => el.toLowerCase().includes(ticker)).slice(0,4)
    },

    addCoin(coin) {
      this.ticker = coin;
      this.addTicker()
    },

    checkDublicate() {
      return this.tickers.includes(this.ticker)
    }

  },
};
</script>
