<!--  -->
<template>
  <div class=''>
    <div>法拍房列表</div>
    <div>
      
    </div>
  </div>
</template>

<script>
import Axios from 'axios';

function handleJsonp(data) {
  console.log('handle:', data)
}

// {
//         id:0,
//         name:'',
//         startPrice:0,

//         bidCount:0,
//         endTime:0,
//         currentPrice:0,
//         bidList:[],
//         accessEnsureNum:0, // 报名人数
//         accessNum:0,// 围观人数
//         auctionStatus:0,// 0未开始，1已开始，2已结束
//         currentBidUserNumber:'',
//       }
// Axios.defaults.baseURL = 'https://api.m.jd.com'
// Axios.defaults.headers.common['Referer'] = 'https://paimai.jd.com/'
// headers: {
// Referer: 'https://paimai.jd.com/'
// }
class PaiInfo {
  constructor(id, name, startPrice) {
    this.id = id;
    this.name = name;
    this.startPrice = startPrice;
    this.timer = null;
    this.start = 0;
    this.end = 9;

    this.currentPrice = startPrice;
    this.endTime = 1697680800000;

    this.bidCount = 0;
    this.bidList = [];
    this.accessEnsureNum = 0; // 报名人数
    this.accessNum = 0;// 围观人数
    this.auctionStatus = 0;// 0未开始，1已开始，2已结束
    this.currentBidUserNumber = '';

    if (Date.now() >= 1697594400000) {
      this.auctionStatus = 1;
    }
  }
}

export default {
  name: '',
  components: {

  },
  data() {
    return {
      paiList: [

      ],

    }
  },
  computed: {},
  watch: {},
  methods: {
    init() {
      this.paiList = [
        new PaiInfo(298642201, '28栋1单元30层3002号', 3466171),
      ];
    },
     initData() {
      for (let i = 0; i < this.paiList.length; i++) {
         this.getRealData(i)
      }
      // this.getRealData
    },
    async getRealData(i) {
      let info = this.paiList[i];
      let paimaiId = info.id, end = info.end, start = info.start;
      let jsonpData = `jsonp_${Date.now()}_${parseInt(Math.random() * 99999)}`
      let res = await Axios.get('api', {
        params: {
          appid: 'paimai',
          functionId: 'getPaimaiRealTimeData',
          body: JSON.stringify({ "end": end, "paimaiId": paimaiId, "source": 0, "start": start }),
          loginType: 3,
          jsonp: jsonpData,
        },
      })
      if (typeof res.data === 'string' && res.data.indexOf(jsonpData) > -1) {
        let doJs = res.data || ''
        doJs = doJs.replace(jsonpData + '(', '')
        doJs = doJs.replace(');', '')
        let raw = JSON.parse(doJs)
        if (raw.message === '成功') {
          let obj = raw.data
          info.currentPrice = obj.currentPrice || info.currentPrice;
          info.endTime = obj.endTime || info.endTime;
          info.bidCount = obj.bidCount || 0;
          info.bidList = obj.bidList || [];
          info.accessEnsureNum = obj.accessEnsureNum; // 报名人数
          info.accessNum = obj.accessNum;// 围观人数
          info.auctionStatus = obj.auctionStatus;// 0未开始，1已开始，2已结束
          info.currentBidUserNumber = obj.currentBidUserNumber;
        }
      }
      if(this.currentBidUserNumber >= 2){
        // return
      }

      let timeout = parseInt(Math.random() * 3000) + 4000

      // 成交总数不等于已经加载出来的 需要立即加载
      if (info.bidCount != info.bidList.length) {
        timeout = 1000
        info.end = parseInt(info.bidCount/10)*10+9
        this.timer = this.loopTimer(timeout, i);
        return;
      }
      // 结束位-成交总数 < 5 结束位+10 保持结束位始终大于成交总数10
      if (info.end - info.bidCount <= 5) {
        info.end += 10
      }
      console.log(info)
      this.timer = this.loopTimer(timeout, i);
    },
    loopTimer(timeout, index) {
      return setTimeout(() => {
        this.getRealData(index)
      }, timeout)
    },
  },
  created() {
    this.init();
  },
  mounted() {
    // this.initData()
  }
}
</script>
<style scoped>
</style>
