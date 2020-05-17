<template>
	<view class="content">
		<image :src='imgSrc'></image><br/>
		<view wx:if='{{textInfo.flag}}' class="textmsg">
			<text>年龄：{{textInfo.age}}</text>
			<text>颜值：{{textInfo.beauty}}</text>
		</view>
		<view class="btn">
			<button type="default" @click="btnclick" data-typebtn="anime">动漫</button>
			<button type="default" @click="btnclick" data-typebtn="n95">口罩</button>
			<button type="default" @click="btnclick" data-typebtn="test">检测</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imgSrc: '../../static/back.jpg',
				token: '',
				textInfo: {
					flag: false,
					age: 0 ,
					beauty: 0
				}
			}
		},
		onLoad() {
			this.getbaidutoken()
		},
		methods: {
			// 开局获取token
			getbaidutoken() {
				if (uni.getStorageSync("access_token")) {
					return this.token = uni.getStorageSync("access_token")
				}
				// 请求获取token
				uni.request({
					method: "POST",
					url: "https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=cT1WxwtfYfQigtXncYEMwHF6&client_secret=rnshLTNFvgM9PmAxDGY71it5POOSbTdk",
					success: (res) => {
						console.log(res)
						// 判断是否请求成功
						if (res.data.error) {
							return uni.showToast({
								title: "请求失败!请联系管理员 ",
								icon: none
							})
						} else {
							// 成功
							this.token = res.data.access_token
							console.log('token', res.data)
							uni.setStorageSync('access_token', res.data.access_token)
							uni.showToast({
								title: "OK! "
							})

						}
					}
				})
			},
			// 点击按钮  功能
			btnclick(e) {
				this.textInfo.flag= false
				let btntype = e.currentTarget.dataset.typebtn
				// 判段哪个按钮
				uni.chooseImage({
					count: 1,
					sizeType: ['original'],
					success: (res) => {
						// 调用图片转换格式函数
						uni.showLoading({
							title: '制作中'
						})
						btntype === 'test'? this.iamgeScore(res.tempFilePaths[0]) : this.imagechange(res.tempFilePaths[0], btntype)
					}
				})
			},
			iamgeScore(img) {
				uni.uploadFile({
					url:'https://ai.qq.com/cgi-bin/appdemo_detectface?g_tk=5381',
					filePath: img,
					 name:"image_file", // 传给后台的键名
					success:(res)=>{
						let rep = JSON.parse(res.data)
						console.log('图片检测结果',rep)
						console.log(img)
						if (rep.ret== 16404) {
							uni.showToast({
								title:"抱歉！上传的土拍你有误，请选择正确的图片！！~",
								icon:'none'
							})
						}else {
							this.textInfo.age=rep.data.face[0].age
							this.textInfo.beauty=rep.data.face[0].beauty	
							this.textInfo.flag= true
							this.imgSrc = img
						}
						uni.hideLoading()
						
					}
				})
			},
			// 图片转换BASE64
			imagechange(imgsr, typebtn) {
				
				console.log('imgsrc', imgsr)
				wx.getFileSystemManager().readFile({
					filePath: imgsr,
					encoding: 'base64',
					success: (res) => {
						console.log('64', res)
						let image64 = res.data
						let data = {
							image: image64,
							type: 'anime'
						}
						// 如果typebtn为N95则改为戴口罩的
						if (typebtn === 'n95') {
							data.type = 'anime_mask'
							data.mask_id = Math.ceil(Math.random() * 8)
							// console.log('data',data)
						}
						// 调用动画化函数
						this.imageTO(data)
					}
				})
			},
			// 请求动画化
			imageTO(data) {
				uni.request({
					method: 'POST',
					url: 'https://aip.baidubce.com/rest/2.0/image-process/v1/selfie_anime?access_token='+this.token,
					header: {
						"Content-Type": 'application/x-www-form-urlencoded'
					},
					data,
					success: (res) => {
						console.log('动漫画', res)
						res.data.image ? this.imgSrc = 'data:image/jpg;base64,' + res.data.image : uni.showToast({
							title: "请求错误",
							icon: "none"
						})
						uni.hideLoading()
					}
				})
			}
		}
	}
</script>

<style>
	page,
	.content,
	image {
		width: 100%;
		height: 100%;
	}
	.textmsg {
		position: fixed;
		left: 50%;
		transform: translateX(-50%);
		bottom: 2vh;
		text-align: center;
		width: 50vh;
		background-color: #fcfcfc;
	}
	.textmsg text {
		color: #000000;
		font-size: 16px;
		font-weight: 400;
		margin-right: 20rpx;
		
	}
	.btn {
		width: 100%;
		display: flex;
		flex-wrap: wrap;
		position: fixed;
		left: 0;
		bottom: 8%;
		
	}

	.btn button {
		background-color: #e64f7f;
		width: 40%;
		height: 100rpx;
		color: #fff;
		margin-top: 30rpx;
	}
</style>
