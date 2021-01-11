<template>
	<div class="menu">
		<button @click="showdifficulty">更改难度</button>
		<button @click="setgrid">再来一局</button>
	</div>
	<div @contextmenu.prevent style="width: 100%;">
		<div v-for="(row,rowi) in grid" class="grid">
			<div v-for="(col,coli) in row" @click="onclick(rowi,coli)" @contextmenu.prevent="changestate(rowi,coli)" @dblclick="clickaround(rowi,coli)"
			 :class="{state0:col.state!=3,state3:col.state==3}">
				<div v-if="col.state==3&&col.arroundnum!=0">{{col.arroundnum}}</div>
				<div v-if="col.ismine==1&&this.showMine">💣</div>
				<div>{{col.state==1?'🚩':col.state==2?'?':''}}</div>
			</div>
		</div>
	</div>
	<div>
		⏱：{{second}}&emsp;&emsp;🚩:{{flagnum}}
	</div>
	<div class="difficulty" v-if="show">
		<difficulty @difficulty="changeDifficulty" />
	</div>
</template>

<script>
	import difficulty from './components/difficulty.vue'
	export default {
		name: 'App',
		components: {
			difficulty
		},
		data: function() {
			return {
				timer: '',
				rownum: 9,
				colnum: 9,
				minenum: 10,
				show: false
			}
		},
		created() {
			this.setgrid()
		},
		methods: {
			showdifficulty() {
				this.show = true
			},
			changeDifficulty(data) {
				this.rownum = data.rownum
				this.colnum = data.colnum
				this.minenum = data.minenum
				this.show = false
				this.setgrid()
			},
			setgrid() {
				let grid = new Array(this.rownum)
				for (var i = 0; i < this.rownum; i++) {
					grid[i] = []
				}
				for (var i = 0; i < this.rownum; i++) {
					for (var j = 0; j < this.colnum; j++) {
						grid[i][j] = {
							ismine: 0,
							state: 0, //0默认 1红旗 2问号 3翻开
						}
					}
				}
				this.grid = grid
				this.isfirst = true
				this.showMine = false
				this.end = false
				clearInterval(this.timer)
				this.second = 0,
					this.flagnum = this.minenum
				this.$forceUpdate()
			},
			setmine(rowi, coli) {
				let coordinates = []
				while (coordinates.length < this.minenum) {
					let coordinate = {
						rowi: Math.floor(Math.random() * this.rownum),
						coli: Math.floor(Math.random() * this.colnum)
					}
					let notaround = coordinate.rowi<rowi-1||coordinate.rowi>rowi+1||coordinate.coli>coli+1||coordinate.coli<coli-1
					let nothave = JSON.stringify(coordinates).indexOf(JSON.stringify(coordinate)) == -1
					if (notaround && nothave) {
						console.log(coordinate);
						coordinates = coordinates.concat(coordinate)
						this.grid[coordinate.rowi][coordinate.coli].ismine = 1
					}
				}
				this.$forceUpdate()
				// console.log(coordinates)
				for (var i = 0; i < this.rownum; i++) {
					for (var j = 0; j < this.colnum; j++) {
						this.grid[i][j].arroundnum = this.getaround(i, j)
					}
				}
				console.log(this.grid)
			},
			getaround(rowi, coli) {
				let mineNum = 0
				let rowmin = rowi == 0 ? 0 : rowi - 1
				let rowmax = rowi == this.rownum - 1 ? this.rownum - 1 : rowi + 1
				let colmin = coli == 0 ? 0 : coli - 1
				let colmax = coli == this.colnum - 1 ? this.colnum - 1 : coli + 1

				for (var i = rowmin; i <= rowmax; i++) {
					for (var j = colmin; j <= colmax; j++) {
						if (this.grid[i][j].ismine == 1) {
							mineNum++
						}
					}
				}
				return mineNum
			},
			onclick(rowi, coli) {
				if (this.end || this.show || this.grid[rowi][coli].state == 1) {
					return
				}
				if (this.isfirst) {
					this.setmine(rowi, coli)
					this.timer = setInterval(() => {
						this.second++
						this.$forceUpdate()
					}, 1000);
					this.isfirst = false
				}
				if (this.grid[rowi][coli].ismine == 1) {
					this.showMine = true
					this.end = true
					clearInterval(this.timer)
					alert("boom")
				} else {
					this.grid[rowi][coli].state = 3
					if (this.getopennum() == this.colnum * this.rownum - this.minenum) {
						this.end = true
						clearInterval(this.timer)
						alert("赢了")
					}
					if (this.grid[rowi][coli].arroundnum == 0) {
						this.autoclick0(rowi, coli)
					}

				}
				this.$forceUpdate()

			},
			autoclick0(rowi, coli) {
				let rowmin = rowi == 0 ? 0 : rowi - 1
				let rowmax = rowi == this.rownum - 1 ? this.rownum - 1 : rowi + 1
				let colmin = coli == 0 ? 0 : coli - 1
				let colmax = coli == this.colnum - 1 ? this.colnum - 1 : coli + 1

				for (var i = rowmin; i <= rowmax; i++) {
					for (var j = colmin; j <= colmax; j++) {
						if (this.grid[i][j].state == 0 || this.grid[i][j].state == 2) {
							this.onclick(i, j)
						}
					}
				}
			},
			changestate(rowi, coli) {
				if (this.end||this.show) {
					return
				}
				let state = this.grid[rowi][coli].state
				if (state == 0) {
					this.flagnum--
					this.grid[rowi][coli].state = 1
				} else if (state == 1) {
					this.flagnum++
					this.grid[rowi][coli].state = 2
				} else if (state == 2) {
					this.grid[rowi][coli].state = 0
				}
				this.$forceUpdate()
			},
			clickaround(rowi, coli) {
				let flagnum = 0
				let rowmin = rowi == 0 ? 0 : rowi - 1
				let rowmax = rowi == this.rownum - 1 ? this.rownum - 1 : rowi + 1
				let colmin = coli == 0 ? 0 : coli - 1
				let colmax = coli == this.colnum - 1 ? this.colnum - 1 : coli + 1

				for (var i = rowmin; i <= rowmax; i++) {
					for (var j = colmin; j <= colmax; j++) {
						if (this.grid[i][j].state == 1) {
							flagnum++
						}
					}
				}
				if (flagnum == this.grid[rowi][coli].arroundnum) {
					for (var i = rowmin; i <= rowmax; i++) {
						for (var j = colmin; j <= colmax; j++) {
							if (this.grid[i][j].state == 0 || this.grid[i][j].state == 2) {
								this.onclick(i, j)
							}
						}
					}
				}
			},
			getopennum() {
				let opennum = 0
				for (var i = 0; i < this.rownum; i++) {
					for (var j = 0; j < this.colnum; j++) {
						if (this.grid[i][j].state == 3) {
							opennum++
						}
					}
				}
				return opennum
			}
		}
	}
</script>

<style>
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
		margin-top: 60px;
		user-select: none;
	}

	.grid {
		display: flex;
		justify-content: center;
	}

	.grid div {
		width: 2rem;
		height: 2rem;
		line-height: 2rem;
		margin: 1px;
	}

	.state0 {
		background-color: #42B983;
		position: relative;
	}
	.state0 div{
		position: absolute;
	}
	.state3 {
		background-color: beige;
	}

	.difficulty {
		background-color: beige;
		position: absolute;
		left: calc(50% - 8rem);
		top: 10rem;
	}

	.menu button {
		margin: 1rem;
	}
</style>
