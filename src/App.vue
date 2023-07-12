<script setup>
import { onMounted, ref, watch } from 'vue';
import TypeLancement from './components/TypeLancement.vue'
//State
let info_prochain_lancement = ref(null)
let decompte = ref(0)
let selected_option = ref(null)
let selected_data = ref([])
// Fonction pour recuperer les données de l'api pour prochain lancement
async function Data_Next_Launch() {
  try{
    let retriving_data = await fetch("https://api.spacexdata.com/v5/launches/latest")
    let data = await retriving_data.json()
    // Formatage de la date
    let formatted_date = data.date_utc.slice(0,10)
    // recuperation des données de SpaceX Api et stockage dans ref info_prochain_lancement
    info_prochain_lancement.value = {prochain_lancement: data, date : formatted_date}
    // Creation du decompte
    let time_to_launch = data.date_unix - Date.now()
    // Date de lancement valide
    if(time_to_launch > 0){
      decompte.value = time_to_launch
      let timer = setInterval(()=>{
        time_to_launch--
        decompte.value = time_to_launch
        if(time_to_launch === 0 ) clearInterval(timer)
      },1000)
    }
    // La date est deja passé 
    else{
      decompte.value = "Pas de lancement annoncé"
    }
  }// Si erreur
  catch(error){
    console.log(error)
  }
}
onMounted(Data_Next_Launch)
//Fetching pour selection de lancement
watch(selected_option, async()=>{
  try{
    //acces au données
    let fetching = await fetch('https://api.spacexdata.com/v5/launches')
    let données = await fetching.json()
    // Etude de cas selon la valeur de la valeur selectionnez puis ajout des donnée recupere et filtrer à selected_data
    switch( selected_option.value){
      case "1" : 
        selected_data.value = await données.filter((obj,index)=> index<10)
        break;
      case "2":
        selected_data.value = await données.filter((obj,index)=> obj.success===true).slice(0,10)
        break;
      case "3":
        selected_data.value = await données.filter((obj,index)=> obj.success===false).slice(0,10)
  }
  //Si erreur
  }catch(err){
    console.log(err)
  }
  
})

</script>


<template>
<div class="main">
  <h1>SpaceX API</h1>
  <section id="Prochain_Lancement">
    <h2>Nom de la mission: {{ info_prochain_lancement && info_prochain_lancement.prochain_lancement.name }}</h2>
    <div id="Time">
      <p>Date du prochain lancement : {{ info_prochain_lancement && info_prochain_lancement.date_launch }}</p>
      <p id="decompte">{{ decompte }}</p>
    </div>
  </section>
  <section id="Selection_Lancement">
    <label>Selectionnez une option </label>
    <select v-model="selected_option" id="launch_options">
      <option value="1">Tous les lancements </option>
      <option value="2">Lancements réussis</option>
      <option value="3">Lancement échoués</option>
    </select>
  <div class="container">
    <TypeLancement v-for="obj in selected_data" :object="obj"/>
  </div>
  </section>
</div>
</template>

<style>
h1{
  display: flex;
  justify-content: center;
}
#Prochain_Lancement{
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 1rem;
}
#Selection_Lancement{
  margin: 1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.container{
  margin-top: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;

}

</style>
