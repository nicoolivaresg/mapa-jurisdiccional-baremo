<template>
  <div id="app">
    <div class="container">
      <div class="row search-row" style="justify-content: center">
        <h1 class="item" style="text-align: center;">{{title}}</h1>
      </div>
      <div class="row search-row" style="justify-content: left">
        <p class="item" style="text-align: justify; margin-top: 20px; margin-bottom: 20px;">{{subtitle}}</p>
      </div>
      <div class="search-row row d-md-flex justify-content-md-center align-items-md-center"  style="flex-wrap: wrap;">
        <img class="item" src="./assets/logo.png" alt="" style="min-width: 5%; width: 10%; height: auto;">
        <input 
          id="query" 
          type="search" 
          placeholder="Qué estas buscando..." 
          class="item form-control d-md-flex form-control" 
          style="min-width: 50%; width: 50%; margin: 5px;"
          @keyup="pressedKey"
          :disabled="loadingResults === true"
        >
        <button 
          id="searchBtn" 
          class="item btn btn-primary" 
          type="button" 
          style="margin: 5px; text-align: center; min-width: 120px; width: 120px;"
          @click="showResults()"
          :disabled="loadingResults === true"
        >
          Buscar
        </button>
        <!-- <select id="visualization" class="item form-control form-select" v-model="visualization" :disabled="currentData === null">
          <option disabled value="">Elige una visualización</option>
          <option value="bodyInjuriesMap">Mapa corporal</option>
          <option value="map">Mapa jurisdiccional</option>
        </select>
        <button 
          :disabled="currentData === null"
          id="visualizeBtn" 
          class="item btn btn-primary"
          style="margin: 5px; width: 120px;"
          @click="visualize()"
        >Visualizar</button> -->
      </div>
      <div class="row search-row">
        <div class="col" style="border: 5px;">
          <div class="row search-row">
            <h2 id="resultsCount" class="item" >Resultados</h2>
          </div>
          <div class="row search-row">
            <Loading v-if="loadingResults" text="Cargando resultados" class="item"/>
            <Loading v-if="loadingVisualization" text="Cargando visualización"/>
            <!-- <div id="results" class="item">
            </div> -->
          </div>
        </div>            
      </div>
      <hr style="width:100%;text-align:left;margin-left:5px">
      <div class="row search-row">
        <h2 class="item">Visualización</h2>
      </div>
    </div>
      <div id="canvas" ></div>
  </div>
</template>

<script>
import Loading from './components/Loading.vue'
import stare from '@nicolas.olivares.g/stare.js-client';
// import JSONFormatter from 'json-formatter-js';
import axios from 'axios';
import _ from 'lodash';
const STARE_API_URL = process.env.VUE_APP_STARE_HOST;

