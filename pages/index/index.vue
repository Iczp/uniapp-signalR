<template>
	<view class="content">
		<view class="logo" @click="setTitle">{{ title }}</view>
		<view :loginToken="accessToken" :change:loginToken="signalR.setLoginToken" :title="title"
			:change:title="signalR.changeTitle">
		</view>

		<view class="" :deviceId="deviceId" :change:deviceId="signalR.setDeviceId">deviceId:{{deviceId}} </view>
		<view class="text-area">
			<text class="title"></text>
		</view>

		<view class="" @click="stop">
			<text class="">Stop[]</text>
		</view>

		<block v-if="connInfo">
			<view class="">
				serverTimeoutInMilliseconds:{{connInfo.serverTimeoutInMilliseconds}}
			</view>
			<view class="">
				keepAliveIntervalInMilliseconds:{{connInfo.keepAliveIntervalInMilliseconds}}
			</view>
			<view class="">
				state:{{connInfo.state}}
			</view>
			<view class="">
				baseUrl:{{connInfo.baseUrl}}
			</view>
			<view class="">
				connectionId:{{connInfo.connectionId}}
			</view>
		</block>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				num: 0,
				connProxy: null,
				connInfo: {
					connectionId: null,
					baseUrl: null,
					state: null,
					serverTimeoutInMilliseconds: null,
					keepAliveIntervalInMilliseconds: null,
				},
				deviceId: null,
				accessToken: `eyJhbGciOiJSUzI1NiIsImtpZCI6IkMxRTc4RTlDNEFDRkNCOTg0OTIwMzQxMjk0ODcyODRBMjkyNUI0NEMiLCJ4NXQiOiJ3ZWVPbkVyUHk1aEpJRFFTbEljb1Npa2x0RXciLCJ0eXAiOiJhdCtqd3QifQ.eyJpc3MiOiJodHRwOi8vMTAuMC41LjIwOjgwNDMvIiwiZXhwIjoxNzQxODY3OTA3LCJpYXQiOjE3NDE3ODE1MDcsImF1ZCI6IklNIiwic2NvcGUiOiJJTSIsImp0aSI6ImQzMzUxODAzLWQ5MmYtNGFmYi04OWJjLWRhMmQ0YTAyYjc2YSIsInN1YiI6IjM2MGNmZWRiLWU5MmQtMzMzMS0xZmFkLTNhMDg2MzcxZTBlNCIsInByZWZlcnJlZF91c2VybmFtZSI6ImFkbWluIiwiZW1haWwiOiIxMDAwQGludHJ5LmNuIiwicm9sZSI6ImFkbWluIiwiZ2l2ZW5fbmFtZSI6Inpob25ncGVpIiwiZmFtaWx5X25hbWUiOiJjaGVuIiwicGhvbmVfbnVtYmVyX3ZlcmlmaWVkIjoiRmFsc2UiLCJlbWFpbF92ZXJpZmllZCI6IkZhbHNlIiwidW5pcXVlX25hbWUiOiJhZG1pbiIsIm9pX3Byc3QiOiJJTV9Td2FnZ2VyIiwib2lfYXVfaWQiOiI4MGE1YjA1OS00YjhiLTdjZGItYmZjMS0zYTEyMmJkODlhMTYiLCJjbGllbnRfaWQiOiJJTV9Td2FnZ2VyIiwib2lfdGtuX2lkIjoiZTIwZDRmNTktYjIwYy0xYWM4LWM0NWYtM2ExODljNzE3NTVmIn0.vDiceeDbbuWW96phcDyckFdNGbjW7XQ9-LJZffX8vgGTABssbVTA3wMqbZlJYT4bST_vcQuXX85dC_Xshup_9fDnMjoAvSSKwq3XCmH2hhxLI4YLKJvwCoVKNcC6d-pX0tx7JtaHlXb5vfH7mQQ6sCbfqIHg4fY-Uf6V0hvbWiPFyI-FgHEHGpa_HAp9FbEByphRI-El4t2QzAD_1dm-VWuTV_FHWITQGwg7s0P1e_FxV64u43xv-Jx812HrO4Px1M1No1j6kG_t0JnOb2-ksPeCTkn7McvYmdfy3olkdxnDGU-EZqfCMmD9ZTSu5_fw8nLvx9fpSJEMxSfU2lPMYw`,
				title: 'Rctea',
			};
		},
		async mounted() {
			const systemInfo = await uni.getSystemInfo()
			const { appId, deviceId, uniPlatform } = systemInfo
			this.deviceId = `${uniPlatform}-${appId}-${deviceId}`
			console.log('deviceId', this.deviceId)
			console.log('systemInfo', systemInfo)
		},
		methods: {
			setConn(conn) {
				console.log("[setConn] Vue", conn);
				this.connProxy = conn
			},
			setConnInfo(connInfo) {
				console.log("[setConnInfo] Vue", connInfo);
				this.connInfo = connInfo
			},
			setTitle() {
				this.num++;
				this.title = `Rctea-${this.num}`;
				console.log('setTitle:', this.title);
			},
			stop() {
				console.log('stop');
			},
			test(e) {
				console.log('methods test', e);
				uni.showToast({ title: `Rctea:${e}`, icon: 'none' });
			},
		},
		onLoad() {},
	};
