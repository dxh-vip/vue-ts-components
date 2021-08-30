<template>
  <div class="sword" :class="{ isFixed: isFixed }">
    <div class="con">
      <span class="item" :style="colorStyle"></span>
      <span class="item" :style="colorStyle"></span>
      <span class="item" :style="colorStyle"></span>
    </div>
    <div v-if="showOpa" class="opa"></div>
  </div>
</template>

<script>
export default {
  props: {
    isFixed: {
      // 是否fixed定位
      type: Boolean,
      default: false
    },
    color: {
      // loading的颜色
      type: String,
      default: "#efeffa"
    },
    showOpa: {
      // 是否展示蒙层
      type: Boolean,
      default: true
    }
  },
  data() {
    return {};
  },
  watch: {},
  computed: {
    colorStyle: function() {
      return {
        "--color": `${this.color}`
      };
    }
  },
  mounted() {},
  methods: {}
};
</script>

<style scoped>
.sword {
  width: 100%;
  height: 100%;
  position: absolute;
  z-index: 9999;
  left: 0;
  top: 0;
}
.isFixed.sword {
  position: fixed;
}
.opa {
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.2);
}
.con {
  position: absolute;
  width: 64px;
  height: 64px;
  border-radius: 50%;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.con .item {
  position: absolute;
  box-sizing: border-box;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  box-shadow: 0 0 60px 1px var(--color);
}
.con :first-child {
  left: 0%;
  top: 0%;
  border-bottom: 3px solid var(--color);
  animation: sword-one 1s linear infinite;
}
.con :nth-child(2) {
  right: 0%;
  top: 0%;
  animation: sword-two 1s linear infinite;
  border-right: 3px solid var(--color);
}

.con :last-child {
  right: 0%;
  bottom: 0%;
  animation: sword-three 1s linear infinite;
  border-top: 3px solid var(--color);
}
@keyframes sword-one {
  0% {
    transform: rotateX(35deg) rotateY(-45deg) rotateZ(0deg);
  }
  100% {
    transform: rotateX(35deg) rotateY(-45deg) rotateZ(360deg);
  }
}

@keyframes sword-two {
  0% {
    transform: rotateX(50deg) rotateY(10deg) rotateZ(0deg);
  }
  100% {
    transform: rotateX(50deg) rotateY(10deg) rotateZ(360deg);
  }
}

@keyframes sword-three {
  0% {
    transform: rotateX(35deg) rotateY(55deg) rotateZ(0deg);
  }
  100% {
    transform: rotateX(35deg) rotateY(55deg) rotateZ(360deg);
  }
}
</style>
