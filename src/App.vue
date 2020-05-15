<template>
  <div id="app">
    <TitleComponent />
    &nbsp;
    <b-button v-bind:variant="ButtonStyle" @click="ToggleRealtimeData()">{{Action}}</b-button>
    &nbsp;
    <b-button @click="Refresh()">Refresh</b-button>
    &nbsp;
    <line-chart :height="80" :chart-data="datacollection" id="mychart"></line-chart>
    <h1>Current Stocks:</h1>
    <div v-for="(stock, index) in stocks_info" :key="index">
      <h3 style="text-align:left;float:left;">{{stock.company_name}} ({{stock.ticker}})</h3> 
      <a style="text-align:left">{{stock.quote_base}}</a> 
      <hr style="clear:both;"/>
    </div>
    
    <h1>Exchanges:</h1>
    <div v-for="(exch, index) in exchange_info" :key="index">
      <h3>{{exch.name}} ({{exch.exchange_ticker}})</h3>
      <a>{{exch.country}}, ({{exch.address}})</a>
    </div>
   
  </div>
</template>

<script>
import LineChart from "./LineChart.js";
import TitleComponent from './components/TitleComponent';


import io from 'socket.io-client';

var socket = io('wss://le-18262636.bitzonte.com', {path: '/stocks'});  

var lbls_array = [];

var stocks_values_dict = {};

var stocks_datasets = [];


function getRandomColor() {
  var letters = '0123456789ABCDEF';
  var color = '#';
  for (var i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}

export default {
  name: 'App',
  components: {
    TitleComponent,
    LineChart
  },
  data(){
    return {
      datacollection: {},
      stocks_info: [],
      exchange_info: [],
      myToggle: false,
      Action: "Start",
      ButtonStyle: "outline-primary"
    };
  },
  methods: {

    fillData (lbls_array, stocks_datasets) {
      this.datacollection = {
        labels: lbls_array,
        datasets: stocks_datasets
      }
    },

    GetExchangeInfo() {
      socket.emit('EXCHANGES', /* */);
      socket.on("EXCHANGES", (data)=>{
        this.exchange_info = data;
      })
    },

    GetStockInfo() {
      socket.emit('STOCKS', /* */);
      socket.on("STOCKS", (data)=>{
        this.stocks_info = data;
      })
    },

    SocketUpdate(data){
      //this.fillData(fetchedData)
      console.log("Update:\t", data);
      var date = new Date(data.time).toLocaleString("es-CL")
      //reviso si deberia agregarlo a la lista o ya está:

      if(!lbls_array.length){
        //empty
          lbls_array.push(date)
      }else{
        //not empty
        if (lbls_array[lbls_array.length - 1] != date){
          lbls_array.push(date)
        }
      }

      if (!(data.ticker in stocks_values_dict)){
        //creo nueva lista de valores para nuevo stock
        var values = [data.value];
        stocks_values_dict[data.ticker] = values;

        var color = getRandomColor();

        //creamos el dataset para nuevo stock
        var dataset = {};
        dataset["label"] = data.ticker;
        dataset["data"] = stocks_values_dict[data.ticker];
        dataset["backgroundColor"] = color;
        dataset["borderColor"] = color;
        dataset["fill"] = false;

        //agregamos el dataset del nuevo stock a la lista de datasets
        stocks_datasets.push(dataset);
      }
      else{
        //solo agregamos el dato al dataset del stock
        stocks_values_dict[data.ticker].push(data.value);
      }

      this.fillData(lbls_array, stocks_datasets);
    },

    StartRealtimeData(){
      socket.on("UPDATE", (data)=>{
        this.SocketUpdate(data);
      });
    },

    ToggleRealtimeData() {  

      if (this.Action == "Start"){
        //Variables Setup
        this.Action = "Stop"
        this.ButtonStyle = "danger"
        this.GetStockInfo();
        this.GetExchangeInfo();
        this.StartRealtimeData();

      }
      else{
        this.Action = "Start"
        this.ButtonStyle = "outline-primary"
        this.StopRealtimeData()
      }
    },

    StopRealtimeData() {  
      socket.removeAllListeners("UPDATE");
      console.log(stocks_values_dict);
    },

    Refresh() {
      if (this.Action == "Start"){
        lbls_array = [];
        stocks_datasets = [];
        stocks_values_dict = {};
        this.exchange_info = [];
        this.stocks_info = [];

        this.fillData(lbls_array, stocks_datasets);
      }
      else{
        lbls_array = [];
        stocks_datasets = [];
        stocks_values_dict = {};
        this.fillData(lbls_array, stocks_datasets);
      }
    },

    getRandomInt () {
      return Math.floor(Math.random() * (50 - 5 + 1)) + 5
    },
  }
};
</script>

<style>
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
  body {
    font-family: Arial, Helvetica, sans-serif;
    line-height: 1.4;
  }
  .small {
  max-width: 400px;
  margin: 150px auto;
  }
  .chartWrapper {
    position: relative;
  }

  .chartWrapper > canvas {
      position: absolute;
      left: 0;
      top: 0;
      pointer-events:none;
  }

  .chartAreaWrapper {
      width: 600px;
      overflow-x: scroll;
  }
</style>
