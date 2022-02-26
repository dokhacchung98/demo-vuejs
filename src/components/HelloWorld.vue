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
              <span class="name-coin" style="margin-right: 10px"
                >1 USD = {{ iChange.Rate.toFixed(4).toString() }}
                {{ iChange.Symbol }}</span
              >
              <ArrowTrend
                :currentV="iChange.Rate"
                :lastV="iChange.RateYesterday"
              />
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
          <ArrowTrend
            :currentV="detailCurrentCoin.price"
            :lastV="detailCurrentCoin.yesterdayPrice"
          />
          <span
            >volume:
            {{
              detailCurrentCoin.VolumeYesterdayUSD.toFixed(4).toString()
            }}</span
          >
        </div>
        <div
          style="
            width: 100%;
            height: 500px;
            margin-top: 24px;
          "
        >
          <div class="view-exchange" >
            <span>Tên sàn</span>
            <span>Giá</span>
            <span>Cập nhật lúc</span>
          </div>
          <div class="view-exchange" v-for="(item, ind) in listExchange" :key="ind">
            <span>{{item.Name}}</span>
            <span>{{item.Price.toFixed(4).toString()}} USDT</span>
            <span>{{formatTime(item.Time)}}</span>
          </div>
        </div>
      </div>
    </div>
    <!-- End view chart -->
  </div>
</template>

<script>
import axios from "axios";

import CoinPrice from "./price.json";
import ArrowTrend from "./ArrowTrend.vue";
import moment from 'moment';

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
        VolumeYesterdayUSD: 0,
      },
      refInterval: null,
      listExchange: [],
    };
  },
  methods: {
    getListChange() {
      this.listChange = [];
      this.isLoadingChange = true;

      this.isLoadingChange = false;

      const res = CoinPrice.Change.USD;
      this.listChange = res;
    },

    getListCoin() {
      this.listCoin = [];
      this.isLoadingCoin = true;
      // Get list coin from api
      axios
        .get("http://localhost:8081/v1/coins")
        .then((res) => {
          // Handle response after call api
          this.listCoin = res;
          if (res.length > 0) {
            this.onSelectCoin(res[0]);
          }
          this.isLoadingCoin = false;
        })
        .catch(() => {
          this.isLoadingCoin = false;
          alert("Lỗi lấy danh sách Coin");
        });
    },

    onSelectCoin(iCoin) {
      this.currentCoin = iCoin.Symbol;
      if (this.refInterval) {
        clearInterval(this.refInterval);
      }
      this.getDetailCurrentCoin();
    },

    getDetailCurrentCoin() {
      // Repeat function after 5second
      this.isLoadingCurrentCoin = true;
      this.listExchange = []
      console.log('listExchangelistExchange', this.listExchange)
      this.refInterval = setInterval(() => {
        // Call api get detail currentCoin: price,...
        axios
          .get(`http://localhost:8081/v1/symbol/${this.currentCoin}`)
          .then((res) => {
            this.isLoadingCurrentCoin = false;
            this.listExchange = res.Exchanges;
            this.detailCurrentCoin = {
              price: res.Coin.Price,
              yesterdayPrice: res.Coin.PriceYesterday || 0,
              timeUpdate: res.Coin.Time,
              VolumeYesterdayUSD: res.Coin.VolumeYesterdayUSD || 0,
            };
          })
          .catch(() => {
            this.isLoadingCurrentCoin = false;
          });
      }, 1000);
    },

    formatTime(t) {
      return moment(t).format('DD/MM/YYYY HH:mm:ss')
    }
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

.view-exchange {
  width: 100%;
  justify-content: space-evenly;
  padding-block: 10px;
  border-bottom: 1px solid #d5d5d5;
  display: flex;
}
</style>