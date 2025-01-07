<template>
	<view class="content" @click="setTitle">
		<image class="logo" src="/static/logo.png"></image>
		<view :loginToken="accessToken" :change:loginToken="signalR.setLoginToken" :title="title"
			:change:title="signalR.changeTitle">

		</view>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				num: 0,
				accessToken: '123',
				title: 'signalR'
			}
		},
		methods: {
			setTitle() {
				this.num++;
				this.title = `signalR-${this.num}`
				console.log('setTitle:', this.title)
			},
			test(e) {
				console.log('methods test', e)
				uni.showToast({ title: `signalR:${e}`, icon: 'none' })
			}
		},
		onLoad() {

		}
	}
</script>
<script module="signalR" lang="renderjs">
	console.log('renderjs -signalR')
	let signalR;
	let lockResolver;
	const isWebLockEnabled = navigator && navigator.locks && navigator.locks.request
	if (isWebLockEnabled) {
		const promise = new Promise((res) => lockResolver = res);
		navigator.locks.request('signalR_unique_lock_name', { mode: "shared" }, () => promise);
	}
	console.log(' Web Lock enabled:', !!isWebLockEnabled, lockResolver)
	export default {
		mounted() {
			if (typeof window.signalR === 'function') {
				this.initSignalR()
			} else {
				// 动态引入较大类库避免影响页面展示
				const script = document.createElement('script')
				// view 层的页面运行在 www 根目录，其相对路径相对于 www 计算
				script.src = 'static/signalr.min.js'
				script.onload = this.initSignalR.bind(this)
				document.head.appendChild(script)
				console.log('appendChild script.src:', script.src)
			}

			// ...
		},
		data() {
			return {
				connection: null,
				loginToken: ''
			}
		},
		methods: {
			async changeTitle(newValue, oldValue, ownerInstance, instance) {
				console.log('changeTitle', newValue, oldValue, ownerInstance, instance)
				if (this.connection) {
					await this.send(newValue);
				} else {
					console.error('this.connection:', this.connection)
				}
			},
			setLoginToken(newValue, oldValue, ownerInstance, instance) {
				this.loginToken = newValue;
				console.log('setLoginToken', newValue, oldValue, ownerInstance, instance)
			},

			receiveMessage(msg) {
				console.log("ReceiveMessage", msg);
				this.$ownerInstance.callMethod('test', msg)
			},
			onreconnected(connectionId) {
				console.log("onreconnected", connectionId);
			},
			onreconnecting(connectionId) {
				console.log("onreconnecting", connectionId);
			},
			async send(msg) {
				console.log("send:", msg);
				await this.connection.invoke("SendMessageAsync", "123456", msg);
			},
			async onclose(error) {
				console.log("onclose", error);
				await this.start();
			},
			async start() {
				try {
					const connection = this.connection;
					await connection.start();
					console.assert(connection.state === signalR.HubConnectionState.Connected);
					await this.send(`'${connection.connectionId}' SignalR Connected.`);
				} catch (err) {
					console.assert(connection.state === signalR.HubConnectionState.Disconnected);
					console.error('[start error]', err);
					setTimeout(() => this.start(), 5000);
				}
			},
			async initSignalR() {
				console.log('initSignalR')
				signalR = window.signalR
				console.log('window.signalR', signalR)
				const connection = this.connection = new signalR.HubConnectionBuilder()
					.withUrl("http://10.0.5.20:8070/signalr-hubs/chat", { accessTokenFactory: () => this.loginToken })
					// .withAutomaticReconnect()
					.configureLogging(signalR.LogLevel.Information)
					.build();
				connection.on("ReceiveMessage", this.receiveMessage)
				connection.onreconnected(this.onreconnected)
				connection.onreconnecting(this.onreconnecting)
				connection.onclose(this.onclose);
				await this.start();
			},
		}
	}
</script>


<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>