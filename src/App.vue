<template>
  <div id="app">

    <div class="head">
      <label>
        Кількість точок
        <input type="number" min="3" minlength="1" step="1" v-model="settings.dot.count"/>
      </label>
      <label>
        Кількість петель()
        <input type="number" min="0" minlength="1" step="1" v-model="settings.ellipse.count"/>
      </label>
      <label>
        Кількість петель
        <input type="number" min="0" minlength="1" step="1" v-model="settings.loop.count"/>
      </label>

      <label>
        Діаметру вершини
        <input type="number" min="1" minlength="1" step="1" v-model="settings.dot.diameter"/>
      </label>
      <label>
        Товщини ребра
        <input type="number" min="1" minlength="1" step="1" v-model="settings.line.width"/>
      </label>
      <label>
        Колір ребра
        <input type="color" v-model="settings.line.color"/>
      </label>
      <label>
        Колір вершини
        <input type="color" v-model="settings.dot.color"/>
      </label>
      <label>
        <input type="button" @click="clearAll()" value="Очистити"/>
      </label>
    </div>
    <canvas :width="sizes.width" :height="sizes.height" ref="canvas"
            @mousedown="setDot"></canvas>

  </div>
</template>

<script>

  export default {
    name: 'app',
    computed: {
      sizes: function () {
        return {
          width: document.body.clientWidth,
          height: document.body.clientHeight - 80,
          diagonal: Math.sqrt(Math.pow(document.body.clientWidth, 2) + Math.pow((document.body.clientHeight - 80), 2))
        }
      }
    },
    data: function () {
      return {
        dots: [],
        dot: {
          x: null,
          y: null
        },
        ellipses: [],
        loops: [],
        ellipse: {
          x: null,
          y: null
        },
        loop: {
          from: {
            x: null,
            y: null
          },
          to: {
            x: null,
            y: null
          }
        },
        settings: {
          dot: {
            count: 5,
            color: '#000000',
            diameter: 10,
            default_: {
              x: null,
              y: null
            }
          },
          ellipse: {
            count: 1,
            default_: {
              x: null,
              y: null
            }
          },
          loop: {
            count: 2,
            default_: {
              from: {
                x: null,
                y: null
              },
              to: {
                x: null,
                y: null
              }
            }
          },
          line: {
            color: '#9c7afa',
            width: 3
          }
        },
        defaultData: {
          dots: [
            {
              x: 500,
              y: 350
            },
            {
              x: 700,
              y: 370
            },
            {
              x: 764,
              y: 220
            },
            {
              x: 590,
              y: 105
            },
            {
              x: 430,
              y: 196
            }
          ]
        },
        ctx: null
      }
    },
    methods: {
      drawLine: function (x_, y_, x, y) {
        this.ctx.beginPath();
        this.ctx.moveTo(x_, y_);
        this.ctx.lineTo(x, y);
        this.ctx.stroke();
      },
      drawEllipse: function (type, x, y, radiusX, radiusY, rotation, startAngle, endAngle, anticlockwise) {
        this.ctx.beginPath();
        this.ctx.ellipse(x, y, radiusX, radiusY, rotation, startAngle, endAngle, anticlockwise);
        this.ctx[type]();
      },
      drawBezierCurve: function (x_, y_, cp1x, cp1y, cp2x, cp2y, x, y) {
        this.ctx.beginPath();
        this.ctx.moveTo(x_, y_);
        this.ctx.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y);
        this.ctx.stroke();
      },
      clearCanvas: function () {
        this.ctx.clearRect(0, 0, this.sizes.width, this.sizes.height);
      },
      clearAll:function () {
        this.clearCanvas();
        this.dots = [];
        this.ellipses = [];
        this.loops = [];
        this.dot = Object.assign({}, this.settings.dot.default_, {});
        this.ellipse = Object.assign({}, this.settings.ellipse.default_, {});
        this.loop = Object.assign({}, this.settings.loop.default_, {});
      },
      setSettings: function () {
        this.ctx.fillStyle = this.settings.dot.color;
        this.ctx.strokeStyle = this.settings.line.color;
        this.ctx.lineWidth = this.settings.line.width;
      },
      setDot: function (e) {
        this.setSettings();

        if (e.which === 3) {
          this.clearCanvas();
          this.drawDefault();
          return
        }

        let length = this.dots.length;
        if (length < this.settings.dot.count) {
          this.dot.x = e.layerX;
          this.dot.y = e.layerY;

          this.drawEllipse('fill', this.dot.x, this.dot.y, this.settings.dot.diameter / 2,
            this.settings.dot.diameter / 2, 0, 2 * Math.PI, false);

          if (length) {
            this.drawLine(this.dots[length - 1].x, this.dots[length - 1].y, this.dot.x, this.dot.y);
          }

          this.dots.push(Object.assign({}, this.dot, {}));

          if (length + 1 === this.settings.dot.count) {
            this.drawLine(this.dot.x, this.dot.y, this.dots[0].x, this.dots[0].y);
            this.dot = Object.assign({}, this.settings.dot.default_, {});
          }
        }
        else {
          this.selectDot(e);
        }
      },
      selectDot: function (e) {
        let smallDiagonal = this.sizes.diagonal;
        let dot = null;
        for (let element of this.dots) {
          let diagonal = Math.sqrt(Math.pow((element.x - e.layerX), 2) + Math.pow((element.y - e.layerY), 2));
          if (diagonal < smallDiagonal) {
            smallDiagonal = diagonal;
            dot = element;
          }
        }

        if (dot) {
          let draw = false;
          if (JSON.stringify(dot) === JSON.stringify(this.dot)) {
            if (this.ellipses.length < this.settings.ellipse.count) {
//              this.drawEllipse('stroke', this.dot.x - this.settings.dot.diameter / 2, this.dot.y - this.settings.dot.diameter / 2,
//                this.settings.dot.diameter, this.settings.dot.diameter, 0, 2 * Math.PI, false);

              this.drawBezierCurve(dot.x, dot.y, (dot.x - e.layerX) / 2, (dot.y + e.layerY) / 2,
                (e.layerX + this.dot.x) / 2, (e.layerY - dot.y) / 2, this.dot.x, this.dot.y);

              this.ellipse.x = this.dot.x;
              this.ellipse.y = this.dot.y;
              this.ellipses.push(Object.assign({}, this.ellipse, {}));
              this.dot = Object.assign({}, this.settings.dot.default_, {});
              draw = true;
              return;
            }
          }
          else {
            if (this.dot && this.dot.x !== undefined && this.dot.x !== null) {
              if (this.loops.length < this.settings.loop.count) {
                this.drawBezierCurve(dot.x, dot.y, (dot.x + e.layerX) / 2, (dot.y + e.layerY) / 2,
                  (this.dot.x + e.layerX) / 2, (dot.y + e.layerY) / 2, this.dot.x, this.dot.y);
                this.loop.from.x = dot.x;
                this.loop.from.y = dot.y;
                this.loop.to.x = this.dot.x;
                this.loop.to.y = this.dot.y;
                this.loops.push(Object.assign({}, this.loop, {}));
                this.dot = Object.assign({}, this.settings.dot.default_, {});
                draw = true;
              }
            }
          }
          if (!draw) {
            this.dot = dot;
          }
          this.drawEllipse('fill', dot.x, dot.y, this.settings.dot.diameter, this.settings.dot.diameter, 0, 2 * Math.PI, false);
        }
      },
      drawDefault: function () {
        let length = this.defaultData.dots.length;
        this.dots = this.defaultData.dots;

        if (this.defaultData.dots[0]){
          this.drawEllipse('stroke', this.defaultData.dots[0].x - this.settings.dot.diameter * 5 / Math.sqrt(2),
            this.defaultData.dots[0].y + this.settings.dot.diameter * 5 / Math.sqrt(2),
            this.settings.dot.diameter * 5, this.settings.dot.diameter * 5, 0, 2 * Math.PI, false);

          this.ellipse.x = this.defaultData.dots[0].x;
          this.ellipse.y = this.defaultData.dots[0].y;
          this.ellipses.push(Object.assign({}, this.ellipse, {}));
        }

        if(this.defaultData.dots[2] && this.defaultData.dots[3]){
          this.drawBezierCurve(this.defaultData.dots[2].x, this.defaultData.dots[2].y,
            (this.defaultData.dots[2].x + 1000) / 2, (this.defaultData.dots[2].y + 300) / 2,
            (this.defaultData.dots[3].x + 500) / 2, (this.defaultData.dots[3].y - 100) / 2,
            this.defaultData.dots[3].x, this.defaultData.dots[3].y);
          this.loop.from.x = this.defaultData.dots[2].x;
          this.loop.from.y = this.defaultData.dots[2].y;
          this.loop.to.x = this.defaultData.dots[3].x;
          this.loop.to.y = this.defaultData.dots[3].y;
          this.loops.push(Object.assign({}, this.loop, {}));
        }

        if(this.defaultData.dots[3] && this.defaultData.dots[4]){
          this.drawBezierCurve(this.defaultData.dots[3].x, this.defaultData.dots[3].y,
            (this.defaultData.dots[3].x + 600) / 2, (this.defaultData.dots[3].y + 0) / 2,
            (this.defaultData.dots[4].x + 50) / 2, (this.defaultData.dots[4].y + 50) / 2,
            this.defaultData.dots[4].x, this.defaultData.dots[4].y);
          this.loop.from.x = this.defaultData.dots[3].x;
          this.loop.from.y = this.defaultData.dots[3].y;
          this.loop.to.x = this.defaultData.dots[4].x;
          this.loop.to.y = this.defaultData.dots[4].y;
          this.loops.push(Object.assign({}, this.loop, {}));
        }

        for (let index in this.defaultData.dots) {
          if (index > 0) {
            this.drawLine(this.defaultData.dots[index - 1].x, this.defaultData.dots[index - 1].y,
              this.defaultData.dots[index].x, this.defaultData.dots[index].y)
          }
          else {
            this.drawLine(this.defaultData.dots[length - 1].x, this.defaultData.dots[length - 1].y,
              this.defaultData.dots[index].x, this.defaultData.dots[index].y)
          }

          this.drawEllipse('fill', this.defaultData.dots[index].x, this.defaultData.dots[index].y,
            this.settings.dot.diameter / 2, this.settings.dot.diameter / 2, 0, 2 * Math.PI, false);
        }
      }
    },
    mounted: function () {
      this.ctx = this.$refs.canvas.getContext("2d");
      this.ctx.font = "20px Georgia";
      this.ctx.fillStyle = this.settings.dot.color;
      this.ctx.strokeStyle = this.settings.line.color;
    }
  }
</script>

<style>
  #app {
    margin: auto;
    height: 100vh;
  }

  body {
    overflow: hidden;
    height: 100vh;
  }

  .head > label {
    height: 60px;
    display: inline-block;
    width: calc(100% / 8 - 4px);
  }

  .head > label > input {
    width: 98%;
    height: 26px;
    border-width: 2px;
    padding: 1px 0;
    cursor: pointer;
  }


  canvas {
    border: 1px solid #BBB;
  }
</style>
