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
              <div v-if="Object.keys(Stocks).length != 0">
                <b-button @click="Refresh()"> <b-icon icon="arrow-clockwise"></b-icon> Refresh</b-button>
              </div>
            </b-nav-item>
          </b-navbar-nav>
          <div class="mx-auto">
            <h3 class='text-light' >Welcome to IIC3103 Exchange <b-icon icon="graph-up"></b-icon> </h3>
          </div>
          <div class="mx-auto">
            <h5 class='text-light' >{{ fecha_hora_actual }}</h5>
          </div>
          <!-- Right aligned nav items -->
          <b-navbar-nav class="ml-auto">
            <b-nav-form>
              <b-form-input size="sm" class="mr-sm-2" placeholder="Search Stock"></b-form-input>
              <b-button size="sm" v-b-modal.modal-2>Buscar</b-button>
            </b-nav-form>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </div>
    <br>
    <b-container fluid>
      <div v-if="Object.keys(Stocks).length == 0">
        <div>
          <b-jumbotron>
            <template v-slot:header>Bienvenido!</template>

            <template v-slot:lead>
              Aquí podrás consultar información de distintas bolsas de valores y acciones en tiempo real.
            </template>
            <hr class="my-4">
            <b-row>
              <b-col class="rounded" cols="9">
                <b-card title="Exchanges" border-variant="primary">
                  <b-card-text>
                    A este lado podrás ver toda la información sobre los Mercados de Valores activos
                  </b-card-text>
                  <b-card-text>
                    Navega en las pestañas para ver más información!
                  </b-card-text>
                </b-card>
              </b-col>
              <b-col class="rounded" cols="3">
                <b-card title="Stocks" border-variant="secondary">
                  <b-card-text>
                    A este lado podrás ver la información sobre las acciones.
                  </b-card-text>
                </b-card>
              </b-col>
            </b-row>
            <hr class="my-4">
            <p>Actualmente estás desconectado</p>
            <b-button v-b-modal.modal-1>Ayuda</b-button>
          </b-jumbotron>
        </div>
      </div>
      <div v-else>
        <b-row class="px-1">
          <b-col class="rounded" cols="9">
            <h2>{{ Object.keys(Exchanges).length }} Exchanges activos:</h2>
            <div v-for="(exch, index) in Exchanges" :key="index">
              <b-card border-variant="primary">
                <b-tabs card>
                  <b-tab title="Información" active>
                    <b-row>
                      <b-col class="rounded" cols="9">
                        <h3>{{exch.name}} ({{exch.exchange_ticker}})</h3>
                        <a>{{exch.country}}, ({{exch.address}})</a>
                        <!-- You will not be able to see this text. -->
                        <line-chart :height="200" :chart-data="datacollection" id="mychart"></line-chart>
                      </b-col>
                      <b-col class="rounded" cols="3">
                        <h3>Indicadores</h3>
                        <b-list-group>
                          <b-list-group-item>
                            <strong title="Cantidad de Acciones"> <b-icon icon="hash"></b-icon>Acciones: {{ exch.num_stocks() }} </strong>
                          </b-list-group-item>
                          <hr style="clear:both;"/>
                          <b-list-group-item>
                            <strong title="Volumen de Compra"> <b-icon icon="arrow-down"></b-icon>Compra: {{ exch.buy_vol() }} </strong>
                          </b-list-group-item>
                          <hr style="clear:both;"/>
                          <b-list-group-item>
                            <strong title="Volumen de Venta"> <b-icon icon="arrow-up"></b-icon>Venta: {{ exch.sell_vol() }} </strong>
                          </b-list-group-item>
                          <hr style="clear:both;"/>
                          <b-list-group-item>
                            <strong title="Volumen Total Transado"> <b-icon icon="arrow-up-down"></b-icon>Total: {{ exch.total_vol() }} </strong>
                          </b-list-group-item>
                          <hr style="clear:both;"/>
                          <b-list-group-item>
                            <strong title="MarketShare"> <b-icon icon="pie-chart-fill"></b-icon> MarketShare: {{ exch.marketshare() }} </strong>
                          </b-list-group-item>
                        </b-list-group>
                      </b-col>
                    </b-row>
                  </b-tab>
                  <b-tab title="Gráfico Volumen">
                    <b-card-text>
                      Esto era un objetivo extra...
                      <br>
                      Sobre el gráfico de las acciones, solo me funcionó bien mostrando todas, por culpa de este
                      <a href="https://github.com/chartjs/Chart.js/issues/3753">error</a>
                      <br>
                      <strong>Protip:</strong>
                      puedes apretar los labels del gráfico para activar/desactivar cada acción :)
                    </b-card-text>
                  </b-tab>
                </b-tabs>
              </b-card>
            <hr style="clear:both;"/>
            </div>
          </b-col>
          
          <b-col class="rounded" cols="3">
            <h2>{{ Object.keys(Stocks).length }} Stocks activos:</h2>
            <div v-for="(stock, index) in Stocks" :key="index">
              <b-card border-variant="secondary">
                <b-tabs card>
                  <b-tab title="Info" active>
                    <b-card-text>
                      <b-row class="px-3">
                        <b-col class="rounded" cols="6">
                          <b-avatar variant="primary" class="mr-3" size="4em">{{stock.ticker}}</b-avatar>
                          <h5>{{stock.company_name}}</h5>
                          <p title="País" ><b-icon icon="flag-fill"></b-icon>{{stock.country}} </p>
                          <hr style="clear:both;"/>
                          <strong title="Moneda"> <b-icon icon="wallet"></b-icon> {{stock.quote_base}} </strong>
                          <hr style="clear:both;"/>
                          <strong title="Volumen Total Transado"><b-icon icon="arrow-up-down"></b-icon> {{stock.total_vol()}} </strong>
                        </b-col>
                        <b-col class="rounded" cols="6">
                          <strong title="Alto Histórico"> <b-icon icon="chevron-double-up"></b-icon> {{stock.all_time_high}} </strong>
                          <hr style="clear:both;"/>
                          <strong title="Bajo Histórico"><b-icon icon="chevron-double-down"></b-icon> {{stock.all_time_low}} </strong>
                          <hr style="clear:both;"/>
                          <strong title="Último precio"><b-icon icon="skip-backward"></b-icon> {{stock.last_price}} </strong>
                          <hr style="clear:both;"/>
                          <strong title="Variación porcentual"><b-icon icon="exclamation-triangle"></b-icon> {{stock.var_percent}} </strong>
                          <hr style="clear:both;"/>
                          <div v-if="!stock.subscribed">
                            <b-button v-b-tooltip.hover title="Notificaciones de altos y bajos históricos" size="sm" :pressed.sync="stock.subscribed" variant="outline-info">Subscribe</b-button>
                          </div>
                          <div v-else>
                            <b-button size="sm" :pressed.sync="stock.subscribed" variant="outline-info">Unsubscribe</b-button>
                          </div>
                        </b-col>
                      </b-row> 
                    </b-card-text>
                  </b-tab>
                  <b-tab title="Gráfico">
                    Me tiraba muchos
                      <a href="https://github.com/chartjs/Chart.js/issues/3753">errores</a>
                    que al parecer no han arreglado :(
                    <strong>Protip:</strong>
                    puedes apretar los labels del gráfico de Exchanges para activar/desactivar cada acción :)
                  </b-tab>
                  <b-tab title="Transado">
                    <strong title="Volumen Compra"> Volumen Compra: {{stock.buy_vol}} </strong>
                    <hr style="clear:both;"/>
                    <strong title="Volumen Venta"> Volumen Venta: {{stock.sell_vol}} </strong>
                    <hr style="clear:both;"/>
                    <strong title="Volumen Total"> Volumen Total: {{stock.total_vol()}} </strong>
                    <hr style="clear:both;"/>
                  </b-tab>
                </b-tabs>
              </b-card>
              <hr style="clear:both;"/>
            </div>
          </b-col>
        </b-row>
      </div>
    </b-container>

    <!-- Footer -->
    <footer>
    <div class="footer-copyright text-center py-3">
      Tarea 3 - IIC3103 - made with ❤️ by Daniel Darritchon 2020
    - <a href="https://github.com/dedarritchon/Tarea3IIC3103">Github</a>
    </div>
    </footer>
    <!-- Footer -->

    <b-modal id="modal-1" title="IIC3103 Exchange">
      <p class="my-4">
        Hola! Para conectarte, pulsa el botón
        <b-button disabled variant="primary"> <b-icon icon="power"></b-icon> Start</b-button>
        <br>
        <i>Hint: está arriba a la izquierda</i>
        <br>
        <br>
        Puedes actualizar la información con el botón
        <b-button disabled> <b-icon icon="arrow-clockwise"></b-icon> Refresh</b-button>
        <br>
        <br>
        Para desconectarte, pulsa el botón
        <b-button disabled variant="danger"> <b-icon icon="power"></b-icon> Stop</b-button>
      </p> 
    </b-modal>

    <b-modal id="modal-2" title="IIC3103 Exchange">
      <b-img src="https://i.ytimg.com/vi/Wo9Y03EmvHY/maxresdefault.jpg" fluid alt="Responsive image"></b-img>
    </b-modal>

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
      fecha_hora_actual: "Estás desconectado",
      datacollection: {},
      stocks_info: [],
      myToggle: false,
      Action: "Start",
      ButtonStyle: "outline-primary",
      //Variables Para el cambio de estructura
      Stocks: {},
      Exchanges: {},
    };
  },

  methods: {

    makeHighToast(ticker, value) {
      this.$bvToast.toast(`La acción llego a un nuevo alto histórico: ${value}`, {
        title: 'Alto Histórico de ' + ticker,
        autoHideDelay: 4000,
        variant: "success",
        appendToast: false
      })
    },

    makeLowToast(ticker, value) {
      this.$bvToast.toast(`La acción llego a un nuevo bajo histórico: ${value}`, {
        title: 'Bajo Histórico de ' + ticker,
        autoHideDelay: 4000,
        variant: "danger",
        appendToast: false
      })
    },

    GetExchangeInfo() {
      socket.emit('EXCHANGES', /* */);
      socket.once("EXCHANGES", (data)=>{
        this.exchange_info = data;
        for (var key in data) {
          this.CreateExchangeObjects(data[key])
        }
      })
    },

    CreateExchangeObjects(exch){
      exch["stocks"] = {}
      for (var key in this.Stocks) {
        if (exch.listed_companies.includes(this.Stocks[key]["company_name"])){
          exch["stocks"][this.Stocks[key]["ticker"]] = this.Stocks[key]
        }
      }
      exch["buy_vol"] = function() {
        var buy_vol_sum = 0;
        for (var key in exch["stocks"]) {
          buy_vol_sum += exch["stocks"][key]["buy_vol"]
        }
        return buy_vol_sum
      }
      exch["sell_vol"] = function() {
        var sell_vol_sum = 0;
        for (var key in exch["stocks"]) {
          sell_vol_sum += exch["stocks"][key]["sell_vol"]
        }
        return sell_vol_sum
      }
      exch["total_vol"] = function() {
        var total_vol_sum = 0;
        for (var key in exch["stocks"]) {
          total_vol_sum += exch["stocks"][key].total_vol()
        }
        return total_vol_sum
      }
      exch["num_stocks"] = function() {
        return Object.keys(exch["stocks"]).length
      }
      var self = this;
      exch["marketshare"] = function() {
        var total_vol = 0
        for (key in self.Exchanges){
          total_vol += self.Exchanges[key].total_vol()
        }
        return (exch.total_vol()/total_vol).toFixed(2) + '%'
      }
      
      this.Exchanges[exch["exchange_ticker"]] = exch;
    },

    GetStockInfo() {
      socket.emit('STOCKS', /* */);
      socket.once("STOCKS", (data)=>{
        this.stocks_info = data;
        data.forEach(this.CreateStockObjects)
      })
    },

    CreateStockObjects(stock){
      //creo Objeto stock y lo guardo en Stocks
      stock["buy_vol"] = 0;
      stock["sell_vol"] = 0;
      stock["all_time_high"] = 0;
      stock["all_time_low"] = 0;
      stock["last_price"] = "-";
      stock["var_percent"] = "-";
      stock["subscribed"] = false;
      stock["total_vol"] = function() {
        var total_vol_sum = 0;
          total_vol_sum += stock["buy_vol"]
          total_vol_sum += stock["sell_vol"]
        return total_vol_sum
      }
      this.Stocks[stock["ticker"]] = stock;
    },

    SocketBuy(data){
      console.log("Buy:\t", data);
      if (data.ticker in this.Stocks){
        this.Stocks[data.ticker]["buy_vol"] += data.volume
      }
    },

    SocketSell(data){
      console.log("Sell:\t", data);
      if (data.ticker in this.Stocks){
        this.Stocks[data.ticker]["sell_vol"] += data.volume
      }
    },

    SocketUpdate(data){
      console.log("Update:\t", data);
      this.fecha_hora_actual = new Date(data.time).toLocaleString("es-CL")

      //Manejo de fechas (Todos los gráficos tendrán las mismas fechas)
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
      //Nueva version
      if (data.ticker in this.Stocks){
        //Indicadores Stock
        if (data.value > this.Stocks[data.ticker]["all_time_high"]){
          this.Stocks[data.ticker]["all_time_high"] = data.value
          if (this.Stocks[data.ticker]["subscribed"]){
            this.makeHighToast(data.ticker, data.value)
          }
        }
        if (this.Stocks[data.ticker]["all_time_low"] == 0){
          this.Stocks[data.ticker]["all_time_low"] = data.value
        }
        if (data.value < this.Stocks[data.ticker]["all_time_low"]){
          this.Stocks[data.ticker]["all_time_low"] = data.value
          if (this.Stocks[data.ticker]["subscribed"]){
            this.makeLowToast(data.ticker, data.value)
          }
        }
        const last_price = this.Stocks[data.ticker]["last_price"]
        if (last_price != "-"){
          this.Stocks[data.ticker]["var_percent"] = ((100 * (last_price - data.value))/last_price).toFixed(2) + '%'
        }
        this.Stocks[data.ticker]["last_price"] = data.value;
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

      this.datacollection = {
        labels: lbls_array,
        datasets: stocks_datasets
      }
    },

    StartRealtimeData(){
      socket.on("UPDATE", (data)=>{
        this.SocketUpdate(data);
      });
      socket.on("BUY", (data)=>{
        this.SocketBuy(data);
      });
      socket.on("SELL", (data)=>{
        this.SocketSell(data);
      });
    },

    ToggleRealtimeData() {  
      if (this.Action == "Start"){
        this.Action = "Stop"
        this.ButtonStyle = "danger"
        //Recibir Info de stocks y crear objetos
        this.GetStockInfo();
        //Recibo Info de Exchanges y crear objetos (Con referencias a las Stocks)
        this.GetExchangeInfo();
        //Socket Comienza a escuchar eventos update, buy y sell
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
      socket.removeAllListeners("BUY");
      socket.removeAllListeners("SELL");
    },

    SemiRefresh() {
      this.GetStockInfo();
      this.GetExchangeInfo();
    },

    TotalRefresh() {
      this.stocks_info = [];
      this.Stocks = {};
      this.Exchanges = {};
      this.StocksData = {};
      this.fecha_hora_actual =  "Estás desconectado";
    },

    Refresh() {
      if (this.Action == "Start"){
        this.TotalRefresh()
      }
      else{
        this.SemiRefresh()
      }
      lbls_array = [];
      stocks_datasets = [];
      stocks_values_dict = {};

      this.datacollection = {
        labels: lbls_array,
        datasets: stocks_datasets
      }
      
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
