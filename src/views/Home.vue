<template>
  <div class="home">
    <p>Raw Pixel</p>
    <div v-for="pixel in pixels" class="pixel_row" v-if="show">
      <div
        v-for="p in pixel"
        class="pixel"
        v-bind:style="{ backgroundColor: 'rgb(' + p + ')' }"
      >
        <!--rgb:{{p}}-->
      </div>
    </div>

    <br />

    <div v-for="pixel in quantization" class="pixel_row">
      <div
        class="pixel"
        v-bind:style="{ backgroundColor: 'rgb(' + pixel + ')' }"
      >
        rgb:{{ pixel }}
      </div>
    </div>

    <br />

    <p>Scalar quantization output</p>
    <div v-for="pixel in pixels_afterCodeBook" class="pixel_row" v-if="show">
      <div
        v-for="p in pixel"
        class="pixel"
        v-bind:style="{ backgroundColor: 'rgb(' + p + ')' }"
      >
        <!--rgb:{{p}}-->
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";

export default {
  name: "Home",
  data: function() {
    return {
      pixels: [
        [
          [202, 118, 148],
          [237, 141, 151],
          [250, 172, 164],
          [239, 166, 169]
        ],
        [
          [122, 36, 41],
          [175, 53, 18],
          [228, 43, 118],
          [233, 134, 60]
        ],
        [
          [70, 49, 62],
          [86, 49, 35],
          [150, 39, 43],
          [169, 56, 58]
        ],
        [
          [151, 132, 233],
          [169, 135, 244],
          [179, 135, 244],
          [174, 139, 244]
        ]
      ],
      // quantization: [
      //   [70,70,70],
      //   [120,120,120],
      //   [155,155,155],
      //   [255,255,255],
      // ],
      quantization: [
        [168, 135, 241],
        [242, 159, 161],
        [221, 98, 108],
        [128, 47, 42]
      ],
      middle: [[], [], [], []],

      show: false,

      partition: [63, 127, 191, 255], //
      codebook: [32, 95, 159, 223], //

      pixels_afterCodeBook: [[], [], [], []]
    };
  },
  mounted() {
    this.init();

    this.step1();

    console.log(this.middle);

    this.getTheMiddle(this.middle[0]);
    this.getTheMiddle(this.middle[1]);
    this.getTheMiddle(this.middle[2]);

    this.show = true;
  },
  methods: {
    init() {},
    // 获取向量距离的平方值
    distance(arr1, arr2) {
      // console.log(arr1, arr2);
      let d = 0;
      for (let i = 0; i < arr1.length; i++) {
        d += (arr2[i] - arr1[i]) * (arr2[i] - arr1[i]);
      }
      return d;
    },
    // 获取最短的距离，相对于quantization，并且返回index
    getTheLowest(arr) {
      let index = 0;
      let minL = this.distance(arr, this.quantization[0]);
      for (let i = 1; i < this.quantization.length; i++) {
        let newDis = this.distance(arr, this.quantization[i]);
        // console.log(newDis);
        if (minL > newDis) {
          index = i;
          minL = newDis;
        }
      }
      return index;
    },
    step1() {
      let results = "";
      let codeBook_results = "";

      for (let pixel = 0; pixel < this.pixels.length; pixel++) {
        for (let p = 0; p < this.pixels[pixel].length; p++) {
          let index = this.getTheLowest(this.pixels[pixel][p]);

          console.log(this.pixels[pixel][p]);

          codeBook_results +=
            this.getCodeBookReturn(this.pixels[pixel][p]) + "   ";

          this.pixels_afterCodeBook[pixel].push(
            this.getCodeBookReturn(this.pixels[pixel][p])
          );

          // console.log('the level:', this.quantization[index]);

          //** 打开后颜色变为量化后的
          // this.pixels[pixel][p] = this.quantization[index];

          this.middle[index].push(this.pixels[pixel][p]);
        }
      }

      console.log("codeBook_results:", codeBook_results);
    },
    getTheMiddle(arr) {
      let a = 0;
      let b = 0;
      let c = 0;
      for (let i = 0; i < arr.length; i++) {
        a += arr[i][0];
        b += arr[i][1];
        c += arr[i][2];
      }

      let new_quantization_level_one = [
        a / arr.length,
        b / arr.length,
        c / arr.length
      ];

      console.log(new_quantization_level_one);
    },
    // based on the partition, evenly distributed the number
    getCodeBookReturn(arr) {
      let newArrary = [];

      for (let i = 0; i < arr.length; i++) {
        let index = 0;
        for (let j = 0; j < this.partition.length; j++) {
          if (arr[i] > this.partition[j]) {
            index = j + 1;
          }
        }
        newArrary.push(this.codebook[index]);
      }

      return newArrary;
    }
  }
};
</script>
<style>
div.pixel_row {
  display: flex;
  width: 240px;
  margin: 0 auto;
}
div.pixel {
  width: 60px;
  height: 60px;
  font-size: 10px;
}
</style>
