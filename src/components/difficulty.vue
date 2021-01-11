<template>
	<ul>
		<li @click="changeDifficulty(9,9,10)">初级（9x9 10个雷）</li>
		<li @click="changeDifficulty(16,16,40)">中级（16x16 40个雷）</li>
		<li @click="changeDifficulty(16,30,99)">高级（16x30 99个雷）</li>
		<li @click="showbySelf">自定义</li>
	</ul>
	<ul v-if="show">
		<li><input v-model="rownum" placeholder="只能输入数字" type="text"> 行</li>
		<li><input v-model="colnum" placeholder="只能输入数字" type="text"> 列</li>
		<li><input v-model="minenum" placeholder="只能输入数字" type="text"> 雷</li>
		<button @click="submit">确定</button>
	</ul>
</template>

<script>
	export default {
		name: 'difficulty',
		props: {
			// msg: String
		},
		data: function() {
			return {
				rownum:'',
				colnum:'',
				minenum:'',
				show:false
			}
		},
		methods: {
			changeDifficulty(row,col,mine) {
				this.$emit("difficulty",{
					rownum:row,
					colnum:col,
					minenum:mine
				})
			},
			showbySelf(){
				this.show=!this.show
			},
			submit(){
				this.changeDifficulty(this.rownum,this.colnum,this.minenum)
			}
		},
		watch: {
			rownum: function(val) {
				this.rownum = val.replace(/\D/g, '')
			},
			colnum: function(val) {
				this.colnum = val.replace(/\D/g, '')
			},
			minenum: function(val) {
				this.minenum = val.replace(/\D/g, '')
			}
		}

	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
	ul {
		list-style-type: none;
		padding: 0;
		width: 16rem;
	}

	li {
		/* display: inline-block; */
		background: #42B983;
		line-height: 1.6rem;
		margin: 0.625rem;
		border-radius: 0.3125rem;
	}

	a {
		color: #42b983;
	}
</style>
