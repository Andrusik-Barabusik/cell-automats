<script>
  export default{
    name: 'AnimalPalette',
    props: {
      numberOfColors: Number,
      minColors: Number,
      maxColors: Number,
      showPlusMinusButtons: {type: Boolean, default: true} 
},
    data() {
      return {
        colorExampleSize: 25,
        colors: 0,
        minusButtonDisabled: false,
        plusButtonDisabled: false
      }
    },
    methods: {
      drawAnimals(){
        const canv=this.$refs.animalSpeciesPalette
        const ctx = canv.getContext("2d");
        ctx.font = "20px Arial";
        for(var i=0;i<this.numberOfColors;i++){
          ctx.fillStyle=this.$store.state.colors[i]
          ctx.fillRect(i*this.colorExampleSize, 0, (i+1)* this.colorExampleSize, this.colorExampleSize)
          ctx.fillStyle="black"
          if(this.$store.state.colors[i] === 'black'){
            ctx.fillStyle="white"   
          }
          ctx.fillText(i+1, i*this.colorExampleSize + 5, this.colorExampleSize - 3)
        }
      },
      checkColors(mode){
        this.colors = this.numberOfColors
        if(mode === '-'){
          this.colors --
          this.plusButtonDisabled = false
          if(this.colors === this.minColors){
            this.minusButtonDisabled = true
          }
        }else{
          this.colors ++
          this.minusButtonDisabled = false
          if(this.colors === this.maxColors){
            this.plusButtonDisabled = true
          }
        }
        this.$emit('changeNumberOfColors', this.colors)
        return this.colors
      }
    },
    updated() {
      this.drawAnimals()
    },
    mounted(){
      this.drawAnimals()
    }
  }
</script>
<template>
  <div>
  <slot/><br>
    <template v-if="showPlusMinusButtons">
      <button @click=" checkColors('-')" :disabled="minusButtonDisabled">-</button>
    </template>
    <canvas ref="animalSpeciesPalette" :width="numberOfColors * colorExampleSize" :height="colorExampleSize" />
    <template v-if="showPlusMinusButtons">
      <button @click=" checkColors('+')" :disabled="plusButtonDisabled">+</button>
    </template>
    &nbsp;&nbsp;
  </div>
  
</template>
<style scoped>
canvas {
  border: 1px solid ;
  margin-bottom:-9px;
}
div {
  margin-bottom: 10px;
}
</style>