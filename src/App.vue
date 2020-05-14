<template>
  <div id="app">
    <TitleComponent />
    <button @click="StartRealtimeData()">Start Connection</button>
    &nbsp;
    <button @click="StopRealtimeData()">Stop Connection</button>
    &nbsp;
    <button @click="Restart()">Restart</button>

    <line-chart :width="300" :height="100" :chart-data="datacollection" id="mychart"></line-chart>  

   
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


/* 

stocks_dict = {}
stocks_dict["FB"] = [1, 2 , 3]

Dataset = [] con cosas de este estilo
{
  label: 'FB',
  backgroundColor: 'green',
  data: [1, 2, 3]
}

*/


export default {
  name: 'App',
  components: {
    TitleComponent,
    LineChart
  },
  data(){
    return {
      datacollection: {}
    };
  },
  methods: {

    fillData (lbls_array, stocks_datasets) {
      this.datacollection = {
        labels: lbls_array,
        datasets: stocks_datasets
      }
    },

    StartRealtimeData() {  
      socket.on("UPDATE", (data)=>{
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
      });
    },

    StopRealtimeData() {  
      socket.removeAllListeners("UPDATE");
      console.log(stocks_values_dict);
    },

    Restart() {
      lbls_array = [];
      stocks_datasets = [];
      stocks_values_dict = {};
      this.fillData(lbls_array, stocks_datasets);
    },

    getRandomInt () {
      return Math.floor(Math.random() * (50 - 5 + 1)) + 5
    }

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
