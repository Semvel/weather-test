<template>
    <div class="wrapper">
        <header>
            <input class="search-input" type="text" placeholder="Введите город" v-model="inputCity" v-on:keyup.enter="cityRequest(this.inputCity)">
            <button class="search-button" type="button" @click="cityRequest(this.inputCity)"><img src="@/assets/images/search-icon.svg"></button>
        </header>

        <main class="weather" v-if="this.weatherActive">
            <div class="weather__selected">
                <div class="left-column">
                    <p class="weather__selected-date">{{this.weatherActive.fullDay}}</p>
                    <h2 class="weather__selected-city">{{ this.city }}</h2>
                    <div class="weather__selected-temp">
                        <div class="weather__selected-picture">
                            <img src="@/assets/images/test-temp.svg" alt="">
                        </div>
                        <span class="weather__selected-day-temp">{{ this.weatherActive.tempDay }}°C</span>
                        <div class="weather__selected-night-temp">
                            <p>ночью</p>
                            <span>{{ this.weatherActive.tempNight }}°C</span>
                        </div>
                    </div>
                </div>
                <div class="right-column">
                    <div class="weather__selected-data">
                        <h3>Ощущается как</h3>
                        <span>{{ this.weatherActive.feelsLike }}°C</span>
                    </div>
                    <div class="weather__selected-data">
                        <h3>Давление</h3>
                        <span>{{ this.weatherActive.pressureMM }}мм рт. ст.</span>
                    </div>
                    <div class="weather__selected-data">
                        <h3>Влажность воздуха</h3>
                        <span>{{ this.weatherActive.humidity }}%</span>
                    </div>
                    <div class="weather__selected-data">
                        <h3>Скорость ветра</h3>
                        <span>{{ this.weatherActive.wind_speed }} м/с</span>
                    </div>
                </div>
            </div>
            <div class="weather__future">
                <h2>Прогноз погоды по дням</h2>
                <div class="weather__items">
                    <div class="weather__item" v-for="(item,index) in weatherArr" :key="item.dt">
                        <h3>{{ dayWeekArr[item.dayWeek] }}</h3>
                        <div class="weather__item-block" v-bind:class="{ active: item.dt === this.weatherActive.dt }" @click="selectDay(index)">
                            <div class="weather__item-picture">
                                <img src="@/assets/images/test-temp.svg" alt="">
                            </div>
                            <div class="weather__item-temp">
                                <p class="weather__item-day">{{item.tempDay}}°C</p>
                                <p class="weather__item-night">{{item.tempNight}}°C</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </main>
        <main v-else-if="city === 0">
            Город не найден
        </main>
    </div>

</template>

<script>
import axios from 'axios'

export default {
    name: "App",
    components: {
    },

    data() {
        return {
            inputCity: '',
            city: '',
            weatherArr: null,
            weatherActive: null,
            dayWeekArr: [
                'Воскресенье',
                'Понедельник',
                'Вторник',
                'Среда',
                'Четверг',
                'Пятница',
                'Суббота'
            ],
            month: [
                'Января',
                'Февраля',
                'Марта',
                'Апреля',
                'Мая',
                'Июня',
                'Июля',
                'Августа',
                'Сентября',
                'Ноября',
                'Декабря',
            ]
        }
    },
    created() {
    },
    updated() {
    },
    methods: {
		cityRequest(city) {
            axios.get(`http://api.openweathermap.org/geo/1.0/direct?q=${city}}&appid=b8d40d2cdfad62a779aceeddae3a3963`)
            .then(responce => {
                console.log(responce.data);
                if(responce.data.length > 0 && responce.data[0].local_names) {
                    this.city = responce.data[0].local_names.ru
                    weatherRequest(responce.data[0]);
                } else {
                    this.weatherArr = null;
                    this.weatherActive = null;
                    this.city = 0;
                }

            })
            .catch(e => {
                console.log(e);
            })

            const weatherRequest = (dataCity) => {
                axios.get(`http://api.openweathermap.org/data/2.5/onecall?lat=${dataCity.lat}&lon=${dataCity.lon}&exclude=current,minutely,hourly,alerts&units=metric&lang=ru&appid=b8d40d2cdfad62a779aceeddae3a3963`)
                .then(responce => {
                    console.log(responce.data);
                    const weatherArr = responce.data.daily.slice(0,5)
                    
                    let newWArr = weatherArr.map(e => {
                        const date = new Date(e.dt * 1000);
                        const day = date.getDate();
                        const month = this.month[date.getMonth()];
                        const Year = date.getFullYear();
                        const dayWeek = date.getDay();
                        const fullDay = `${day} ${month}, ${Year}`
                        const tempDay = Math.round(e.temp.day);
                        const tempNight = Math.round(e.temp.night);
                        const feelsLike = Math.round(e.feels_like.day);
                        const pressureMM = Math.round(e.pressure / 1.3333);
                        let newElem = {...e, fullDay, dayWeek,tempDay,tempNight,feelsLike,pressureMM}
                        return newElem
                    });
                    console.log(newWArr);

                    this.weatherArr = newWArr;
                    this.weatherActive = newWArr[0];
                })
                .catch(e => {
                    console.log(e);
                })
            }
		},
        selectDay(index) {
            let newWeatherActive = this.weatherArr[index]
            this.weatherActive = newWeatherActive;
        }
	}
};