export default {
  name: 'App',
  components: {
    Loading
  },
  data(){
    return{
      title: 'Baremo Daño Moral: Accidentes Laborales y Enfermedades Profesionales (Stare.js)',
      subtitle: 'Esta herramienta demostrativa permite buscar en Baremo Daño Moral: Accidentes del Trabajo y Enfermedades Profesionales. Podrá ver los resultados a través de visualizaciones implementadas con Stare.js, una herramienta desarrollada en la Universidad de Santiago de Chile para la construcción de visualizaciones visuales de resultados de búsqueda.',
      loadingResults: false,
      loadingVisualization: false,
      engine: '',
      query:null,
      pageNumber: 50,
      resultsVisualization: '',
      results: '', 
      resultsCount: 0,
      metric: null, 
      library: 'd3',
      visualization: process.env.VUE_APP_VISUALIZATION, //map or bodyInjuriesMap
      visualizeBtn:null,
      canvas: null,

      currentData: null,
    }
  },
  mounted(){
    this.resultsCount = document.querySelector("#resultsCount");
    this.query = document.querySelector('#query');
    this.results = document.querySelector('#results');
    this.canvas = document.querySelector('#canvas');
  },
  methods:{
    pressedKey(e){
      
      if(e.key === "Enter") {
        this.currentData = null;
        this.showResults();
      }
    },

    getResults(engine, query, pageNumber){
      this.loadingResults = true;
      return new Promise((resolve, reject) => {
        axios.get(`${STARE_API_URL}?query=${query}&numberOfResults=${pageNumber}`)
          .then(response => resolve(_.get(response, 'data')))
          .catch(error => reject(error));
      })
    },

    showResults(){
      this.canvas.innerHTML = '';
      this.query.classList = '';

      if (this.query.value === '') {
        this.query.classList = 'error';
        return;
      }


      this.getResults(this.engine, this.query.value, this.pageNumber)
        .then(data => {
          // this.results.innerHTML = '';
          this.currentData = data;
          // let formatter = new JSONFormatter(data);
          this.loadingResults = false;
          // this.results.appendChild(formatter.render());
          if(this.currentData.numberOfItems > 0){
            this.resultsCount.innerHTML = `${this.currentData.numberOfItems} resultados para "${this.query.value}"`;
          }else{
            this.resultsCount.innerHTML = `Búsqueda para "${this.query.value}" no arrojó resultados`;
          }
          this.visualize();
        })
        .catch(err => {
          // this.results.innerHTML = err;
          console.log(err);
          alert("No se puede recuperar resultados por el momento. Reintente más tarde.")
          this.resetState();
        });
    },
    visualize(){
      this.loadingVisualization = true;
      if (_.isEmpty(this.currentData)) {
        alert('No hay resultados para visualizar, haga una búsqueda primero.');
        this.loadingVisualization = false;
        this.resetState();
        return;
      }

      let chart = null;
      chart = stare(this.library, this.visualization);
      

      if (chart) {
        document.querySelector('#canvas').innerHTML = '';
        
        if (this.visualization === 'bodyInjuriesMap') {
          chart('#canvas', this.currentData, {mainBackgroundColor: 'white'});
        } 
        if(this.visualization === 'map')
        {
          chart('#canvas', this.currentData, {});
          // chart('#canvas', this.currentData, {stroke: true, strokeColor: 'black'});
          // let svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
          // svg.setAttribute('id', 'svg');
          // this.canvas.appendChild(svg);
          // chart('#svg', this.currentData, {});
        }
      }else{
        alert("No se puede dibujar la visualización en este momento. Intente más tarde.");
        this.loadingVisualization=false;
        this.resetState();
      }
      this.loadingVisualization= false;
    },

    resetState(){
      this.canvas.innerHTML = '';
      this.query.classList = '';
      this.loadingResults=false;
      this.loadingVisualization=false;
      // this.results.innerHTML = "";
      this.resultsVisualization = "";
      this.resultsCount.innerHTML = "";
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  min-height: 100%;
  height: 100%;
}

.container-fluid{
  height: 100%;
}



.search-row{
  display: flex;
  justify-content: left;
  flex-flow: row wrap;
  width: 100%;
}

.search-row .item{
  margin: 5px;
}

html, body {
  height: 100%;
  width: 98vw;
}
div{
  position: relative;
}
#canvas{
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

area {
  display: flex;
  border-color: hotpink;
}


.container {
  padding: 10px;
  position: relative;
}

.selector-container{
  position: relative;
}
.row .col {
  float: left;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  padding: 0 .75rem;
  min-height: 1px;
}
.col.s6 {
  width: 50%;
  margin-left: auto;
  left: auto;
  right: auto;
}
.error {
  border-color: red;
}
pre {
  border-radius: 2px;
  border: 1px solid #ccc;
  padding: 5px;
  background-color: #f7f7f7;
  overflow: auto;
}

.centroid{
  fill: red;
  width: 100px;
  border-radius: 1%;
}
.stare__tooltip {
  position: absolute;
  padding: 4px;
  width: 100%;
  background-color: #f9f9f9;
  border: 1px solid;
  border-radius: 2px;
  pointer-events: none;
}
.stare__modal {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 4px;
  width: 500px;
  background-color: grey;
  border: 2px solid;
  border-radius: 2px;
  pointer-events: none;
}
/* // network node */
.mainNode {
  z-index: -1;
}

.svg-tiles {
  min-width: 100px;
  min-height: 100px;
  max-width: 400px;
  max-height: 400px;
  border: 1px solid black;
  display: inline-block;
  margin-right: 2px;
}
</style>
