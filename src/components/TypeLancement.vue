<script setup>
import { defineProps, onMounted, ref } from 'vue';
// Definition des props passée dans App.vue
let additional_data = ref(null)
let isActive = ref(false)
const props = defineProps({
    object : {
        type : Object,
    }
})
// Une fois que le componant est monté dans le DOM, Fetching de données pour eviter asynchronisme
onMounted(async ()=>{
    try{
        // recuperation du 
        let payloads_id = props.object.payloads
        for (let item of payloads_id){
            let fetching = await fetch(`https://api.spacexdata.com/v4/payloads/${item}`);
            let data = await fetching.json()
            additional_data.value = await {...additional_data.value, [[data.name]] : data}
        }
    //Si erreur sur le fetching
    }catch(err){
        console.log(err,"PPP")
    }
    
})
//<a :href="'https://www.youtube.com/watch?v='+object.links.youtube_id" target="_blank">Click to see the video</a>
</script>
<template>
<div class="lancement_box">
    <h3>Nom de la mission: {{ object.name}}</h3>
    <p>Date lancement: {{ object.date_utc.slice(0,10) }}</p>
    <p>{{ object.details ? object.details : "" }}</p>
    <img :src="object.links.patch.large" style="width :20vw">
    <a :href="object.links.article" target="_blank">Click to read the article</a>
    <div :class="isActive ?'display_video' : 'hide_video' " @click="isActive=!isActive">Click me to see the video
        <video controls>
            <source :src="'https://www.youtube.com/embed/'+object.links.youtube_id">
        </video>
    </div>
    <h3>Payloads:</h3>
    <div v-for="item in additional_data" :key="item.id">
        <h4>- {{ item.name }}</h4>
        <p>Client: {{ item.customers.join("") }}</p>
    </div>
    <p></p>
    <p></p>
</div>
</template>

<style>
.display_video{
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
}
.display_video *{
    display: flex;
    width: 60%;
}
.hide_video *{
    display: none;
}
.lancement_box{
    display: flex;
    flex-direction: column;
    align-items: center;
    border: 1px solid;
    width: 60%;
    text-align: center;
}
a:first-of-type{
    margin: 1rem;
}
</style>