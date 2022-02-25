<template>
  <div class="view-bound">
    <!-- View left -->
    <div class="view-left">
      <div class="container fl-4">
        <span class="title">Tỷ giá USD</span>
        <div v-if="isLoadingChange">
          <span>Loading...</span>
        </div>
        <div class="scroll-view" style="height: 200px" v-else>
          <div v-for="(iChange, indChange) in listChange" :key="indChange">
            <div class="view-change" style="padding-block: 6px">
              <span class="name-coin"
              style="margin-right: 10px;"
                >1 USD = {{ iChange.Rate.toFixed(4).toString() }}
                {{ iChange.Symbol }}</span
              >
              <ArrowTrend :currentV="iChange.Rate" :lastV="iChange.RateYesterday" />
            </div>
          </div>
        </div>
      </div>

      <div class="container fl-7">
        <span class="title">Danh sách coin</span>
        <div v-if="isLoadingCoin">
          <span>Loading...</span>
        </div>
        <div class="scroll-view" style="height: 500px" v-else>
          <div
            @click="onSelectCoin(iCoin)"
            v-for="(iCoin, indCoin) in listCoin"
            :key="indCoin"
            :style="{
              'background-color': currentCoin == iCoin.Symbol ? '#ff3' : '#fff',
              'padding-inline': '10px',
            }"
          >
            <div class="view-hori">
              <span class="name-coin">{{ iCoin.Name }}</span>
              <span class="name-coin">{{ iCoin.Symbol }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- End view left -->

    <!-- View Chart -->
    <div class="view-chart">
      <div class="container fl-1">
        <span class="title"> Chi tiết coin</span>
        <div>
          <span style="font-size: 25px"
            >{{ currentCoin ? currentCoin : "--" }}/USDT</span
          >
          <span style="font-size: 25px; padding-inline: 12px"
            >{{ detailCurrentCoin.price.toFixed(4).toString() }}/USDT</span
          >
          <ArrowTrend :currentV="detailCurrentCoin.price" :lastV="detailCurrentCoin.yesterdayPrice" />
        </div>
      </div>
    </div>
    <!-- End view chart -->
  </div>
</template>

<script>
// import axios from "axios";
import CoinList from "./coin_list.json";
import CoinPrice from "./price.json";
import ArrowTrend from "./ArrowTrend.vue";

export default {
  components: {
    ArrowTrend,
  },
  data() {
    return {
      listCoin: [],
      currentCoin: null,
      listChange: [],
      isLoadingChange: false,
      isLoadingCoin: false,
      isLoadingCurrentCoin: false,
      currentChange: null,
      detailCurrentCoin: {
        price: 0,
        yesterdayPrice: 0,
      },
      refInterval: null,
    };
  },
  methods: {
    getListChange() {
      this.listChange = [];
      this.isLoadingChange = true;
      // axios
      //   .get("URL")
      //   .then((res) => {
      //   this.isLoadingChange = false;
      // })
      //   .catch((error) => {});

      this.isLoadingChange = false;

      const res = CoinPrice.Change.USD;
      this.listChange = res;
    },

    getListCoin() {
      this.listCoin = [];
      this.isLoadingCoin = true;
      // Get list coin from api
      // axios
      //   .get("URL")
      //   .then((res) => {
      // })
      //   .catch((error) => {});

      this.isLoadingCoin = false;

      const res = CoinList.CompleteCoinList;
      this.listCoin = res;
      if (res.length > 0) {
        this.onSelectCoin(res[0]);
      }
    },

    onSelectCoin(iCoin) {
      this.currentCoin = iCoin.Symbol;
      if (this.refInterval) {
        clearInterval(this.refInterval);
      }
      let coinExist = CoinList.Coins.find((t) => t.Symbol == this.currentCoin);
      if (coinExist) {
        this.detailCurrentCoin = {
          price: coinExist.Price,
          yesterdayPrice: coinExist.VolumeYesterdayUSD,
        };
      } else this.getDetailCurrentCoin();
    },

    getDetailCurrentCoin() {
      // Repeat function after 5second
      this.refInterval = setInterval(() => {
        // Call api get detail currentCoin: price,...
        // axios
        //   .get("URL")
        //   .then((res) => {
        // })
        //   .catch((error) => {});

        this.detailCurrentCoin = {
          price: Math.random() * 10,
          yesterdayPrice: Math.random() * 10,
        };
      }, 1000);
    },
  },
  created() {
    this.getListChange();
    this.getListCoin();
  },
};
</script>

<style>
.view-bound {
  flex-direction: row;
  margin-inline: 24px;
  display: flex;
  height: 100%;
}

.view-left {
  display: flex;
  flex: 3;
  height: 100%;
  padding: 8px;
  flex-direction: column;
}

.view-chart {
  display: flex;
  flex: 7;
  height: 100%;
  padding: 8px;
  flex-direction: column;
}

.container {
  padding: 14px;
  border-radius: 4px;
  margin-bottom: 6px;
  border: 1px solid #d5d5d5;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.title {
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 8px;
}

.scroll-view {
  overflow: scroll;
  flex-shrink: 0;
  width: 100%;
}

html,
body {
  height: 100%;
}

.view-change {
  flex-direction: row;
}

.view-hori {
  flex-direction: row;
  justify-content: space-between;
  display: flex;
  padding-block: 6px;
  border-bottom: 1px solid #f1f1f1;
}
</style>