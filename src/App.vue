<template>
  <div id="app">
    <div>
      <b-navbar toggleable="lg" type="dark" variant="dark">
        <b-navbar-brand href="#">Tarea 3 IIC3103</b-navbar-brand>
        <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
        <b-collapse id="nav-collapse" is-nav>
          <b-navbar-nav>
            <b-nav-item>
              <b-button v-bind:variant="ButtonStyle" @click="ToggleRealtimeData()"> <b-icon icon="power"></b-icon> {{Action}}</b-button>
            </b-nav-item>
            <b-nav-item>
              <b-button @click="Refresh()"> <b-icon icon="arrow-clockwise"></b-icon> Refresh</b-button>
            </b-nav-item>
          </b-navbar-nav>
          <div class="mx-auto">
            <h2 class='text-light' >Welcome to IIC3103 Exchange <b-icon icon="graph-up"></b-icon> </h2>
          </div>
          <!-- Right aligned nav items -->
          <b-navbar-nav class="ml-auto">
            <b-nav-form>
              <b-form-input size="sm" class="mr-sm-2" placeholder="Search Stock"></b-form-input>
              <b-button size="sm" class="my-2 my-sm-0" type="submit">Search</b-button>
            </b-nav-form>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </div>

    <br>

    <b-container fluid>
      <b-row class="px-3">
        <b-col class="border border-dark" cols="9">
          <h2>Exchanges:</h2>   
            <div v-for="(exch, index) in exchange_info" :key="index" class="px-2">
              <b-row>
              <b-col class="border border-dark" cols="9">
                <h3>{{exch.name}} ({{exch.exchange_ticker}})</h3>
                <a>{{exch.country}}, ({{exch.address}})</a>
                <line-chart :height="200" :chart-data="datacollection" id="mychart"></line-chart>
                <br>
              </b-col>
              <b-col class="border border-dark" cols="3">
                <h3>Indicadores</h3>
                 <b-list-group>
                  <b-list-group-item>
                    Indicador 1
                  </b-list-group-item>
                  <hr style="clear:both;"/>
                  <b-list-group-item>
                    Indicador 2
                  </b-list-group-item>
                  <hr style="clear:both;"/>
                  <b-list-group-item>
                    Indicador 3
                  </b-list-group-item>
                </b-list-group>
                {{exch}}
              </b-col>
              </b-row>
              <hr style="clear:both;"/>
            </div>
        </b-col>
        <b-col class="border border-dark">
          <h2>Current Stocks:</h2>
          <div v-for="(stock, index) in stocks_info" :key="index">

            <b-card body-class="text-center" header-tag="nav">
              <template v-slot:header>
                <b-nav card-header tabs>
                  <b-nav-item active>Info</b-nav-item>
                  <b-nav-item>Gráfico</b-nav-item>
                  <b-nav-item>Indicadores</b-nav-item>
                </b-nav>
              </template>

              <b-card-text>
                
                <b-avatar variant="primary" class="mr-3" size="4em">{{stock.ticker}}</b-avatar>
                <h3>{{stock.company_name}}</h3>
                <strong>País: {{stock.country}} </strong>
                <br>
                <strong>Moneda: {{stock.quote_base}} </strong>
              </b-card-text>

            </b-card>
            <br>     
          </div>
        </b-col>
      </b-row>
    </b-container>

    <!-- Footer -->

    <!-- Footer -->

  </div>
</template>

<script>
import LineChart from "./LineChart.js";


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

    onClick(value) {
      this.$bvModal.msgBoxOk('User name: Fred Flintstone', {
        title: value,
        size: 'sm',
        buttonSize: 'sm',
        okVariant: 'success',
        headerClass: 'p-2 border-bottom-0',
        footerClass: 'p-2 border-top-0'
      })
    },


    fillData (lbls_array, stocks_datasets) {
      this.datacollection = {
        labels: lbls_array,
        datasets: stocks_datasets
      }
    },

    GetExchangeInfo() {
      socket.emit('EXCHANGES', /* */);
      socket.once("EXCHANGES", (data)=>{
        this.exchange_info = data;
      })
    },

    GetStockInfo() {
      socket.emit('STOCKS', /* */);
      socket.once("STOCKS", (data)=>{
        this.stocks_info = data;
      })
    },

    SocketUpdate(data){
      //this.fillData(fetchedData)
      console.log("Update:\t", data);
      var date = new Date(data.time).toLocaleTimeString("es-CL")
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
        this.exchange_info = [];
        this.stocks_info = [];
      }
      this.exchange_info = [];
      this.stocks_info = [];
      lbls_array = [];
      stocks_datasets = [];
      stocks_values_dict = {};
      if (this.Action == "Stop"){
        this.GetStockInfo();
        this.GetExchangeInfo();
      }
      this.fillData(lbls_array, stocks_datasets);
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
