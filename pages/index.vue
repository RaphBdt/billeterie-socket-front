<template>
  <div>
    <b-container class="my-5">
      <b-row align-h="end">
        <b-col lg="9">
          <p class="mx-1">Nombre de connecté<span v-if="numberOfUsersConnected > 1">s</span> : {{ numberOfUsersConnected }}</p>
        </b-col>
      </b-row>
      <b-row>
        <b-col lg="3">
          <img class="w-100" src="@/assets/images/logo.svg.png" />
        </b-col>
        <b-col lg="9" class="d-flex flex-wrap">
          <div v-for="place in places.places" :key="place.id">
            <div v-if="!place.reserved" @click="addANewPlaceToCart(place.id)" class="m-1 place d-flex justify-content-center align-items-center" :id="'place-' + place.id">{{ place.id }}</div>
            <div v-else-if="place.user == idOfTheUser" class="m-1 place d-flex justify-content-center align-items-center place-reserved-by-user">{{ place.id }}</div>
            <div v-else class="m-1 place d-flex justify-content-center align-items-center reserved">{{ place.id }}</div>
          </div>
        </b-col>
      </b-row>
      <b-row align-h="end">
        <b-col lg="9">
          <a @click="orderNewPlaces" class="btn btn-dark w-100 mx-1 my-4">Acheter</a>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import {io} from "socket.io-client";

export default {
  name: 'IndexPage',
  data() {
    return {
      selectedPlaces: [],
      numberOfUsersConnected: 0,
      places: [],
      idOfTheUser: 0,
      maximumPlacesThatUserCanAddToCart: 3
    }
  },
  methods: {
    addANewPlaceToCart(place) {
      if(this.maximumPlacesThatUserCanAddToCart >= 1) {
        if(this.selectedPlaces.length < this.maximumPlacesThatUserCanAddToCart && !this.selectedPlaces.includes(place)) {
          this.selectedPlaces.push(place);
          document.getElementById(`place-${place}`).classList.add("selected-place");
        } else if (!this.selectedPlaces.includes(place)) {
          let placeToRemove = this.selectedPlaces[0];
          document.getElementById(`place-${placeToRemove}`).classList.remove("selected-place");
          this.selectedPlaces.shift();
  
          this.selectedPlaces.push(place);
          document.getElementById(`place-${place}`).classList.add("selected-place");
        }
      }
    },
    orderNewPlaces() {
      fetch('http://localhost:3003/buy', {
        headers: {'Content-Type': 'application/json'},
        method: 'POST',
        body: JSON.stringify({
          place: this.selectedPlaces,
          user: this.idOfTheUser
        })
      })
      this.selectedPlaces = [];

    }
  },
  beforeMount() {
    const socket = io("http://localhost:3003", {});

		socket.on('users', users => {
      this.numberOfUsersConnected = users;
		});

    socket.on('allThePlaces', allThePlaces => {
      this.places = allThePlaces;

      allThePlaces.places.forEach(element => {
        if(element.user == this.idOfTheUser) {
          this.maximumPlacesThatUserCanAddToCart--;
        }
      });
		});

    if(localStorage.getItem('user')) {
      this.idOfTheUser = localStorage.getItem('user');
      console.log("Connexion réussie, votre identifiant est : " + this.idOfTheUser);
    } else {
      this.idOfTheUser = Date.now();
      localStorage.setItem("user", this.idOfTheUser);
      console.log("Inscription réussie, votre identifiant est : " + localStorage.getItem('user'));
    }
  },
}
</script>

<style>
  .place {
    width: 60px;
    height: 60px;
    border: 1px solid #eeeeee;
    border-radius: 3px;
  }

  .place:hover {
    cursor: pointer;
  }

  .selected-place {
    background-color: rgb(255, 222, 0);
  }

  .reserved {
    cursor: none;
    background-color: #aeaeae;
  }

  .place-reserved-by-user {
    cursor: none;
    background-color: #43cd1d;
  }
</style>