</script>


<style>
    @import './assets/base.css';
    body {
        background: linear-gradient(180deg, #6892FF 0%, rgba(255, 255, 255, 0) 100%), #0A2260;
        color: #000;
        font-family: 'Montserrat', sans-serif;
        font-weight: 400;
        line-height: 1.2;
        font-size: 18px;
        letter-spacing: 0.025em;
    }
    .wrapper {
        max-width: 1220px;
        margin: 0 auto;
        padding: 2rem;
        padding-bottom: 0;
    }

    header {
        position: relative;
        margin-bottom: 100px;
    }
    header .search-input {
        background: rgba(255, 255, 255, 0.5);
        backdrop-filter: blur(19px);
        border-radius: 60px;
        width: 100%;
        height: 40px;
        padding: 10px 50px;
    }
    header .search-button {
        position: absolute;
        background: rgba(255, 255, 255, 0.5);
        border-radius: 60px;
        right: 0;
        height: 40px;
        width: 100px;
    }
    header .search-button img {
        width: 27px;
        height: 27px;
    }

    .weather__selected {
        display: flex;
        justify-content: space-between;
        max-width: 830px;
        margin: 0 auto 70px;
    }

    .weather__selected .left-column {
        display: flex;
        flex-direction: column;
        align-items: center;
        max-width: 350px;
    }
    .weather__selected-city {
        margin-top: 5px;
        margin-bottom: 20px;
        font-size: 35px;
        font-weight: 700;
        color: #fff;
    }
    .weather__selected-date {
        color: #fff;
    }
    .weather__selected-temp {
        display: flex;
        align-items: center;
        justify-content: center;
        background: #FFFFFF;
        box-shadow: 0px 5px 10px rgba(13, 37, 98, 0.3);
        width: 210px;
        height: 210px;
        border-radius: 50%;
    }
    .weather__selected-day-temp {
        font-weight: 300;
        font-size: 70px;
    }

    .weather__selected-night-temp {
        display: flex;
        flex-direction: column;
        align-items: center;
        position: absolute;
        bottom: 15px;
    }  

    .weather__selected-night-temp p {
        color: rgba(0, 0, 0, 0.5);
    }

    .weather__selected-night-temp span {
        font-weight: 300;
        font-size: 20px;
        color: rgba(0, 0, 0, 0.5);
    }

    .weather__selected-picture {
        position: absolute;
        width: 70px;
        height: 56px;
        top: 15px;
        left: calc(50% - 35px);
    }

    .weather__selected-picture img {
        object-fit: contain;
    }
    
    .weather__selected .right-column * {
        color: #fff;
    }

    .weather__selected .right-column {
        align-self: end;
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 60px 130px;
        max-width: 400px;
        text-align: center;
    }
    .weather__selected-data {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
    }
    .weather__selected .right-column h3 {
        font-weight: 700;
        font-size: 16px;
        margin-bottom: 5px;
    }

    .weather__future {
        background: rgba(255, 255, 255, 0.5);
        backdrop-filter: blur(19px);
        border-radius: 60px 60px 0px 0px;
        padding: 40px 90px;
    }

    .weather__future h2 {
        font-weight: 700;
        font-size: 20px;
    }

    .weather__items {
        display: flex;
        justify-content: space-between;
        margin: 0 -20px 20px;
    }

    .weather__item {
        margin: 20px;
        text-align: center;
    }

    .weather__item-block {
        margin-top: 20px;
        background: #FFFFFF;
        box-shadow: 0px 5px 10px rgba(13, 37, 98, 0.3);
        border-radius: 60px;
        padding: 22px;
        width: 165px;
        display: flex;
        flex-direction: column;
        align-items: center;
        cursor: pointer;
        transition: all .3s ease;
    }
    .weather__item-block:hover {
        box-shadow: inset 0px 5px 10px rgba(13, 37, 98, 0.3);
    }
    .weather__item-picture {
        width: 82px;
        height: 82px;
        margin-bottom: 20px;
    }
    .weather__item-picture img {
        object-fit: contain;
        width: 100%;
    }
    .weather__item-temp {
        display: flex;
        justify-content: space-between;
        align-items: center;
        align-self: stretch;
        margin-bottom: 3px;
    }

    .weather__item-day {
        font-weight: 300;
        font-size: 20px;
    }
    .weather__item-night {
        color: rgba(0, 0, 0, 0.5);
    }

    .active.weather__item-block {
        background: #7D9BE6;
        box-shadow: inset 0px 5px 9px rgba(13, 37, 98, 0.3);
    }

    .active .weather__item-day {
        color: #fff;
    }
    .active .weather__item-night {
        color: rgba(255, 255, 255, 0.5);

    }
    
</style>