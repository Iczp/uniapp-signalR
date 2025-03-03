<template>
	<view class="content" @click="setTitle">
		<image class="logo" src="/static/logo.png"></image>
		<view :loginToken="accessToken" :change:loginToken="signalR.setLoginToken" :title="title"
			:change:title="signalR.changeTitle">
		</view>
		<view class="text-area">
			<text class="title">{{ title }}</text>
		</view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				num: 0,
				accessToken: `eyJhbGciOiJSUzI1NiIsImtpZCI6IjNEOERGQjVERkJCNjdDMjJCMEQyMEY0RjBBRTU1RUJFQkNDRjM1NjYiLCJ4NXQiOiJQWTM3WGZ1MmZDS3cwZzlQQ3VWZXZyelBOV1kiLCJ0eXAiOiJhdCtqd3QifQ.eyJpc3MiOiJodHRwczovL2xvY2FsaG9zdDo0NDMzMC8iLCJleHAiOjE3NDEwMTE4OTIsImlhdCI6MTc0MTAwODI5MiwiYXVkIjoiQ2hhdCIsInNjb3BlIjoiQ2hhdCIsImp0aSI6ImIwMWJjYzZiLTQ2ZWQtNDRjYi1hNmE3LTQzMDViNmM0M2UwNSIsInN1YiI6IjFhMWMzYjFkLTBhM2ItZThlYi02NGIyLTNhMDdjZmQyZDhjYSIsInByZWZlcnJlZF91c2VybmFtZSI6ImFkbWluIiwiZW1haWwiOiJhZG1pbkBhYnAuaW8iLCJyb2xlIjoiYWRtaW4iLCJnaXZlbl9uYW1lIjoiYWRtaW4iLCJwaG9uZV9udW1iZXJfdmVyaWZpZWQiOiJGYWxzZSIsImVtYWlsX3ZlcmlmaWVkIjoiRmFsc2UiLCJ1bmlxdWVfbmFtZSI6ImFkbWluIiwicmVtZW1iZXJfbWUiOiJUcnVlIiwib2lfcHJzdCI6IkNoYXRfU3dhZ2dlciIsIm9pX2F1X2lkIjoiMDY3MzcxNDUtMDEzMS01YmM5LWE5YTYtM2ExNzc1YzM2YjA4IiwiY2xpZW50X2lkIjoiQ2hhdF9Td2FnZ2VyIiwib2lfdGtuX2lkIjoiZmNjMmMyYjMtZDE0NS1hYzFjLWJlYWMtM2ExODZlNWIyMGYwIn0.Wi9o5slPmHyimkQkIp7-QY26mN6P_5ChWhxhBX_Xbdf0uSPlun1-Bp1fIuJpDzV8TDQL-GNWHzb12oX6f8Q0yvMSmjTBBi6SEY16L5QF5Jw5izxKWs7_IpoOhe8FEE-Je6pyX0iN8SXrQCXbECHQD8TnptWkFiyV8HoLpgGhSuwtUxQmVHto1cFiLoaQKRRS5ovonrkIkmBBpRPPFDDAaPBP8ginrzKUX9KnfS6yj4azGckP5Irhe1IYZTVKd21iQQdgNo0YBRyKJNRjaUWIh26hGhZTNLgBmRJ7ZA0PMxjyDYHoPOTdBI1pJyhlEFvUpp8QiVwMxqwTqXYw0iLRPg`,
				title: 'signalR',
			};
		},
		methods: {
			setTitle() {
				this.num++;
				this.title = `signalR-${this.num}`;
				console.log('setTitle:', this.title);
			},
			test(e) {
				console.log('methods test', e);
				uni.showToast({ title: `signalR:${e}`, icon: 'none' });
			},
		},
		onLoad() {},
	};
</script>
<script module="signalR" lang="renderjs">
	console.log('renderjs -signalR')
	let signalR;
	let lockResolver;
	const isWebLockEnabled = navigator && navigator.locks && navigator.locks.request
	if (isWebLockEnabled) {
		// 使用 Web Lock 将选项卡保持为唤醒状态，并避免意外的连接关闭。
		const promise = new Promise((res) => lockResolver = res);
		navigator.locks.request('signalR_unique_lock_name', { mode: "shared" }, () => promise);
		// 关闭连接时，通过调用 lockResolver() 释放锁定。 释放锁定时，选项卡可进入睡眠状态
		// lockResolver?.call(this)
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