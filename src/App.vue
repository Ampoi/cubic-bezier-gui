<template>
  <p>結果=> {{easingA}}</p>
  <easing-editor v-model="easingA"/>
  <!--<img src="/sankou.png" style="width:275px">-->
</template>
<script>
import EasingEditor from "./components/easing-editor.vue"
export default {
  components: {EasingEditor},
  data(){return{
    easingA: 'cubic-bezier(0.4, 0.1, 0.4, 0.8)'
  }}
}
</script>