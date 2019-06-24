<template>
	<div id="app" :class="background">
		<div class="preloader" v-if="load == true">
			<div class="preloader-image"></div>
			<div class="preloader-text">Загрузка</div>
		</div>
		<div class="container" :class="{blured: load || popup.visible}">
			<div class="city">
				<p class="city-name">{{coords.city}}</p>
				<div class="city-choose">
					<a href="#" class="city-choose__search" @click.prevent="popup.visible = true">Сменить город</a>
					<a href="#" class="city-choose__geo" @click.prevent="updateInfo">Мое местоположение</a>
				</div>
			</div>
			<div class="main">
				<div class="main-temp">
					<div class="main-temp__icon">
						<img :src="weather.icon" alt="">
					</div>
					<p class="main-temp__degree">{{weather.temp}}°</p>
				</div>
				<p class="main-description">{{weather.description}}</p>
			</div>
			<div class="additional">
				<div class="additional-info">
					<div class="additional-info__title">Ветер</div>
					<div class="additional-info__data"> <b>{{weather.wind.speed}}</b> м/с, {{windDirection(weather.wind.deg)}}</div>
				</div>
				<div class="additional-info">
					<div class="additional-info__title">Давление</div>
					<div class="additional-info__data"> <b>{{weather.pressure}}</b> мм рт. ст.</div>
				</div>
				<div class="additional-info">
					<div class="additional-info__title">Влажность</div>
					<div class="additional-info__data"> <b>{{weather.humidity}}</b>%</div>
				</div>
				<div class="additional-info">
					<div class="additional-info__title">Облачность</div>
					<div class="additional-info__data"> <b>{{weather.clouds}}</b>%</div>
				</div>
			</div>
		</div>
		<div class="popup" v-show="popup.visible">
			<div class="popup-overlay" @click="popup.visible = false; popup.search = ''; popup.error = ''"></div>
			<div class="popup-window">
				<h2>Выберите город:</h2>
				<p class="popup-window__error" v-show="popup.error">{{popup.error}}</p>
				<input type="text" v-model="popup.search" placeholder="Введите название...">
				<button @click.prevent="citySearch">Искать</button>
			</div>
		</div>
	</div>
</template>

<script>
const API_KEY = 'a0533a229c6b65b7b434a8632cab3f0a';

export default {
	name: "app",
	data: function() {
		return {
			load: true,
			popup: {
				visible: false,
				search: '',
				error: ''
			},
			coords: {
				lat: 0.0,
				lon: 0.0,
				city: 'Moscow'
			},
			weather: {
				temp: 20,
				icon: '01d',
				description: 'Sunny',
				wind: {
					speed: 0,
					deg: 0
				},
				pressure: 0,
				humidity: 0,
				clouds: 0
			},
			background: 'sunny'
		}
	},
	methods: {
		updateInfo: function() {

			if (!navigator.geolocation) {
				alert('Sorry, geoposition not working')
				this.load			= false
				this.popup.visible 	= true
			} else {
				this.load = true

				navigator.geolocation.getCurrentPosition(position => {
					this.coords.lat = position.coords.latitude
					this.coords.lon = position.coords.longitude

					this.$http
					.get('http://api.openweathermap.org/data/2.5/weather?lat=' + this.coords.lat 
						+ '&lon=' + this.coords.lon 
						+ '&APPID=' + API_KEY 
						+ '&lang=ru' 
						+ '&units=metric'
					)
					.then(response => {
						this.setData(response)
						this.load = false
					}, () => {
						alert('Sorry, geoposition not working');
						this.load = false;
						this.popup.visible = true;
					})

				}, () => {
					alert('Sorry, geoposition not working')
					this.load 			= false
					this.popup.visible 	= true
				});
			}
		},
		citySearch: function() {
			this.popup.error = ''
			this.load = true
			this.$http
				.get('http://api.openweathermap.org/data/2.5/weather?q=' + this.popup.search
					+ '&APPID=' + API_KEY 
					+ '&lang=ru' 
					+ '&units=metric'
				)
				.then(response => {
					this.setData(response)
					this.load = false;
					this.popup.visible = false
				}, () => {
					this.load = false
					this.popup.error = 'Не нашли такого города, попробуйте снова.'
				})
		},
		setBackground: function(id) {
			
			if ( (id >= 200 && id < 800) || (id > 800 && id <= 804)) {
				this.background = 'rainy'
			} else  {
				this.background = 'sunny'
			}
		},
		setData: function(response) {
			this.coords.city 			= response.data.name
			this.weather.temp 			= Math.trunc(response.data.main.temp)
			this.weather.icon 			= 'http://openweathermap.org/img/w/' + response.data.weather[0].icon + '.png'
			this.weather.description 	= response.data.weather[0].description
			this.weather.wind.speed 	= response.data.wind.speed
			this.weather.wind.deg 		= response.data.wind.deg
			this.weather.pressure 		= Math.trunc(response.data.main.pressure * 0.750063755419211)
			this.weather.humidity 		= response.data.main.humidity
			this.weather.clouds			= response.data.clouds.all

			if (response.data.weather[0].icon.slice(-1) === 'n') { // icon id contains time of day info
				this.background = 'night'
			} else {
				this.setBackground(response.data.weather[0].id)
			}
		},
		windDirection: function (deg) {
			const array = [0, 22.5, 45, 67.5, 90, 112.5, 135, 157.5, 180, 202.5, 225, 247.5, 270, 292.5, 315, 337.5, 360]
			deg = array.sort((x, y) => Math.abs(deg - x) - Math.abs(deg - y))[0]

			switch(deg) {
				case 0 		: return 'Северный'
				case 22.5 	: return 'Северо-северо-восточный'
				case 45 	: return 'Северо-восточный'
				case 67.5 	: return 'Восточно-северо-восточный'
				case 90 	: return 'Восточный'
				case 112.5 	: return 'Восточно-юго-восточный'
				case 135 	: return 'Юго-восточный'
				case 157.5 	: return 'Юго-юго-восточный'
				case 180 	: return 'Южный'
				case 202.5 	: return 'Юго-юго-западный'
				case 225 	: return 'Юго-западный'
				case 247.5	: return 'Западно-юго-западный'
				case 270 	: return 'Западный'
				case 292.5 	: return 'Западно-северо-западный'
				case 315 	: return 'Северо-западный'
				case 337.5 	: return 'Северо-северо-западный'
				case 360	: return 'Северный'
				default		: return ''
			}
		}
	},
	mounted: function() {
		this.updateInfo()
	}
};
</script>

