<template>
	<section class="container headerSign">
		<div class="step3 menu">
			<div class="row">
				<div class="col-12 col-lg-8">
					<div class="row justify-content-center">
						<h2>Select Your Recipes</h2>
					</div>
					<div class="row">
						<template v-for="(recipee, index) in recipes">
							<div class="col-12 col-sm-4" @click="modal(recipee)">
								<div class="col-12 menuCard" data-toggle="modal" :data-target="'#' + recipee.name">
									<img :src="recipee.main_image" :alt="recipee.name">
									<h3>{{recipee.name}}</h3>
								</div>
								<p><i class="fa fa-check check" v-if="(recipee.check == true)"></i><i class="fa fa-times-circle decline" v-if="(recipee.check == true)" @click="unRecipeChecking(recipee, index)"></i><i class="fa fa-plus-circle plus" v-if="!recipee.check && add_recipee == false" @click="recipeChecking(recipee, index)"></i></p>
							</div>

							<div class="modal fade" :id="recipee.name" tabindex="-1" role="dialog" aria-hidden="true">
								<div class="modal-dialog" role="document">
									<div class="modal-content">
										<div class="col-12">
											<button type="button" class="close float-right" data-dismiss="modal" aria-label="Close">
												<span aria-hidden="true">&times;</span>
											</button>
										</div>
										<h2 class="text-center">{{recipee.name}}</h2>
											<img :src="recipee.main_image" :alt="recipee.name">
											<div class="container recipeParam">
												<div class="row">
													<div class="col-4">
														<h3 class="text-center"><i class="fa fa-clock-o"></i></h3>
														<p class="text-center">{{recipee.cooking_time}} Minutes</p>
													</div>
													<div class="col-4">
														<h3 class="text-center" v-if="recipee.difficulty == 'easy'"><i class="fa fa-leaf"></i></h3>
														<h3 class="text-center" v-else-if="recipee.difficulty == 'middle'"><i class="fa fa-leaf"></i><i class="fa fa-leaf"></i></h3>
														<h3 class="text-center" v-else-if="recipee.difficulty == 'hard'"><i class="fa fa-leaf"></i><i class="fa fa-leaf"></i><i class="fa fa-leaf"></i></h3>
														<p class="text-center">{{recipee.difficulty}}</p>
													</div>
													<div class="col-4">
														<h3 class="text-center">{{recipee.calories}}</h3>
														<p class="text-center">Calories</p>
													</div>
												</div>
											</div>
											<div class="col-12 recipeScoop">
												<h3>The Description</h3>
												<p>{{recipee.description}}</p>
											</div>
											<div class="recipeSend">
												<h3>What We Send</h3>
												<div class="row">
													<template v-for="ingredient in ingredients">
														<div class="col-4">
															<p><i class="fa fa-check-square-o"></i>{{ingredient.name}}</p>
														</div>
													</template>
												</div>
											</div>
										</div>
									</div>
								</div>
							</template>
						</div>
					</div>

					<!-- ORDER -->
					<div class="col-12 col-lg-4">
						<div class="orderCard">
							<h4 class="text-center">WHATS IN YOUR BOX</h4>
							<p class="text-center portionTitle">Number of recipes</p>
							<ul class="nav nav-pills mb-3 justify-content-center radioButtons" id="pills-tab">
								<template v-for="(recipee, index) in filterPlan.recipes">
									<li class="nav-item">
										<a class="nav-link" :class="{active: recipee == recipeeOrder}" @click="recipesCheck(recipee, index)">{{recipee}}</a>
									</li>
								</template>
							</ul>
							<p class="text-center portionTitle">Portion per recipe</p>
							<ul class="nav nav-pills mb-3 justify-content-center radioButtons" id="pills-tab2">
								<template v-for="(person, index) in filterPlan.persons">
									<li class="nav-item">
										<a class="nav-link" :class="{active: person == personOrder}" @click="personsCheck(person, index)">{{person}}</a>
									</li>
								</template>
							</ul>

							<p>Per Portion: <span class="float-right">${{statePlan.price}}</span></p>
							<p v-if="statePlan.delivery_fee !== '0.00'">Delivery: <span class="float-right">${{statePlan.delivery_fee}}</span></p>
							<p v-if="statePlan.packaging_fee !== '0.00'" class="bbottom">Packaging Fee: <span class="float-right">${{statePlan.packaging_fee}}</span></p>
							<p class="total">Total: <span class="float-right">$<span id="totalPrice">{{statePlan.total}}</span></span></p>
							<div class="row">
								<div class="col-12">
									<div class="confirm">
										<h5 class="text-center">Select Your Delivery Day</h5>
										<div class="btn-group nav justify-content-center radioButtons" data-toggle="buttons"> 
											<label class="btn active" @click="deliveryTuesday()">
												<input type="radio" name="options" autocomplete="off" checked> Tuesday
											</label>
											<label class="btn" @click="deliveryFriday()">
												<input type="radio" name="options" autocomplete="off"> Friday
											</label>
										</div>
										<p class="text-center"><small>Your order will be delivered on <span id="del_day"></span></small></p>
										<button class="btn imealButton orangeButton" @click="updateNextWeek()" v-if="showNextStep == true">Confirm</button>
									</div>
								</div>
							</div>
						</div>
					</div>

				</div>
			</div>
		</section>
	</template>
	<script>
	export default {
		data() {
			return {
				order: {},
				recipes: [],
				checkRecipee: [{}],
				add_recipee: false,
				showNextStep: false,
				ingredients: [],
				newValue: {
					recipes: '',
					persons: ''
				},
				filterPlan: {
					recipes: [],
					persons: [],
				},
				plansList: [{}],
				statePlan: {
					price: ''
				},
				recipeeIndex: 1,
				personIndex: 1,
				day: {
					tuesday: '',
					friday: ''
				},
				recipeeOrder: '',
				personOrder: ''
			}
		},

		methods: {
			updateNextWeek() {
				this.order.user_id = this.$root.authUser
				let price = this.statePlan.total
				price = parseInt(price)
				this.order.price = price
				this.order.recipee_id = this.checkRecipee
				axios.post('/api/order-detail', this.order)
					.then(response => {
						window.location.href = '/profile'
					})

			},
			modal(recipee) {
				axios.get('/api/recipee/' + recipee.id)
				.then(response => {
					this.ingredients = response.data.ingredients
				})
				.catch(error => {
					console.log(error);
				});
			},
			recipeChecking(recipee, index) {
				let recipeeCheck = this.newValue.recipes
				let declines = document.getElementsByClassName('decline').length
				declines = parseInt(declines)
				recipeeCheck = parseInt(recipeeCheck)
				recipeeCheck = recipeeCheck - 1
				if(declines < recipeeCheck) {
					this.add_recipee = false
					this.checkRecipee[index] = {
						id: recipee.id
					}
					recipee.check = true
					this.showNextStep = false
				} else if (declines == recipeeCheck) {
					this.checkRecipee[index] = {
						id: recipee.id
					}
					recipee.check = true
					this.add_recipee = true
					this.showNextStep = true
				}
			},
			unRecipeChecking(recipee, index) {
				this.add_recipee = false
				this.showNextStep = false
				this.checkRecipee[index] = {
					id: false
				}
				recipee.check = '' 
			},
			recipesCheck(recipee, index) {
				recipee = parseInt(recipee)
				this.newValue.recipes = recipee
				let declines = document.getElementsByClassName('decline').length
				declines = parseInt(declines)
				let result = _.where(this.plansList, {recipes: this.newValue.recipes, person: this.newValue.persons})
				if ((result.length !== 0) && (declines < recipee) ) {
					this.recipeeOrder = recipee
					this.recipeeIndex = index
					this.statePlan = result[0]
					this.order.plan_id = this.statePlan.id
					this.add_recipee = false
					this.showNextStep = false
			} else if ((result.length !== 0) && (declines == recipee)) {
				this.recipeeOrder = recipee
				this.recipeeIndex = index
				this.statePlan = result[0]
				this.order.plan_id = this.statePlan.id
				this.add_recipee = true
				this.showNextStep = true
			} else {
				this.recipeeOrder = this.recipeeOrder
				this.recipeeIndex = this.recipeeIndex
			}
		},
		personsCheck(person, index) {
			person = parseInt(person)
			this.newValue.persons = person
			let result = _.where(this.plansList, {recipes: this.newValue.recipes, person: this.newValue.persons})
			if (result.length !== 0) {
				this.personOrder = person
				this.personIndex = index
				this.statePlan = result[0]
				this.order.plan_id = this.statePlan.id
			} else {
				this.personOrder = this.personOrder
				this.personIndex = this.personIndex
			}
		},
		deliveryFriday() {
			this.order.delivery_day = this.day.friday
			this.order.prefer_day = 'friday'
			document.getElementById('del_day').innerHTML = '' + this.day.friday + ''
			let day = document.getElementById('del_day').innerHTML
			day = this.day.friday
		},
		deliveryTuesday() {
			this.order.delivery_day = this.day.tuesday
			this.order.prefer_day = 'tuesday'
			document.getElementById('del_day').innerHTML = '' + this.day.tuesday + '';
			let day = document.getElementById('del_day').innerHTML
			day = this.day.tuesday
		},
	},

	created() {
		this.checkRecipee[0] = {
			id: false
		}

		axios.get('/api/menu/next')
		.then(response => {
			this.order.delivery_day = response.data.tuesday
			this.day.tuesday = response.data.tuesday
			this.day.friday = response.data.friday
			document.getElementById('del_day').innerHTML = '' + this.day.tuesday + '';
		})

		axios.get('/api/menu/now')
		.then(response => {
			this.recipes = response.data[0].recipes
		})

		axios.get('/api/plan-group')
		.then(response => {
			this.plansGroup = response.data
			this.plansList = response.data[0].plans
			let result = response.data[0].plans
			let recipes = _.keys(_.countBy(result, function(result) { return result.recipes; }))
			let person = _.keys(_.countBy(result, function(result) { return result.person; }))
			this.filterPlan.recipes = recipes
			this.filterPlan.persons = person

			let newPerson = response.data[0].plans[2].person
			newPerson = parseInt(newPerson)
			let newRecipes = response.data[0].plans[2].recipes
			newRecipes = parseInt(newRecipes)
			this.newValue.recipes = newRecipes
			this.newValue.persons = newPerson
			this.personOrder = newPerson
			this.recipeeOrder = newRecipes
			let resultWhere = _.where(this.plansList, {recipes: this.newValue.recipes, person: this.newValue.persons})
			this.statePlan = resultWhere[0]
			this.order.plan_id = this.statePlan.id
		})
		.catch(error => {
			console.log(error);
		});
	}
}
</script>
