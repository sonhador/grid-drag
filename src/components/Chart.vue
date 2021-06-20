<template>
  <div class="container" ref="container">
    <div class="chart" @mousedown="mousedown" @mouseover="mouseover" @mouseleave="mouseleave">
      <vue3-chart-js
          :id="doughnutChart.id"
          :type="doughnutChart.type"
          :data="doughnutChart.data"
      ></vue3-chart-js>
    </div>
  </div>
</template>

<script>
import Vue3ChartJs from '@j-t-mcc/vue3-chartjs'

export default {
  name: 'App',
  components: {
    Vue3ChartJs,
  },
  methods: {
    calcGridRowsCols: function() {
      const containerWidth = this.$refs.container.clientWidth;
      const containerHeight = this.$refs.container.clientHeight;

      const cellWidth = containerWidth / (20 * (containerWidth / containerHeight));
      const cellHeight = containerHeight / 20;

      let cols = [];
      let rows = [];

      for (let x = 0; x<containerWidth; x++) {
        if (x % cellWidth == 0) {
          cols.push(x);
          x += cellWidth;
          x--;
        }
      }
      
      for (let y = 0; y<containerHeight; y++) {
        if (y % cellHeight == 0) {
          rows.push(y);
          y += cellHeight;
          y--;
        }
      }

      this.gridRows = rows;
      this.gridCols = cols;
    },

    cellRowCol: function() {
      const eventX = this.targetSelectedX;
      const eventY = this.targetSelectedY;

      let col = 0;
      let row = 0;

      for (let x = 0; x<this.gridCols.length-1; x++, col++) {
        if (eventX >= this.gridCols[x] && eventX < this.gridCols[x+1]) {
          break;
        }
      }

      for (let y = 0; y<this.gridRows.length-1; y++, row++) {
        if (eventY >= this.gridRows[y] && eventY < this.gridRows[y+1]) {
          break;
        }
      }

      const coord = {row: row, col: col};

      return coord;
    },

    mousemove: function(e) {
      if (e.x < 0 || e.y < 0) {
        return;
      }

      this.targetSelectedX = e.x;
      this.targetSelectedY = e.y;

      const target = this.targetSelected;
      const rowCol = this.cellRowCol(e);

      target.parentNode.style.setProperty("grid-row", rowCol.row);
      target.parentNode.style.setProperty("grid-column", rowCol.col);
    },

    mouseup: function() {
      window.removeEventListener("mousemove", this.mousemove);
      window.removeEventListener("mouseup", this.mouseup);

      this.targetSelected.style.setProperty("cursor", "grab");

      this.unHighlight(this.targetSelected);
    },

    mousedown: function(e) {
      this.targetSelected = e.target;
      this.highlight(e.target);
      this.calcGridRowsCols();

      e.target.style.setProperty("cursor", "grabbing");

      window.addEventListener("mousemove", this.mousemove);
      window.addEventListener("mouseup", this.mouseup);
    },

    mouseover: function(e) {
      this.targetSelected = e.target;
      this.highlight(e.target);
    },

    mouseleave: function() {
      this.unHighlight(this.targetSelected);
    },

    highlight: function(elem) {
      elem.style.setProperty("border-width", "1px");
      elem.style.setProperty("border-color", "red");
      elem.style.setProperty("border-style", "solid");
      elem.style.setProperty("cursor", "grab");
    },

    unHighlight: function(elem) {
      elem.style.setProperty("border-width", "");
      elem.style.setProperty("border-color", "");
      elem.style.setProperty("border-style", "");
      elem.style.setProperty("cursor", "");
    },

  }, 
  setup () {
    const doughnutChart = {
      id: 'doughnut',
      type: 'doughnut',
      data: {
        gridRows: [],
        gridCols: [],
        targetSelected: {},
        targetSelectedX: 0,
        targetSelectedY: 0,
        labels: ['VueJs', 'EmberJs', 'ReactJs', 'AngularJs'],
        datasets: [
          {
            backgroundColor: [
              '#41B883',
              '#E46651',
              '#00D8FF',
              '#DD1B16'
            ],
            data: [40, 20, 80, 10]
          }
        ]
      }
    }
    
    return {
      doughnutChart
    }
  },
}
</script>

<style>
  .chart {
    height: 300px;
    width: 300px;
    /*
    display: flex;  
    flex-direction: column;
    */
  }

  .container {
    height: 900px;
    width: 1800px;
    display: grid;
    grid-template-columns: repeat(40, 1fr);
    grid-template-rows: repeat(20, 1fr);
  }
</style>