<style lang="scss">

$b-clear: #038aef;
$b-rain: #678dbc;
$b-night: #292929;

* {
	margin: 0;
	padding: 0;
	border: 0;
	box-sizing: border-box;
}

.preloader {
	display: flex;
	flex-direction: column;
    justify-content: center;
    align-items: center;
	position: fixed;
	top: 0;
	left: 0;
	z-index: 999;
	width: 100%;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.57);

	&-image {
		display: inline-block;
		width: 80px;
		height: 80px;
		vertical-align: text-bottom;
		border: 7px solid rgb(255, 255, 255);
		border-right-color: transparent;
		border-radius: 50%;
		animation: spin 1s linear infinite;
	}

	&-text {
		font-size: 20px;
		color: white;
		margin-top: 1em;
	}
}

.popup {
	position: fixed;
	top: 0;
	left: 0;
	z-index: 990;
	width: 100%;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.57);
	display: flex;

	&-overlay {
		position: absolute;
		z-index: 991;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		cursor: pointer;
	}

	&-window {
		position: relative;
		z-index: 992;
		width: 100%;
		max-width: 400px;
		margin: auto;
		padding: 15px;
		background-color: white;
		color: #000;

		&__error {
			color: rgb(161, 0, 0);
			margin-top: 16px
		}

		input[type=text] {
			width: 100%;
			height: 2.4em;
			padding: 5px;
			border: 1px solid #dadada;
			border-radius: 3px;
			margin-top: 16px;
		}

		button {
			padding: 10px 20px;
			border: 1px solid #dadada;
			background-color: #2196F3;
			color: white;
			margin-top: 16px;
			border-radius: 3px;
			cursor: pointer;
			transition: background-color .5s ease;

			&:hover {
				background-color: rgb(23, 100, 163);
			}
		}
	}
}

#app {
	font-family: Helvetica, Arial, sans-serif;
	color: white;
	width: 100vw;
	height: 100vh;
	overflow-y: auto;
	
	&.sunny {
		background-color: $b-clear;
	}

	&.rainy {
		background-color: $b-rain;
	}

	&.night {
		background-color: $b-night;
	}
}

.container {
	display: flex;
	flex-direction: column;
	justify-content: space-between;
	max-width: 1000px;
	width: 100%;
	margin: auto;
	padding: 3em 15px;
	height: 100%;

	&.blured {
		filter: blur(10px);
	}
}

.city {
	align-self: flex-start;

	&-name {
		font-size: 40px;
		margin-bottom: 10px;
	}

	&-choose {
		a {
			font-size: 12px;
			color: rgba(255, 255, 255, 0.5);
			text-decoration: none;
			margin-right: 15px;
			cursor: pointer;
		}
	}
}

.main {
	align-self: center;

	&-temp {
		&__icon {
			display: inline-block;
		}

		&__degree {
			display: inline-block;
			font-size: 100px;
		}
	}

	&-description {
		text-transform: capitalize;
		font-size: 16px;
		color: rgba(255, 255, 255, 0.8);
	}
}

.additional {
	display: flex;
	flex-direction: row;
	justify-content: space-between;
	flex-wrap: wrap;

	&-info {

		@media (max-width: 768px) {
			width: 50%;
			margin-bottom: 16px;
		}

		@media (max-width: 500px) {
			width: 100%;
			margin-bottom: 16px;
		}

		&__title {
			font-size: 12px;
			color: rgba(255, 255, 255, 0.5);
			margin-bottom: 1em;
		}
	}
}

@keyframes spin {
	to { transform: rotate(360deg); }
}
</style>
