<script>
export default{
  name: "LanguageSelector",
  data(){
    return {
      width: 32,
      height: 24,

    }
  },
  methods:{
    languageSelected(e, lang){
        if(this.$store.state.selectedLanguage != lang){
          if(lang === 'de'){
            alert("Wir erwarten, dass unsere deutsche Anwender über eine ausreichende Ausbildung verfügen, um die Überschriften auf Englisch zu verstehen")
            lang = 'gb'
          }
          this.$store.state.selectedLanguage = lang
          this.$store.commit('changeLanguage', lang)
        }
    }
  }
}
</script>
<template>
<div class="language-container">
  <div class="language-item"  @click="languageSelected(e, 'gb')">
    <img src="https://flagcdn.com/40x30/gb.png"  :width="width" :height="height" :class= "{'language-item--selected': 'gb'===$store.state.selectedLanguage}">
  </div>
  <div class="language-item"  @click="languageSelected(e, 'de')">
    <img src="https://flagcdn.com/40x30/de.png"  :width="width" :height="height" :class= "{'language-item--selected': 'de'===$store.state.selectedLanguage}">
  </div>
  <div class="language-item" @click="languageSelected(e, 'ru')">
    <img src="https://flagcdn.com/40x30/ru.png"  :width="width" :height="height"  :class= "{'language-item--selected': 'ru'===$store.state.selectedLanguage}">
  </div>

</div>
</template>
<style scoped>
.language-container{
    display: flex;
  flex-wrap: wrap;
}
.language-item{
  width: 10%;
    margin: 5px;
  border: 1px black solid;
  border-radius: 5px;
  padding: 2px;
  padding-top: 5px;
  background-color: beige;
}
.language-item:hover{
  cursor: pointer;
}
.language-item--selected {
  background-color: pink;
  margin: 1px;
  padding: 5px;
}
</style>