<template>
  <div id="app">

    <div class="head">
      <label>
        Speed animation
        <input :disabled="showSpeedAnimation" min="0" minlength="1" type="number" step="1" v-model="options.time"/>
      </label>
      <label>
        Length
        <input type="number" min="0" minlength="1" step="1" v-model="options.length"/>
      </label>
      <label>
        Evasion
        <input type="number" min="0" minlength="1" step="0.1" v-model="evasion.speed"/>
      </label>
      <label>
        Pursuer
        <input type="number" min="0" minlength="1" step="0.1" v-model="pursuer.speed"/>
      </label>
      <label>
        Type of run
        <select v-model="typeRun">
          <option v-for="type in typesRun" :value="type.value">{{type.label}}</option>
        </select>
      </label>
      <label>
        Type of move
        <select v-model="typeMove">
          <option v-for="type in typesMove" :value="type">{{type}}</option>
        </select>
      </label>
      <label v-if="typeMove === 'coefficient'">
        Coefficient
        <input type="number" min="0" max="1" minlength="1" step="0.05" v-model="coefficient"/>
      </label>
    </div>
    <canvas :width="sizes.width" :height="sizes.height" ref="canvas"
            @mousedown="setDot"></canvas>
    <!--v-insert-message="typeRun"></canvas>-->


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
        }
      }
    },
    data() {
      return {
        showSpeedAnimation: false,
        options: {
          radius: 5,
          time: 50,
          e: 0.5,
          length: 0
        },
        evasion: {
          x: '',
          y: '',
          prevX: '',
          prevY: '',
          speed: 2
        },
        pursuer: {
          x: '',
          y: '',
          speed: 3.1
        },
        ctx: null,
        typeRun: 'horizontal',
        typesRun: [
          {
            label: 'horizontal',
            value: 'horizontal'
          },
          {
            label: 'vertical',
            value: 'vertical'
          },
          {
            label: 'random',
            value: 2 * Math.PI
          },
          {
            label: 'half-plane',
            value: Math.PI
          },
          {
            label: 'PI/2',
            value: Math.PI / 2
          },
          {
            label: 'PI/4',
            value: Math.PI / 4
          },
          {
            label: 'PI/8',
            value: Math.PI / 8
          },
          {
            label: 'PI/24',
            value: Math.PI / 24
          },
          {
            label: 'line',
            value: 0
          }
        ],
        typeMove: 'doggy style',
        typesMove: [
          'doggy style',
          'smart',
          'coefficient'
        ],
        coefficient: 0.4
      }
    },
    methods: {
      setDot: function (e) {

        if (this.evasion.x === '' || this.evasion.y === '') {
          this.evasion.x = e.layerX;
          this.evasion.y = e.layerY;
          this.pursuer.x = '';
          this.pursuer.y = '';
          this.ctx.fillStyle = "black";
          this.ctx.beginPath();
          this.ctx.ellipse(this.evasion.x, this.evasion.y, this.options.radius, this.options.radius, 0, 2 * Math.PI, false);
          this.ctx.fill();
        }
        else if (this.pursuer.x === '' || this.pursuer.y === '') {
          this.showSpeedAnimation = true;

          this.pursuer.x = e.layerX;
          this.pursuer.y = e.layerY;
          this.ctx.fillStyle = "orange";
          this.ctx.beginPath();
          this.ctx.ellipse(this.pursuer.x, this.pursuer.y, this.options.radius, this.options.radius, 0, 2 * Math.PI, false);
          this.ctx.fill();
          if (this.evasion.speed && this.pursuer.speed) {
            let firstLine = setInterval(
              function () {
                this.ctx.strokeStyle = "black";
                this.ctx.beginPath();
                this.ctx.moveTo(this.evasion.x, this.evasion.y);


                if (this.typeRun === 'horizontal') {
                  this.evasion.prevX = this.evasion.x;
                  this.evasion.prevY = this.evasion.y;

                  this.evasion.x += parseFloat(this.evasion.speed);
                } else if (this.typeRun === 'vertical') {
                  this.evasion.prevX = this.evasion.x;
                  this.evasion.prevY = this.evasion.y;

                  this.evasion.y += parseFloat(this.evasion.speed);
                } else {
                  let alpha = Math.random() * (this.typeRun / 2 + this.typeRun / 2) - this.typeRun / 2;

                  if (this.evasion.prevX === '' || this.evasion.prevY === '') {
                    this.evasion.prevX = this.pursuer.x;
                    this.evasion.prevY = this.pursuer.y;
                  }

                  let vectors = [
                    {
                      x: this.evasion.x - this.evasion.prevX,
                      y: this.evasion.y - this.evasion.prevY
                    }
                  ];

                  let phi = 0;
                  if (vectors[0].x === 0) {
                    phi = (vectors[0].y > 0) ? Math.PI / 2 : -Math.PI / 2;
                  }
                  else {
                    phi = Math.atan(vectors[0].y / vectors[0].x);
                    if (vectors[0].x < 0) {
                      phi = phi + Math.PI;
                    }
                  }

                  this.evasion.prevX = this.evasion.x;
                  this.evasion.prevY = this.evasion.y;

                  this.evasion.x += Math.cos(alpha + phi) * this.evasion.speed;
                  this.evasion.y += Math.sin(alpha + phi) * this.evasion.speed;

                }

                this.ctx.lineTo(this.evasion.x, this.evasion.y);
                this.ctx.stroke();


                let vector = {
                  x: this.evasion.prevX - this.pursuer.x,
                  y: this.evasion.prevY - this.pursuer.y
                };

                this.ctx.strokeStyle = "orange";
                this.ctx.beginPath();
                this.ctx.moveTo(this.pursuer.x, this.pursuer.y);

                if (this.typeMove === 'doggy style') {

                  let length = Math.sqrt(vector.x * vector.x + vector.y * vector.y),
                    k = length / parseFloat(this.pursuer.speed);

                  this.pursuer.x += vector.x / k;
                  this.pursuer.y += vector.y / k;

                } else if (this.typeMove === 'smart') {

                  let vectorNew = {
                    x: this.evasion.x - this.evasion.prevX,
                    y: this.evasion.y - this.evasion.prevY
                  };
                  let dots = [
                    {
                      x: this.evasion.x,
                      y: this.evasion.y
                    },
                    {
                      x: this.pursuer.x + vectorNew.x,
                      y: this.pursuer.y + vectorNew.y
                    }
                  ];


                  let l = (dots[1].y - dots[0].y) / (dots[1].x - dots[0].x),
                    a = 1 + l * l,
                    b = -2 * this.pursuer.x + 2 * l * (-l * dots[0].x + dots[0].y) - 2 * this.pursuer.y * l,
                    c = this.pursuer.x * this.pursuer.x +
                      Math.pow(dots[0].y - l * dots[0].x, 2) -
                      2 * this.pursuer.y * (dots[0].y - l * dots[0].x) +
                      this.pursuer.y * this.pursuer.y - this.pursuer.speed * this.pursuer.speed,
                    d = b * b - 4 * a * c,
                    dotsSolution = [
                      {
                        x: (-b + Math.sqrt(d)) / (2 * a),
                        y: 0
                      },
                      {
                        x: (-b - Math.sqrt(d)) / (2 * a),
                        y: 0
                      },
                    ];
                  dotsSolution[0].y = (dots[1].y - dots[0].y) * (dotsSolution[0].x - dots[0].x)
                    / (dots[1].x - dots[0].x) + dots[0].y;
                  dotsSolution[1].y = (dots[1].y - dots[0].y) * (dotsSolution[1].x - dots[0].x)
                    / (dots[1].x - dots[0].x) + dots[0].y;


                  let lengths = [
                    Math.sqrt((dotsSolution[0].x - this.evasion.x) * (dotsSolution[0].x - this.evasion.x) +
                      (dotsSolution[0].y - this.evasion.y) * (dotsSolution[0].y - this.evasion.y)),
                    Math.sqrt((dotsSolution[1].x - this.evasion.x) * (dotsSolution[1].x - this.evasion.x) +
                      (dotsSolution[1].y - this.evasion.y) * (dotsSolution[1].y - this.evasion.y))
                  ];

                  if (lengths[0] < lengths[1]) {
                    this.pursuer.x = dotsSolution[0].x;
                    this.pursuer.y = dotsSolution[0].y;
                  }
                  else {
                    this.pursuer.x = dotsSolution[1].x;
                    this.pursuer.y = dotsSolution[1].y;
                  }
                }
                else {
                  let vectorNew = {
                    x: this.evasion.x - this.evasion.prevX,
                    y: this.evasion.y - this.evasion.prevY
                  };
                  let dots = [
                    {
                      x: this.evasion.x,
                      y: this.evasion.y
                    },
                    {
                      x: this.pursuer.x + vectorNew.x,
                      y: this.pursuer.y + vectorNew.y
                    }
                  ];


                  let l = (dots[1].y - dots[0].y) / (dots[1].x - dots[0].x),
                    a = 1 + l * l,
                    b = -2 * this.pursuer.x + 2 * l * (-l * dots[0].x + dots[0].y) - 2 * this.pursuer.y * l,
                    c = this.pursuer.x * this.pursuer.x +
                      Math.pow(dots[0].y - l * dots[0].x, 2) -
                      2 * this.pursuer.y * (dots[0].y - l * dots[0].x) +
                      this.pursuer.y * this.pursuer.y - this.pursuer.speed * this.pursuer.speed,
                    d = b * b - 4 * a * c,
                    dotsSolution = [
                      {
                        x: (-b + Math.sqrt(d)) / (2 * a),
                        y: 0
                      },
                      {
                        x: (-b - Math.sqrt(d)) / (2 * a),
                        y: 0
                      },
                    ];
                  dotsSolution[0].y = (dots[1].y - dots[0].y) * (dotsSolution[0].x - dots[0].x)
                    / (dots[1].x - dots[0].x) + dots[0].y;
                  dotsSolution[1].y = (dots[1].y - dots[0].y) * (dotsSolution[1].x - dots[0].x)
                    / (dots[1].x - dots[0].x) + dots[0].y;


                  let lengths = [
                    Math.sqrt((dotsSolution[0].x - this.evasion.x) * (dotsSolution[0].x - this.evasion.x) +
                      (dotsSolution[0].y - this.evasion.y) * (dotsSolution[0].y - this.evasion.y)),
                    Math.sqrt((dotsSolution[1].x - this.evasion.x) * (dotsSolution[1].x - this.evasion.x) +
                      (dotsSolution[1].y - this.evasion.y) * (dotsSolution[1].y - this.evasion.y))
                  ];

                  let length = Math.sqrt(vector.x * vector.x + vector.y * vector.y),
                    k = length / parseFloat(this.pursuer.speed),
                    vectors = [
                      {
                        x: vector.x / k,
                        y: vector.y / k,
                      },
                      {
                        x: 0,
                        y: 0
                      }
                    ];


                  if (lengths[0] < lengths[1]) {
                    vectors[1].x = dotsSolution[0].x - this.pursuer.x;
                    vectors[1].y = dotsSolution[0].y - this.pursuer.y;
                  }
                  else {
                    vectors[1].x = dotsSolution[1].x - this.pursuer.x;
                    vectors[1].y = dotsSolution[1].y - this.pursuer.y;
                  }

                  let phi = 0;
                  if (vectors[0].x === 0) {
                    phi = (vectors[0].y > 0) ? Math.PI / 2 : -Math.PI / 2;
                  }
                  else {
                    phi = Math.atan(vectors[0].y / vectors[0].x);
                    if (vectors[0].x < 0) {
                      phi = phi + Math.PI;
                    }
                  }

                  let cosAlpha = (vectors[0].x * vectors[1].x + vectors[0].y * vectors[1].y) /
                    Math.sqrt(vectors[0].x * vectors[0].x + vectors[0].y * vectors[0].y) /
                    Math.sqrt(vectors[1].x * vectors[1].x + vectors[1].y * vectors[1].y);
                  let alpha = Math.acos(cosAlpha);
                  let newAlpha = this.coefficient * alpha;


                  this.pursuer.x += Math.cos(newAlpha + phi) * this.pursuer.speed;
                  this.pursuer.y += Math.sin(newAlpha + phi) * this.pursuer.speed;
                }

                this.ctx.lineTo(this.pursuer.x, this.pursuer.y);
                this.ctx.stroke();

                if (Math.abs(this.pursuer.x - this.evasion.x) < this.options.e &&
                  Math.abs(this.pursuer.y - this.evasion.y) < this.options.e) {
                  clearInterval(firstLine);
                  this.showSpeedAnimation = false;
                  alert('Його спіймали!');
                }
                if (this.evasion.x > this.sizes.width + this.options.length || this.evasion.x < -this.options.length ||
                  this.evasion.y > this.sizes.height + this.options.length || this.evasion.y < -this.options.length) {
                  clearInterval(firstLine);
                  this.showSpeedAnimation = false;
                  if (this.evasion.x === '') {
                    alert("Їх вкрали інопланетяни!");
                  }
                  else {
                    alert("Він втік!");
                  }
                }

//              }.bind(this), this.evasion.speed
              }.bind(this), 1000 / this.options.time
            );
          }
        }
        else {
          this.showSpeedAnimation = false;
          this.pursuer.x = '';
          this.pursuer.y = '';
          this.evasion.x = '';
          this.evasion.y = '';
          this.evasion.prevX = '';
          this.evasion.prevY = '';
          this.ctx.clearRect(0, 0, this.sizes.width, this.sizes.height);
        }
      }

    },
    mounted: function () {
      this.ctx = this.$refs.canvas.getContext("2d");
      this.ctx.fillStyle = "black";
      this.ctx.strokeStyle = "black";
      this.ctx.font = "20px Georgia";
    },
//    directives: {
//      insertMessage: function (canvasElement, binding) {
//        console.log('insertMessage')
//      }
//    }
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
    width: calc(100% / 7 - 4px);
  }

  .head > label > * {
    width: 98%;
    height: 26px;
  }

  .head > label > input {
    height: 26px;
  }

  .head > label > select {
    height: 32px;
  }

  canvas {
    border: 1px solid #BBB;
  }
</style>
