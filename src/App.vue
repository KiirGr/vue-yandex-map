<script setup>
import { ref, toRaw } from 'vue'
</script>
<template>
  <div>
    <span>Адреса:</span>
    <ul>
      <li v-for="(singleAdress, idx) in adressList" v-bind:key="idx">
        {{ singleAdress }}
        <button v-on:click="deletePoint(idx)" type="button">
          Удалить точку - {{ idx }}
        </button>
      </li>
    </ul>
  </div>
  <div>
    Карта:
    <div id="map" style="width: 600px; height: 400px"></div>
  </div>
  <div>
    <label for="adressField" class="block text-sm font-medium text-gray-700"
      >Введите адрес:</label
    >
    <input
      v-model="adressField"
      v-on:keydown.enter="addPoint($event.target.value)"
      type="text"
      name="adressField"
      id="adressField"
      placeholder="Метро Пушкинская"
    />
    <button v-on:click="buttonClick()" type="button">Добавить</button>
  </div>
</template>

<script>
export default {
  name: 'App',

  data() {
    return {
      adressField: '',
      pointToAdd: '',
      pointToDelete: '',
      adressList: [],
      coordinatesArr: [],
      myMap: new Object(),
    }
  },

  created() {
    // Инициализация карты после создания
    window.ymaps.ready(this.initMap)
  },

  methods: {
    // Создание карты
    initMap() {
      // const myMap = ref(null)
      this.myMap = new window.ymaps.Map('map', {
        center: [55.76, 37.64],
        zoom: 13,
      })
      const suggestedAdress = new window.ymaps.SuggestView('adressField')

      return suggestedAdress
    },

    // добавление адреса в список и отправка координат точки
    addPoint(adressInputValue) {
      this.adressField = adressInputValue

      const resultCoordinates = window.ymaps.geocode(this.adressField)

      resultCoordinates
        .then(res => {
          this.pointToAdd = res.geoObjects.get(0).geometry.getCoordinates()
        })
        .catch(err => alert(`Произошла ошибка: ${err}`))
      this.adressList.push(this.adressField)
      this.adressField = ''
    },
    buttonClick() {
      this.addPoint(document.getElementById('adressField').value)
    },

    deletePoint(pointIndex) {
      this.pointToDelete = pointIndex
    },
  },

  watch: {
    // отслеживание получения координат точки и установка последней на карту согласно коорданатам
    pointToAdd(value) {
      this.coordinatesArr.push(value)
      const points = JSON.parse(JSON.stringify(this.coordinatesArr))
      const toRawMap = toRaw(this.myMap)
      toRawMap.geoObjects.removeAll()

      if (points.length === 1) {
        const placemark = new window.ymaps.Placemark(points[0])

        toRawMap.geoObjects.add(placemark)
        toRawMap.setCenter(points[0])
        return
      }
      window.ymaps
        .route(points, {
          multiRoute: false,
        })
        .done(
          route => {
            route.options.set('mapStateAutoApply', true)
            toRawMap.geoObjects.add(route)
          },
          err => {
            // err.message or write your own message
            const errorMsg = `Произошла ошибка: ${err}`

            alert(errorMsg)
          },
          this,
        )
    },

    pointToDelete(value) {
      this.coordinatesArr.splice(value, 1)
      const points = JSON.parse(JSON.stringify(this.coordinatesArr))
      console.log(this.coordinatesArr)
      const toRawMap = toRaw(this.myMap)
      toRawMap.geoObjects.removeAll()

      if (points.length === 1) {
        const placemark = new window.ymaps.Placemark(points[0])

        toRawMap.geoObjects.add(placemark)
        toRawMap.setCenter(points[0])
        return
      }
      window.ymaps
        .route(points, {
          multiRoute: false,
        })
        .done(
          route => {
            route.options.set('mapStateAutoApply', true)
            toRawMap.geoObjects.add(route)
          },
          err => {
            // err.message or write your own message
            const errorMsg = `Произошла ошибка: ${err}`

            alert(errorMsg)
          },
          this,
        )
    },
  },
}
</script>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
