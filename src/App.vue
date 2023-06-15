<script>
import axios from 'axios';
axios.defaults.baseURL = 'http://localhost:3005';
import dayjs from 'dayjs';
import 'dayjs/locale/ru';
import relativeTime from 'dayjs/plugin/relativeTime';

dayjs.extend(relativeTime);
dayjs.locale('ru');



// Код компонента
export default {
	data() {
		return {
			shop: [],
			gold: null,
			plantIndex: null,
			count: 1,
			plants: [],
		};
	},

	mounted() {
		setInterval(() => {
			this.loadShop();
			this.loadGarden();
		}, 2000)
		this.loadUser();

	},

	methods: {
		async loadShop() {
			let responce = await axios.get(`/shop`);

			this.shop = responce.data;
		},

		async harvest(item) {
			let responce = await axios.post('/garden/harvest', {
				id: item._id,
			});

			this.loadUser();

		},	

		async loadUser() {
			let responce = await axios.get('/user');

			this.gold = responce.data.gold;
		},

		async loadGarden() {
			let responce = await axios.get('/garden');
			this.plants = responce.data
		},

		getTimeToHarvest(date) {
			let day = dayjs(date);
			let now = dayjs();
			if(now > day) {
				return 'Готово'
			} else {
				return day.fromNow();

			}
		},	

		getPrice() {
			let plant = this.shop[this.plantIndex];

			let price = 0;

			if(plant && this.count > 0) {
				price = plant.buyPrice * this.count
			}

			return price
		},

		async buy(evt) {
			evt.preventDefault();
			let responce = await axios.post('/shop/buy', {
				title: this.shop[this.plantIndex].title,
				count: this.count
			})

			this.loadGarden();
			this.loadUser();

		},


	},
};
</script>

<template>
	<div class="app">
		<div class="container my-3">

			<!-- Форма покупки растения -->
			<form class="shop" @submit="buy">
				<div class="row top-row">
					<div class="col">
						<!-- Магазин -->
						<div class="shop-plants">
							<!-- Растение -->
							<label class="shop-item" v-for="(item, index) in shop">
								<img :src="'src/assets/'+ item.image + '.png'">
								<p>{{ item.title }}</p>
								<p>${{ item.buyPrice}}/${{ item.sellPrice}}</p>
								<input type="radio" v-model="plantIndex" :value="index">
							</label>
						</div>
					</div>
					<div class="col">
						<div class="shop-form">
							<!-- Количество -->
							<div class="shop-info-row">
								<div class="shop-info-left">
									<label for="count" class="form-label">Количество:</label>
								</div>
								<div class="shop-info-right">
									<input type="number" id="count" class="form-control count-input" v-model="count">
								</div>
							</div>
							<div class="shop-info-row">
								<div class="shop-info-left">
									Итого:
								</div>
								<div class="shop-info-right">
									${{ getPrice() }}
								</div>
							</div>

							<div class="shop-info-row">
								<div class="shop-info-left">
									Ваш баланс:
								</div>
								<div class="shop-info-right">
									${{ gold }}
								</div>
							</div>

							<div class="shop-info-row">
								<div class="shop-info-left">
								</div>
								<div class="shop-info-right">
									<button type="submit" class="btn btn-primary">
										Посадить
									</button>
								</div>
							</div>
						</div>
					</div>
				</div>
			</form>


			<!-- Огород с растениями -->
			<div class="garden">
				<!-- Растение в огороде -->
				<div class="garden-plant" v-for="(item, inedx) in plants">
					<img :src="'src/assets/'+ item.image +'.png'" @click="harvest(item)">
					<p>{{ getTimeToHarvest(item.timeToHarvest) }}</p>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
.shop {
	border: 4px solid black;
	padding: 10px;
}

.top-row {
	padding: 20px;
}

.shop-info-row {
	display: flex;
	align-items: center;
	font-size: 20px;
	gap: 20px;
	height: 40px;
	margin: 20px;
}
.shop-info-left {
	width: 120px;
	text-align: right;
}
.shop-info-right {
	width: 100px;
}
.total {
	font-size: 20px;
	height: 40px;
	display: flex;
	align-items: center;
}

.count-input  {
	font-size: 20px;
}

.btn-primary {
	font-size: 20px;
}

.user-info {
	font-size: 30px;
	padding-left: 40px;
	padding-right: 10px;
	text-align: right;
}

.shop-plants {
	display: flex;
	flex-wrap: wrap;
	justify-content: flex-start;
}

.shop-item {
    width: 150px;
    padding: 20px;
	text-align: center;
}

.shop-item img {
	width: 100%;
}


.garden {
	border: 4px solid black;
	padding: 20px;
	display: flex;
	flex-wrap: wrap;
	background-image: url('src/assets/background.jpg');
	background-size: cover;
	min-height: 300px;
}

.garden-plant {
	width: 150px;
	padding: 20px;
	text-align: center;
	color: white;
}

.garden-plant img {
	width: 100%;
}
</style>