</script>
<script module="signalR" lang="renderjs">
	console.log('renderjs -signalR')
	let signalR;
	let lockResolver;
	let deviceId;
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
				this.initSignalR('function')
			} else {
				// 动态引入较大类库避免影响页面展示
				const script = document.createElement('script')
				// view 层的页面运行在 www 根目录，其相对路径相对于 www 计算
				script.src = 'static/signalr.min.js'
				script.onload = this.initSignalR.bind(this, ['script'])
				document.head.appendChild(script)
				console.log('appendChild script.src:', script.src)
			}

			// ...
		},
		data() {
			return {
				connection: null,
				loginToken: '',
				deviceId: '',
			}
		},
		methods: {
			setDeviceId(newValue, oldValue, ownerInstance, instance) {
				if (!newValue) {
					console.warn('setDeviceId', newValue, oldValue, ownerInstance, instance)
					return;
				}
				this.deviceId = newValue
				console.log('setDeviceId', newValue, oldValue, ownerInstance, instance)
			},
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
				// console.log('setLoginToken', newValue, oldValue, ownerInstance, instance)
			},

			receiveMessage(msg) {
				console.log("ReceiveMessage", msg);
				this.$ownerInstance.callMethod('test', msg)
			},
			onconnected(conn) {
				console.log("[onconnected]", conn);
				const { connectionId, state, baseUrl, serverTimeoutInMilliseconds, keepAliveIntervalInMilliseconds } = conn
				const connInfo = { connectionId, state, baseUrl, serverTimeoutInMilliseconds, keepAliveIntervalInMilliseconds }
				console.log('connInfo', JSON.stringify(connInfo))
				this.$ownerInstance.callMethod('setConnInfo', connInfo)
				this.$ownerInstance.callMethod('setConn', conn)
			},
			onreconnected(connectionId) {
				console.log("[onreconnected]", connectionId);
			},
			onreconnecting(connectionId) {
				console.log("onreconnecting", connectionId);
			},
			async send(msg) {
				console.log("send:", msg);
				await this.connection.invoke("SendMessageAsync", "123456", msg);
			},
			async onclose(msg) {
				// Error: WebSocket closed with status code: 1006 (no reason given).
				// Error: Server returned an error on close: Connection closed with an error. at app-renderjs.js:115
				console.warn("[onclose]", msg);
				setTimeout(() => this.start(), 5000);
			},
			async start() {
				try {
					const connection = this.connection;
					await connection.start();
					console.assert(connection.state === signalR.HubConnectionState.Connected);
					this.onconnected(connection)
					// await this.send(`'${connection.connectionId}' SignalR Connected.`);
				} catch (err) {
					// Error: Failed to complete negotiation with the server: Error: Unauthorized: Status code '401'
					//errorType:FailedToNegotiateWithServerError
					for (const e in err) {
						console.warn('[err]', typeof e, e, err[e]);
					}
					console.error('[start error]', typeof err, err.toString(), err.errorType);
					setTimeout(() => this.start(), 5000);
				}
			},
			async initSignalR(args) {
				console.log('initSignalR', args)
				signalR = window.signalR
				console.log('window.signalR', signalR)
				console.log('window.signalR deviceId', this.deviceId)
				const connection = this.connection = new signalR.HubConnectionBuilder()
					.withUrl(`http://10.0.5.20:8044/signalr-hubs/chat?deviceId=${this.deviceId}`, {
						accessTokenFactory: () => this
							.loginToken
					})
					// .withAutomaticReconnect()
					.configureLogging(signalR.LogLevel.Information)
					.build();
				connection.on("ReceivedMessage", this.receiveMessage)
				// connection.onconnected(this.onconnected)
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