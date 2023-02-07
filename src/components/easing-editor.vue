<template>
  <div id="mainDiv">
    <canvas width="250.5" height="20" ref="demoCanvas"/>
    <div id="cubicEditor">
      <div>
        <button v-for="buttonBezier in buttonsBezier" :key="buttonBezier.key" @click="cubicBezier = buttonBezier.concat()">
          <svg
            width="45" height="45" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" class="buttonBezier"
          >
            <path :d="`M 0 100 L ${buttonBezier[0]*100} ${(1-buttonBezier[1])*100}`" stroke="#6B6B6B" stroke-width="3"/>
            <path :d="`M 100 0 L ${buttonBezier[2]*100} ${(1-buttonBezier[3])*100}`" stroke="#6B6B6B" stroke-width="4"/>
            <path :d="`
              M 0 100
              C ${buttonBezier[0]*100} ${(1-buttonBezier[1])*100}
              ${buttonBezier[2]*100} ${(1-buttonBezier[3])*100}
              100 0
            `" fill="none" stroke="black" stroke-width="4"/>
            <circle
              :cx="buttonBezier[0]*100" :cy="(1-buttonBezier[1])*100" r="4" stroke="none" fill="#6B6B6B"
            /> <!--始点の操作-->
            <circle
              :cx="buttonBezier[2]*100" :cy="(1-buttonBezier[3])*100" r="4" stroke="none" fill="#6B6B6B"
            />
          </svg>
        </button>
      </div>
      <svg
        :width="inputSize" :height="inputSize" :viewBox="`${(100-svgSize)/2} ${(100-svgSize)/2} ${svgSize} ${svgSize}`" xmlns="http://www.w3.org/2000/svg" ref="mainView"
        @mousemove="movePoint"
        @mousedown="startMove"
        @mouseup="mouseDown = false"
        @mouseleave="mouseDown = false"
      >
        <path :d="`M 0 100 L ${cubicBezier[0]*100} ${(1-cubicBezier[1])*100}`" stroke="#A994BE" stroke-width="1"/>
        <path :d="`M 100 0 L ${cubicBezier[2]*100} ${(1-cubicBezier[3])*100}`" stroke="#A994BE" stroke-width="1"/>
        <path :d="`
          M 0 100
          C ${cubicBezier[0]*100} ${(1-cubicBezier[1])*100}
          ${cubicBezier[2]*100} ${(1-cubicBezier[3])*100}
          100 0
        `" fill="none" stroke="black" stroke-width="1"/>
        <circle
          :cx="cubicBezier[0]*100" :cy="(1-cubicBezier[1])*100" r="3" stroke="none" fill="#7B46A2"
        /> <!--始点の操作-->
        <circle
          :cx="cubicBezier[2]*100" :cy="(1-cubicBezier[3])*100" r="3" stroke="none" fill="#7B46A2"
        /> <!--終点の操作-->
      </svg>
    </div>
    <p>
      cubic-bezier(<span
        v-for="(value, index) in cubicBezier"
        :key="index"
        id="cubicInput"
      >
        <input
          type="number"
          v-model="cubicBezier[index]"
        ><span>,</span>
      </span>)
    </p>
  </div>
</template>
<script>
export default {
  props: ["modelValue"],
  emits: ["update:modelValue"],
  data(){return {
    cubicBezier: [0.4, 0.1, 0.4, 0.8],
    inputSize: 165,
    svgSize: 140,
    mouseDown: false,
    aimStartPoint: true,
    buttonsBezier: [
      [0.4, 0, 0.6, 1],
      [0.4, 0, 1, 1],
      [0, 0, 0.6, 1]
    ]
  }},
  mounted(){
    let newCubicBezier = this.modelValue.replace("cubic-bezier(", "").replace(")", "").split(", ")
    let i = 0
    newCubicBezier.forEach((value)=>{
      newCubicBezier[i] = Number(value)
      i++
    })
    this.cubicBezier = newCubicBezier
  },
  methods:{
    movePoint(e){
      if(this.mouseDown){
        const mouseLeft = this.getMousePos(e.clientX, this.$refs.mainView.getBoundingClientRect().left)
        const mouseTop = Math.round((1 - this.getMousePos(e.clientY, this.$refs.mainView.getBoundingClientRect().top)) * 100) / 100
        if(this.aimStartPoint){
          this.cubicBezier[0] = mouseLeft
          this.cubicBezier[1] = mouseTop
        }else{
          this.cubicBezier[2] = mouseLeft
          this.cubicBezier[3] = mouseTop
        }
      }
    },
    getMousePos(mousePos, elementPos){
      return Math.round(((mousePos - elementPos) - (((this.svgSize-100)/2) / this.inputSize * this.svgSize)) / this.inputSize * this.svgSize) / 100
    },
    startMove(e){
      this.mouseDown = true
      const mouseLeft = this.getMousePos(e.clientX, this.$refs.mainView.getBoundingClientRect().left)
      const mouseTop = Math.round((1 - this.getMousePos(e.clientY, this.$refs.mainView.getBoundingClientRect().top)) * 100)/100
      const startPointDistance = Math.sqrt((this.cubicBezier[0] - mouseLeft)**2 + (this.cubicBezier[1]- mouseTop)**2)
      const endPointDistance = Math.sqrt((this.cubicBezier[2] - mouseLeft)**2 + (this.cubicBezier[3]- mouseTop)**2)
      this.aimStartPoint = startPointDistance < endPointDistance
    },
    startDemoAnime(){
      const canvas = this.$refs["demoCanvas"]
      const ctx = canvas.getContext("2d")
      const moveWidth = canvas.width - 30
      ctx.fillStyle = "#7C46A320"
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      let t = 0
      while (t<1){
        const y = this.returnYpos(t)
        ctx.beginPath()
        ctx.arc(10+y*moveWidth, 10, 10, 0, Math.PI*2, false);
        ctx.fill()
        t += 0.05
      }
    },

    returnYpos(x){
      const y2 = this.cubicBezier[1]
      const y3 = this.cubicBezier[3]
      const t = this.getTimeFromX(x)
      return (1-t)**3*0 + 3*(1-t)**2*t*y2 + 3*(1-t)*t**2*y3 + t**3*1
    },

    getTimeFromX(x){
      if(x <= 0){return 0}
      if(x >= 1){return 1}
      let oldT
      let t = x
      while(Math.abs(t - oldT) > 1e-4){
        oldT = t
        t = t - ((this.besierX(t) - x) / this.bezierDX(t))
      }
      return t
    },

    bezierX(t){
      const bx = 3 * this.cubicBezier[0]
      const ax = 1 - bx
      return t * (t * (bx + t * ax));
    },

    bezierDX(t){
      const bx = 3 * this.cubicBezier[0]
      const ax = 1 - bx
      return t * (2 * bx + 3 * ax * t);
    }
  },
  watch: {
    cubicBezier: {
      deep: true,
      handler(){
        let newCubicBezier = "cubic-bezier("
        let i = 0
        this.cubicBezier.forEach((value)=>{
          newCubicBezier += `${value}`
          if(i != 3){newCubicBezier += ","}
          i++
        })
        newCubicBezier += ")"
        this.$emit("update:modelValue", newCubicBezier)
        this.startDemoAnime()
      }
    }
  }
}
</script>
<style scoped>
#cubicInput input::-webkit-inner-spin-button,
#cubicInput input::-webkit-outer-spin-button {
  -webkit-appearance:none !important;
  -moz-appearance:textfield !important;
  margin: 0 !important;
}

#cubicInput input {
  width: 33px;
}

#cubicInput:last-child span {
  display: none;
}

#mainDiv {
  display: flex;
  flex-direction: column;
  gap: 15px;
  box-shadow: 0 10px 25px 0 rgba(0, 0, 0, .5);
  width: 280.5px;
  padding: 15px;
  border-radius: 6px;
}

#cubicEditor {
  display: flex;
  flex-direction: row;
  gap: 10px;
  margin: 0 auto 0 auto;
}

#cubicEditor div {
  gap: 15px;
  display: flex;
  flex-direction: column;
}

.buttonBezier {
  background-color:#f5f5f5;
  padding: 6px;
  margin: 0 0 0 0;
}
</style>