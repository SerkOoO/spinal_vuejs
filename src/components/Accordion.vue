<script setup>

import axios from 'axios'
import { onMounted, ref } from 'vue';

// Déclaration des variables
let apiData = ref(null)
let roomOccupationStatus = ref({});
let floorOccupationPercentage = ref({});

// Fonction qui permet de récupérer les données de l'API
const fetchData = async () => {
  try {
    // Appel API
    const response = await axios.get('https://api-developers.spinalcom.com/api/v1/geographicContext/space');
    apiData.value = response.data;

    // Parcours des données pour calculer le pourcentage d'occupation
    apiData.value.children.forEach(building => {
      building.children.forEach(floor => {

        let totalRooms = floor.children.length;
        let occupiedRooms = 0;

        floor.children.forEach(room => {
          // On vérifie si la salle est occupée et on met à jour le compteur
          roomOccuper(room.dynamicId).then(occupied => {
            if (occupied) {
              occupiedRooms++;
            }

            // Ici on calcule le pourcentage d'occupation pour chaque étage
            if (floor.children.indexOf(room) === floor.children.length - 1) {
              floorOccupationPercentage.value[floor.dynamicId] = (occupiedRooms / totalRooms) * 100;
            }

          });         
        });
      });
    });

  } catch (error) {

    console.error('Erreur lors de la requête API', error);
  }
};


// Fonction qui permet de vérifier si une salle est occupée.
const roomOccuper = async (room) => {

  const apiUrl = `https://api-developers.spinalcom.com/api/v1/room/${room}/control_endpoint_list`;
  const response = await axios.get(apiUrl);
  const data = response.data;

  // On vérifie si la salle est occupée et met à jour le statut.
  if (data && data[0] && data[0].endpoints) {
    const occupationEndpoint = data[0].endpoints.find(endpoint => endpoint.name === 'Occupation');

    if (occupationEndpoint) {
      // Retourne si la salle est true ou false
      roomOccupationStatus.value[room] = occupationEndpoint.currentValue == 1 ? 'TRUE' : 'FALSE';
      return occupationEndpoint.currentValue == 1;
    }
  }


};

// Utilisation du hook onMounted pour appeler fetchData lors du montage du composant.
onMounted(() => {
  fetchData();  
});


</script>

<template>
 
    <div v-if="apiData" class="accordion" id="accordionExample">
      <div class="accordion-item" v-for="(building, buildingIndex) in apiData.children" :key="buildingIndex">
        <h2 class="accordion-header" :id="'headingBuilding' + buildingIndex">
          <button class="accordion-button" type="button" data-bs-toggle="collapse" :data-bs-target="'#collapseBuilding' + buildingIndex" :aria-expanded="buildingIndex === 0" :aria-controls="'collapseBuilding' + buildingIndex">
            {{ building.name }}
          </button>
        </h2>
        <div :id="'collapseBuilding' + buildingIndex" class="accordion-collapse collapse" :class="{ 'show': buildingIndex === 0 }" :aria-labelledby="'headingBuilding' + buildingIndex" data-bs-parent="#accordionExample">
          <div class="accordion-body">
            <div class="accordion" :id="'accordionFloors' + buildingIndex">
              <div class="accordion-item" v-for="(floor, floorIndex) in building.children" :key="floorIndex">
                <h2 class="accordion-header" :id="'headingFloor' + buildingIndex + floorIndex">
                    <button class="accordion-button" type="button" data-bs-toggle="collapse" :data-bs-target="'#collapseFloor' + buildingIndex + floorIndex" :aria-expanded="floorIndex === 0" :aria-controls="'collapseFloor' + buildingIndex + floorIndex">
                    {{ floor.name }} - Occupation: {{ floorOccupationPercentage[floor.dynamicId].toFixed(2) || '0' }}%
                    </button>

                </h2>
                <div :id="'collapseFloor' + buildingIndex + floorIndex" class="accordion-collapse collapse border-0" :aria-labelledby="'headingFloor' + buildingIndex + floorIndex" data-bs-parent="'#accordionFloors' + buildingIndex">
                <div class="accordion-body">
                  <div class="">
                    <ul class="room-list">

                    
                    <li v-for="(room, roomIndex) in floor.children" :key="roomIndex">
                    Salle: {{ room.name }}
                    - {{ roomOccupationStatus[room.dynamicId] || 'UNDEFINED' }} <!-- Si la salle est ni true ni false, elle sera undefined -->
                    <br v-if="roomIndex !== floor.children.length - 1">
                    </li>
                  </ul>
                </div>
                  
                </div>
              </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  
  



</template>

<style scoped>

*{

/* border-color: black; */
background-color: #343a40;

}

.accordion {
max-width: 70%; 
margin: auto; 
color: orange;

margin-top: 40px;

line-height: 3;

}
.accordion-collapse {
transition: height 0.3s ease-out;

}

.accordion-body{
background-color: #343a40;
border: none;
}

.accordion-button:not(.collapsed) {
background-color: #212529 !important;
color: #FFAC49 !important;
/* border-color: #212529 !important; */
border: none !important;
}

.accordion-button:not(.collapsed):hover {
background-color: #212529 !important;
color: #FFAC49 !important;
border: none !important;
}


.accordion-button {
max-width: 100%;
background-color: #212529 !important;
color: #FFAC49 ;
box-shadow: 0 0 0 0.25rem #343a40;
}

.accordion-button:focus {
box-shadow: 0 0 0 0.25rem #343a40;
}


.room-list {
list-style: none;
padding: 0;
}

.room-list li {
border: 1px solid rgba(0, 0, 0, 0.166); 
padding: 8px;
margin-bottom: 5px;
background-color: #ffe5b50a;
border-radius: 5px;
}

</style>