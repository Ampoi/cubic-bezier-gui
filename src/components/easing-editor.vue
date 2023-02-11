<template>
  <div id="mainDiv">
    <div>
      <canvas width="250.5" height="20" ref="demoCanvas"/>
      <div id="demoAnimeBall" :class="{'runDemoAnime':runDemoAnime}"/>
    </div>
    <div id="cubicEditor">
      <div>
        <button v-for="buttonBezier in buttonsBezier" :key="buttonBezier.key" @click="useTemplate(buttonBezier)">
          <svg
            width="50" height="50" viewBox="-10 -10 120 120" xmlns="http://www.w3.org/2000/svg" class="buttonBezier"
          >
            <path :d="`M 0 100 L ${buttonBezier[0]*100} ${(1-buttonBezier[1])*100}`" stroke="#6B6B6B" stroke-width="3"/>
            <path :d="`M 100 0 L ${buttonBezier[2]*100} ${(1-buttonBezier[3])*100}`" stroke="#6B6B6B" stroke-width="4"/>
            <path :d="`
              M 0 100
              C ${buttonBezier[0]*100} ${(1-buttonBezier[1])*100}
              ${buttonBezier[2]*100} ${(1-buttonBezier[3])*100}
              100 0
            `" fill="none" stroke="black" stroke-width="7"/>
            <circle
              :cx="buttonBezier[0]*100" :cy="(1-buttonBezier[1])*100" r="7" stroke="none" fill="#6B6B6B"
            /> <!--始点の操作-->
            <circle
              :cx="buttonBezier[2]*100" :cy="(1-buttonBezier[3])*100" r="7" stroke="none" fill="#6B6B6B"
            />
          </svg>
        </button>
      </div>
      <svg
        :width="inputSize" :height="inputSize" :viewBox="`${(100-svgSize)/2} ${(100-svgSize)/2} ${svgSize} ${svgSize}`" xmlns="http://www.w3.org/2000/svg" ref="mainView"
        @mousedown="startMove"
      >
        <path :d="`M 0 100 L ${cubicBezier[0]*100} ${(1-cubicBezier[1])*100}`" stroke="#C587D1" stroke-width="1.5"/>
        <path :d="`M 100 0 L ${cubicBezier[2]*100} ${(1-cubicBezier[3])*100}`" stroke="#C587D1" stroke-width="1.5"/>
        <path :d="`
          M 0 100
          C ${cubicBezier[0]*100} ${(1-cubicBezier[1])*100}
          ${cubicBezier[2]*100} ${(1-cubicBezier[3])*100}
          100 0
        `" fill="none" stroke="black" stroke-width="2.5"/>
        <circle
          :cx="cubicBezier[0]*100" :cy="(1-cubicBezier[1])*100" r="6" stroke="none" fill="#9C26B0"
        /> <!--始点の操作-->
        <circle
          :cx="cubicBezier[2]*100" :cy="(1-cubicBezier[3])*100" r="6" stroke="none" fill="#9C26B0"
        /> <!--終点の操作-->
      </svg>
    </div>
    <p id="cubicOutput">{{modelValue}}</p>
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
    ],
    runDemoAnime: false
  }},
  mounted(){
    let newCubicBezier = this.modelValue.replace("cubic-bezier(", "").replace(")", "").split(", ")
    let i = 0
    newCubicBezier.forEach((value)=>{
      newCubicBezier[i] = Number(value)
      i++
    })
    this.cubicBezier = newCubicBezier

    addEventListener("mousemove", this.movePoint)
    addEventListener("mouseup", this.endMove)
  },
  methods:{
    movePoint(e){
      if(this.mouseDown){
        const mouseLeft = this.getMousePos(e.clientX, this.$refs.mainView.getBoundingClientRect().left)
        let newPointLeft
        if(mouseLeft < 0){
          newPointLeft = 0
        }else if(1 < mouseLeft){
          newPointLeft = 1
        }else{
          newPointLeft = mouseLeft
        }
        const mouseTop = Math.round((1 - this.getMousePos(e.clientY, this.$refs.mainView.getBoundingClientRect().top)) * 100) / 100
        if(this.aimStartPoint){
          this.cubicBezier[0] = newPointLeft
          this.cubicBezier[1] = mouseTop
        }else{
          this.cubicBezier[2] = newPointLeft
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
      this.resetDemoAnime()
    },

    endMove(){
      this.mouseDown = false
      this.startDemoAnime()
    },

    startDemoAnime(){
      this.runDemoAnime = true
    },

    resetDemoAnime(){
      this.runDemoAnime = false
    },

    useTemplate(buttonBezier){
      this.cubicBezier = buttonBezier.concat()
      this.resetDemoAnime()
      setTimeout(this.startDemoAnime, 30)
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
      }
    }
  }
}
</script>
<style scoped>
#cubicOutput {
  font-size: 12px;
  text-align: center;
}

#mainDiv {
  display: flex;
  flex-direction: column;
  gap: 40px;
  box-shadow: 0 1px 4px 0.5px rgba(0, 0, 0, .2);
  width: 275px;
  padding: 10px;
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

<style scoped>
#demoAnimeBall {
  height: 20px;
  width: 20px;
  margin-top: -20px;
  background: #9C26B0;
  border-radius: 50%;
}

.runDemoAnime {
  transition: all 1.0s v-bind(modelValue);
  margin-left: 230px;
}
</style>