<template>
	<view class="content">
		<canvas @click="user($event)" style="width: 720rpx; height: 720rpx;" canvas-id="map" id="mapcanvas"></canvas>
		<u-button shape="square" size="mini" type="primary" plain="" style="width:100rpx" @click="reset">重置</u-button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				chessBoard: [],
				myWin: [],
				computerWin: [],
				over: false, //游戏是否结束,
				count: 0,
				me: true,
				win: [],
				ctx: {},
				ceilWidth: 0,
				chessBoardmap: new Uint8ClampedArray([])
			}
		},
		onReady(e) {
			this.ctx = uni.createCanvasContext('map')
			let that = this;
			let width = 0;
			uni.getSystemInfo({
				success(res) {
					width = res.windowWidth - 15;
					that.ctx.drawImage('/static/chessbg.jpg', 0, 0, width, width)
					that.ctx.draw();
					//	棋盘
					that.ceilWidth = width / 15;
					that.ctx.strokeStyle = "#636766"
					for (var i = 0; i < 16; i++) {
						that.ctx.moveTo(i * that.ceilWidth, 0);
						that.ctx.lineTo(i * that.ceilWidth, width);
						that.ctx.stroke();
						that.ctx.moveTo(0, i * that.ceilWidth);
						that.ctx.lineTo(width, i * that.ceilWidth);
						that.ctx.stroke();
					}
					that.ctx.draw(true);
				}
			})

			//棋盘二位数组初始化，使得一个位置只能下一个（种）棋子
			for (var i = 0; i < 15; i++) {
				this.$set(this.chessBoard, i, []);
				for (var j = 0; j < 15; j++) {
					this.$set(this.chessBoard[i], j, 0);
				}
			}
			this.winSchema()
			//计算机和用户胜局数组的初始化
			for (var i = 0; i < this.count; i++) {
				this.$set(this.myWin, i, 0);
				this.$set(this.computerWin, i, 0);
			}
		},
		onLoad() {

		},
		methods: {
			//重置
			reset() {
				this.over = false;
				this.computerWin = [];
				this.myWin = [];
				this.win = [];
				this.ctx.height = this.ctx.height;
				let data = this.chessBoardmap;
				uni.canvasPutImageData({
					canvasId: 'map',
					x: 0,
					y: 0,
					width: 360,
					height: 360,
					data: data,
					success(res) {
						console.log(res)
					},
					fail(err) {
						console.log(err)
					},
					complete(resp) {
						console.log(resp)
					}
				}, this)
			},
			//用户下棋
			user(ev) {
				if (this.over || !this.me) return;
				var i = Math.floor(ev.detail.x / this.ceilWidth);
				var j = Math.floor(ev.detail.y / this.ceilWidth);
				if (this.chessBoard[i][j] == 0) {
					this.oneStep(i, j, this.me);
					this.chessBoard[i][j] = 1;
					for (var k = 0; k < this.count; k++) {
						if (this.win[i][j][k]) {
							this.myWin[k]++;
							this.computerWin[k] = 6;
							if (this.myWin[k] == 5) {
								this.over = true;
								this.$u.toast('恭喜，你获胜', 3000)
							}
						}
					}
					if (!this.over) {
						this.me = !this.me;
						this.computer();
					}
				} else {
					this.$u.toast("这个位置已经有棋子了！");
				}
			},
			//棋子的实现
			oneStep(i, j, me) {
				this.ctx.beginPath();
				this.ctx.arc(i * this.ceilWidth, j * this.ceilWidth, 10, 0, 2 * Math.PI);
				this.ctx.closePath();
				var gradient = this.ctx.createCircularGradient(12 + i * this.ceilWidth, 13 + j * this.ceilWidth, 13, 12 + i * this.ceilWidth, 13 + j * this.ceilWidth, 0);
				if (me) {
					gradient.addColorStop(0, "#0A0A0A");
					gradient.addColorStop(1, "#636766");
				} else {
					gradient.addColorStop(0, "#D1D1D1");
					gradient.addColorStop(1, "#F9F9F9");
				}
				this.ctx.setFillStyle(gradient);
				this.ctx.fill();
				this.ctx.draw(true);
			},
			//赢局情况
			winSchema() {
				//初始化三维数组
				for (var i = 0; i < 15; i++) {
					this.win[i] = [];
					for (var j = 0; j < 15; j++) {
						this.win[i][j] = [];
					}
				}
				//①横排
				for (var i = 0; i < 15; i++) {
					for (var j = 0; j < 11; j++) {
						for (var k = 0; k < 5; k++) {
							this.win[i][j + k][this.count] = true;
						}
						this.count++;
					}
				}
				//②纵列
				for (var i = 0; i < 15; i++) {
					for (var j = 0; j < 11; j++) {
						for (var k = 0; k < 5; k++) {
							this.win[j + k][i][this.count] = true;
						}
						this.count++;
					}
				}
				//③斜线
				for (var i = 0; i < 11; i++) {
					for (var j = 0; j < 11; j++) {
						for (var k = 0; k < 5; k++) {
							this.win[i + k][j + k][this.count] = true;
						}
						this.count++;
					}
				}
				//④反斜线
				for (var i = 0; i < 11; i++) {
					for (var j = 14; j > 3; j--) {
						for (var k = 0; k < 5; k++) {
							this.win[i + k][j - k][this.count] = true;
						}
						this.count++;
					}
				}
			},
			//电脑下棋
			computer() {
				var myScore = [],
					computerScore = [];
				var maxScore = 0,
					u = 0,
					v = 0;
				for (var i = 0; i < 15; i++) {
					myScore[i] = [];
					computerScore[i] = [];
					for (var j = 0; j < 15; j++) {
						myScore[i][j] = 0;
						computerScore[i][j] = 0;
					}
				}
				for (var i = 0; i < 15; i++) {
					for (var j = 0; j < 15; j++) {
						if (this.chessBoard[i][j] == 0) {
							for (var k = 0; k < this.count; k++) {
								if (this.win[i][j][k]) {
									if (this.myWin[k] == 1) {
										myScore[i][j] += 20;
									} else if (this.myWin[k] == 2) {
										myScore[i][j] += 40;
									} else if (this.myWin[k] == 3) {
										myScore[i][j] += 300;
									} else if (this.myWin[k] == 4) {
										myScore[i][j] += 1000;
									}
									if (this.computerWin[k] == 1) {
										computerScore[i][j] += 220;
									} else if (this.computerWin[k] == 2) {
										computerScore[i][j] += 420;
									} else if (this.computerWin[k] == 3) {
										computerScore[i][j] += 3200;
									} else if (this.computerWin[k] == 4) {
										computerScore[i][j] += 20000;
									}
								}
							}
							if (myScore[i][j] > maxScore) {
								maxScore = myScore[i][j];
								u = i;
								v = j;
							} else if (myScore[i][j] == maxScore) {
								if (computerScore[i][j] > computerScore[u][v]) {
									u = i;
									v = j;
								}
							}
							if (computerScore[i][j] > maxScore) {
								maxScore = computerScore[i][j];
								u = i;
								v = j;
							} else if (computerScore[i][j] == maxScore) {
								if (myScore[i][j] > myScore[u][v]) {
									u = i;
									v = j;
								}
							}
						}
					}
				}
				this.oneStep(u, v, false);
				this.chessBoard[u][v] = 2;
				for (var k = 0; k < this.count; k++) {
					if (this.win[u][v][k]) {
						this.myWin[k] = 6;
						this.computerWin[k]++;
						if (this.computerWin[k] == 5) {
							this.over = true;
							this.$u.toast('计算机获胜', 3000)
						}
					}
				}
				if (!this.over) {
					this.me = !this.me;
				}
			}
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		padding: 12rpx;
	}
</style>
