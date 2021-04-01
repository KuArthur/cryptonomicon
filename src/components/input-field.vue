<template>
  <div class="flex">
    <div class="max-w-xs">
      <label for="wallet" class="block text-sm font-medium text-gray-700"
        >Тикер</label
      >
      <div class="mt-1 relative rounded-md shadow-md">
        <input
          v-model = "search"
          @keydown.enter="addTicker"
          type="text"
          name="wallet"
          id="wallet"
          class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
        />
      </div>
      <div
        v-if="ticker"
        class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
      >
        <span
          v-for="(coin, idx) in coins(ticker)"
          :key="idx"
          @click="addCoin(coin)"
          class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
        >
          {{ coin }}
        </span>
      </div>
      <div class="text-sm text-red-600">Такой тикер уже добавлен</div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      required: true,
    },
    coins: {
      type: Function,
      required: true,
    },
  },

  data() {
    return {
      search: this.value
    }
  },

  watch: {
    search(value) {
      this.$emit('search',value)
    },
  },

  methods: {
    
    addTicker() {
      this.$emit("addTicker",this.tickerProp);
      
      console.log(this.tickerProp);

    },

    addCoin(coin) {
      this.$emit("addCoin", coin);
    },
  },
};
</script>
