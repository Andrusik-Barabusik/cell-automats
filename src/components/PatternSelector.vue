<script>
  export default{
    name: 'PatternSelector',
    data(){
      return {
        selectedPatternIndex: -1
      }
    },
    methods:{
      clickPattern(e, index){
        if(this.selectedPatternIndex === index){
          this.selectedPatternIndex = -1
        }else{
          this.selectedPatternIndex = index
        }
        this.$emit('selectPatternIndex', this.selectedPatternIndex)
      },
      getPatternName(name){
        return this.$store.state.messages[name]
      }
    }
  }
</script>
<template>
<div class="pattern-selector">
  <div><slot/></div>
  <div class="pattern-container">
    <div v-for="(pattern, index) in $store.state.lifePatterns" class="pattern-item" :class= "{'pattern-item--selected': index===selectedPatternIndex}" @click="clickPattern(evt, index)" ref="pattern-{{index}}">
      {{pattern.size}}<br>{{getPatternName(pattern.name)}}
    </div>
  </div>
</div>
</template>
<style scoped>
.pattern-selector{
  border: 1px black solid;
  border-radius: 15px;
  padding-top: 10px;
}
.pattern-container {
  display: flex;
  flex-wrap: wrap;
}
.pattern-item {
  width: 20%;
  margin: 5px;
  border: 1px black solid;
  border-radius: 5px;
  padding: 4px;
  background-color: beige;
}

.pattern-item--selected {
  background-color: pink;
  margin: 1px;
  padding: 8px;
}
</style>