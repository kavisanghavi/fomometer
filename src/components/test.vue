<template>
  <div class="hello">
    <form>
      <input type="text" id="ticker" v-model="ticker">
       <button type="submit" @click="submit">Find out how much you lost</button>
       <br>
       <div><Socket v-if="loading"></Socket></div>
    </form>
    <div v-if="submitted">
      <h1>{{name}}</h1>
      <br>
      <img :src="logo">
      <ul>
        <li>Ipo Date - {{ipo_date}}</li>
        <li>Stock price on IPO day - ${{priceOnIpoDay}}</li>
        <li>Stock price last market day - ${{lastClose}}</li>
      </ul>
      <h1 v-if="totalChange<0">If you invested $1000 for {{name}}'s IPO, you could have lost <span style="color:red"> ${{Math.abs(totalChange).toLocaleString()}}</span> over {{diff}} {{diff_time_unit}} which is loss of <span style="color:red">{{parseFloat(growth).toLocaleString()}}%</span> </h1>
      <h1 v-else>If you invested $1000 for {{name}}'s IPO, you could have made  <span style="color:green">${{Math.abs(totalChange).toLocaleString()}}</span> over {{diff}} {{diff_time_unit}} which is growth of <span style="color:green">{{parseFloat(growth).toLocaleString()}}% </span></h1>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import {Socket} from 'vue-loading-spinner'
export default {
  components: {
      Socket
    },
  name: 'Test',
  data() {
    return {
      response:{},
      ticker:'',
      ipo_date:'',
      name:'',
      logo:'',
      lastClose: 0,
      priceOnIpoDay:0,
      totalChange:0,
      submitted:false,
      diff:'',
      diff_time_unit:'',
      growth:0,
      loading:false
    }
  },
  methods:{
    submit: function(){
      this.loading=true
      this.submitted=false
      const compProfile = "https://finnhub.io/api/v1/stock/profile2"
      const stockdata="https://finnhub.io/api/v1/stock/candle";
      const currentPrice = "https://finnhub.io/api/v1/quote"
      console.log(stockdata)
      const profileParams={
      params:{
        symbol:this.ticker,
        token:"brdgavnrh5rft913smdg",
      }
    }
    axios.get(currentPrice,profileParams)
        .then(respCurrent=>{
          this.lastClose=respCurrent.data.c
          })
    axios.get(compProfile,profileParams)
    .then(resp=>{
      this.response=resp.data
      this.name=resp.data.name
      this.logo=resp.data.logo
      this.ipo_date=resp.data.ipo
      return resp
      }).then(compDetails=>{
        const date1 = new Date(compDetails.data.ipo).getTime() / 1000;
        const date2 = (new Date(compDetails.data.ipo).getTime() + 86400000) / 1000;
        console.log(date1,date2)
        const getPriceQuoteParams={
          params:{
            symbol:this.ticker,
            token:"brdgavnrh5rft913smdg",
            from: date1,
            to: date2,
            resolution:"D"
          }
        }
        axios.get(stockdata,getPriceQuoteParams)
        .then(resp3=>{
          console.log("Final Resp",resp3)
          this.priceOnIpoDay=resp3.data.o[0]
          const initialShares = parseInt(1000/this.priceOnIpoDay)
          this.totalChange = parseFloat(initialShares*(this.lastClose - this.priceOnIpoDay)).toFixed(2)
          const d1 = new Date(this.ipo_date)
          const d2 = new Date()
          this.diff = Math.floor((d2 - d1) / (1000*60*60*24))
          this.diff_time_unit="Days"
          if(this.diff>365){
            this.diff=parseFloat(this.diff/365).toFixed(2)
            this.diff_time_unit="Years"
          }
          this.growth=parseFloat(((this.totalChange-1000)/1000)*100).toFixed(2)
          this.submitted=true
          this.loading=false
          })
      })
    .catch(err=>console.log(err))

    }

  },
  mounted(){

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.spinner--socker{
  margin:0 auto;
  margin-top:20px;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
</style>
