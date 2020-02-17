<template>
	<el-container id="app">
		<!-- 手机导航栏 -->
		<div id="mobile-bar">
			<img src="./assets/logo.png" alt="" class="menu-button" />
			<div class="logo"><a href="/">Orion-Stress-Tester</a></div>
		</div>
		<!-- 电脑导航栏 -->
		<el-header id="pc-bar">
			<a id="logo" href="/">
				<img src="./assets/logo.png" alt="" />
				<span>Orion-Stress-Tester</span>
			</a>
		</el-header>

		<el-container>
			<!-- 中心栏 -->
			<el-main id="main">
				<el-form :model="requestData" label-width="120px" label-position="right" ref="requestTable">
					<!-- 请求类型选择 -->
					<el-form-item :label="$t('requestType')">
						<el-radio-group v-model="requestData.requestType" :placeholder="$t('select')">
							<el-radio label="HTTP">HTTP/S</el-radio>
							<el-radio label="WebSocket">WebSocket/S</el-radio>
							<el-radio label="TCP">TCP/S</el-radio>
						</el-radio-group>
					</el-form-item>
					<!-- HTTP相关 -->
					<el-form-item
						v-if="requestData.requestType == 'HTTP'"
						:label="$t('requestUrl')"
						prop="url"
						:rules="[{ required: true, message: $t('requestTips') }, { pattern: /^(http|https):\/\/.+$/, message: $t('httpUrlInvalidTips') }]"
						required
					>
						<el-input :placeholder="$t('httpUrlPlaceholder')" v-model="requestData.url">
							<el-select v-model="requestData.method" slot="prepend" placeholder="请选择" style="width: 6rem;color: #222;">
								<el-option value="GET">GET</el-option>
								<el-option value="POST">POST</el-option>
								<el-option value="OPTIONS">OPTIONS</el-option>
								<el-option value="HEAD">HEAD</el-option>
								<el-option value="PUT">PUT</el-option>
								<el-option value="DELETE">DELETE</el-option>
								<el-option value="TRACE">TRACE</el-option>
								<el-option value="CONNECT">CONNECT</el-option>
								<el-option value="PATCH">PATCH</el-option>
								<el-option value="OTHER">OTHER</el-option>
							</el-select>
						</el-input>
					</el-form-item>
					<!-- WebSocket相关 -->
					<el-form-item
						v-if="requestData.requestType == 'WebSocket'"
						:label="$t('requestUrl')"
						prop="url"
						:rules="[{ required: true, message: $t('requestTips') }, { pattern: /^(ws|wss):\/\/.+$/, message: $t('websocketUrlInvalidTips') }]"
						required
					>
						<el-input :placeholder="$t('websocketUrlPlaceholder')" v-model="requestData.url"></el-input>
					</el-form-item>
					<!-- TCP相关 -->
					<el-form-item v-if="requestData.requestType == 'TCP'" :label="$t('requestUrl')" required>
						<el-col :xs="24" :sm="2">
							<el-form-item><el-checkbox v-model="requestData.isSSL">SSL</el-checkbox></el-form-item>
						</el-col>
						<el-col :xs="24" :sm="14">
							<el-form-item prop="host" :rules="[{ required: true, message: $t('tcpHostTips') }]">
								<el-input :placeholder="$t('tcpHostPlaceholder')" v-model="requestData.host"></el-input>
							</el-form-item>
						</el-col>
						<el-col :xs="0" :sm="1">&nbsp;</el-col>
						<el-col :xs="24" :sm="7">
							<el-form-item><el-input type="number" min="0" max="65535" :placeholder="$t('tcpPortPlaceholder')" v-model="requestData.port"></el-input></el-form-item>
						</el-col>
					</el-form-item>
					<!-- 证书选择 -->
					<el-form-item :label="$t('certSetting')" v-if="requestData.isSSL">
						<el-col :span="24">
							<el-radio-group v-model="requestData.cert">
								<el-radio label="DEFAULT">{{ $t('certDefault') }}</el-radio>
								<el-radio label="PEM">{{ $t('certPem') }}</el-radio>
								<el-radio label="PFX">{{ $t('certPfx') }}</el-radio>
								<el-radio label="JKS">{{ $t('certJks') }}</el-radio>
							</el-radio-group>
						</el-col>
						<el-col :span="24" v-if="requestData.cert == 'PEM' || requestData.cert == 'PFX' || requestData.cert == 'JKS'">
							<el-input type="textarea" :placeholder="$t('certKey')" v-model="requestData.certKey"></el-input>
						</el-col>
						<el-col :span="24" v-if="requestData.cert == 'PEM' || requestData.cert == 'PFX' || requestData.cert == 'JKS'">
							<el-input type="textarea" :placeholder="$t('certValue')" v-model="requestData.certValue"></el-input>
						</el-col>
						<el-col v-if="requestData.cert == 'PEM' || requestData.cert == 'PFX' || requestData.cert == 'JKS'">
							<el-button type="primary" size="small" style="margin-right: 5px;" @click="$refs.readCertKey.click()">{{ $t('btnReadCertKey') }}</el-button>
							<input type="file" style="display: none" accept=".pem,.key,.p12,.jks,.txt" ref="readCertKey" @change="loadCertKey" />
							<el-button type="primary" size="small" @click="$refs.readCertValue.click()">{{ $t('btnReadCertValue') }}</el-button>
							<input type="file" style="display: none" accept=".pem,.key,.p12,.jks,.txt" ref="readCertValue" @change="loadCertValue" />
						</el-col>
					</el-form-item>
					<!-- WebSocket的版本 -->
					<el-form-item label="WebSocketVersion" v-if="requestData.requestType == 'WebSocket'">
						<el-select v-model="requestData.webSocketVersion" placeholder="请选择" style="width: 100%;">
							<el-option value="V00">V00</el-option>
							<el-option value="V07">V07</el-option>
							<el-option value="V08">V08</el-option>
							<el-option value="V13">V13</el-option>
						</el-select>
					</el-form-item>
					<!--  WebSocket的SubProtocols -->
					<el-form-item label="SubProtocols" v-if="requestData.requestType == 'WebSocket'">
						<el-col :span="24">
							<el-form-item v-for="(item, index) in requestData.subProtocols" :key="item.key">
								<el-col :xs="20" :sm="21"><el-input v-model="item.value" :placeholder="$t('inputValue')"></el-input></el-col>
								<el-col :xs="1" :sm="1">&nbsp;</el-col>
								<el-col :xs="3" :sm="2">
									<el-button @click.prevent="removeSubProtocols(item)" size="small" type="danger">{{ $t('btnRemove') }}</el-button>
								</el-col>
							</el-form-item>
						</el-col>
						<el-col :span="24">
							<el-button type="primary" size="small" @click="addSubProtocols">{{ $t('btnAdd') }}</el-button>
						</el-col>
					</el-form-item>
					<!-- 请求的Header内容 -->
					<el-form-item :label="$t('requestHeaders')" v-if="requestData.requestType == 'HTTP' || requestData.requestType == 'WebSocket'">
						<el-col :span="24">
							<el-form-item v-for="(item, index) in requestData.headers" :key="index">
								<el-col :xs="24" :sm="10"><el-input v-model="item.key" :placeholder="$t('inputName')"></el-input></el-col>
								<el-col :xs="0" :sm="1">&nbsp;</el-col>
								<el-col :xs="20" :sm="10"><el-input v-model="item.value" :placeholder="$t('inputValue')"></el-input></el-col>
								<el-col :xs="1" :sm="1">&nbsp;</el-col>
								<el-col :xs="3" :sm="2">
									<el-button @click.prevent="removeHeader(item)" size="small" type="danger">{{ $t('btnRemove') }}</el-button>
								</el-col>
							</el-form-item>
						</el-col>
						<el-col :span="24">
							<el-button type="primary" size="small" @click="addHeader">{{ $t('btnAdd') }}</el-button>
						</el-col>
					</el-form-item>
					<!-- 请求的SNI server name内容 -->
					<el-form-item label="ServerName(SNI)" v-if="requestData.requestType == 'TCP'">
						<el-input :placeholder="$t('requestServerNamePlaceholder')" v-model="requestData.serverName"></el-input>
					</el-form-item>
					<!-- 请求的body内容 -->
					<el-form-item :label="$t('requestBody')"><el-input type="textarea" :placeholder="$t('requestBodyPlaceholder')" v-model="requestData.body"></el-input></el-form-item>
					<!-- 请求统计信息 -->
					<el-form-item :label="$t('requestCount')" required>
						<el-col :xs="24" :sm="6">
							<el-form-item prop="count" :rules="[{ required: true, type: 'number', min: 0, message: $t('requestCountTips') }]" required>
								<el-input
									:placeholder="requestData.requestType == 'HTTP' ? $t('requestCountPlaceholder') : $t('requestTcpCountPlaceholder')"
									v-model.number="requestData.count"
								></el-input>
							</el-form-item>
						</el-col>
						<el-col :sm="1">&nbsp;</el-col>
						<el-col :xs="24" :sm="8">
							<el-form-item prop="average" :rules="[{ required: true, type: 'number', min: 0, message: $t('requestAverageTips') }]" required>
								<el-input
									:placeholder="requestData.requestType == 'HTTP' ? $t('requestAveragePlaceholder') : $t('requestTcpAveragePlaceholder')"
									v-model.number="requestData.average"
								></el-input>
							</el-form-item>
						</el-col>
						<el-col :sm="1">&nbsp;</el-col>
						<el-col :xs="24" :sm="8">
							<el-form-item prop="interval" :rules="[{ required: true, type: 'number', min: 1, message: $t('requestIntervalTips') }]" required>
								<el-input
									:placeholder="requestData.requestType == 'HTTP' ? $t('requestIntervalPlaceholder') : $t('requestTcpIntervalPlaceholder')"
									v-model.number="requestData.interval"
								></el-input>
							</el-form-item>
						</el-col>
					</el-form-item>
					<el-form-item v-show="isStatistics">{{ statisticsInfo }}</el-form-item>
					<el-form-item>
						<div style="display: flex;flex-wrap: wrap;">
							<div style="margin-right: 0.6rem">
								<el-tooltip class="item" effect="dark" :content="$t('requestConfigPrintInfoTips')" placement="top">
									<el-checkbox v-model="requestConfig.printResInfo" :label="$t('requestConfigPrintInfo')"></el-checkbox>
								</el-tooltip>
							</div>
							<div style="margin-right: 0.6rem;"><el-input :placeholder="$t('requestConfigTimeout')" type="number" v-model.number="requestConfig.timeout"></el-input></div>
							<div style="margin-right: 0.6rem;"><el-checkbox v-model="requestConfig.keepAlive" :label="$t('requestConfigKeepAlive')"></el-checkbox></div>
							<div style="flex: 1;" v-show="requestConfig.keepAlive">
								<el-input :placeholder="$t('requestConfigPoolSize')" type="number" v-model.number="requestConfig.poolSize"></el-input>
							</div>
						</div>
					</el-form-item>
					<el-form-item>
						<el-button type="primary" :loading="isExecuting" @click="execute">{{ isExecuting ? $t('btnIsExecute') : $t('btnExecute') }}</el-button>
						<el-button v-show="isExecuting" @click="executeCancel">{{ $t('btnCancel') }}</el-button>
					</el-form-item>
				</el-form>
				<div v-show="isExecuted">
					<div id="response-heander" :style="watchResponseHeaderColor">
						<!-- 请求进度 -->
						<div class="response-header-box" v-show="responseSucceeded != 0 || responseFailed != 0">
							<div class="response-header-label">
								<b>{{ $t('executeProgress') }}</b>
							</div>
							<div class="response-header-body">
								<div style="line-height: 34px;" class="sm-margin-left-0x">{{ $t('succee') }}:&nbsp;</div>
								<div style="color: #28a745;font-size: 36px;">{{ responseSucceeded }}</div>
								<div class="small">{{ $t('fail') }}:&nbsp;</div>
								<div style="color: #dc3545;font-size: 36px;">{{ responseFailed }}</div>
								<div class="small">
									&nbsp;{{ responseSucceeded + responseFailed }} / {{ requestData.requestType == 'HTTP' ? requestData.count * requestData.average : requestData.count }}
								</div>
							</div>
						</div>
						<!-- 请求统计信息-->
						<div class="response-header-box" v-show="responseTimeCount.length > 0">
							<div class="response-header-label" style="align-items: flex-start;">
								<b style="line-height: 34px;">{{ $t('responseTimesCount') }}</b>
							</div>
							<div class="response-header-body" style="flex-direction: column;align-items: flex-start;">
								<div style="display: flex;flex-wrap: wrap;" v-for="(item, index) in responseTimeCount" :key="index">
									<div class="small sm-margin-left-0x">{{ $t('responseCode') }}:{{ item.code }}</div>
									<div class="small">{{ $t('responseCount') }}:{{ item.count }}</div>
									<div class="small">{{ $t('responseTotalTime') }}:{{ item.total }}ms</div>
									<div class="small">{{ $t('responseMaxTime') }}:{{ item.max }}ms</div>
									<div class="small">{{ $t('responseMeanTime') }}:{{ item.mean }}ms</div>
								</div>
							</div>
						</div>
						<!-- 数据信息 -->
						<div class="response-header-box" v-show="requestDataLen != 0 || responseDataLen != 0">
							<div class="response-header-label">
								<b>{{ $t('dataLenInfo') }}</b>
							</div>
							<div class="response-header-body">
								<div style="display: flex;flex-wrap: wrap;">
									<div class="small sm-margin-left-0x">{{ $t('requestDataLen') }}:{{ requestDataLen }} byte</div>
									<div class="small">{{ $t('responseDataLen') }}:{{ responseDataLen }} byte</div>
								</div>
							</div>
						</div>
						<!-- 服务器指标 -->
						<div class="response-header-box" v-show="serverMetrics != null && serverMetrics.processors != 0">
							<div class="response-header-label">
								<b>{{ $t('serverMemory') }}</b>
							</div>
							<div class="response-header-body">
								<div style="display: flex;flex-wrap: wrap;">
									<div class="small sm-margin-left-0x">{{ $t('serverProcessors') }}:{{ serverMetrics.processors }}</div>
									<div class="small">{{ $t('serverMaxMemory') }}:{{ serverMetrics.maxMemory }} m</div>
									<div class="small">{{ $t('serverTotalMemory') }}:{{ serverMetrics.totalMemory }} m</div>
									<div class="small">{{ $t('serverFreeMemory') }}:{{ serverMetrics.freeMemory }} m</div>
								</div>
							</div>
						</div>
					</div>
					<div id="response-body" ref="responseConsoleBody"></div>
				</div>
			</el-main>
		</el-container>
	</el-container>
</template>

<script>
/**http请求类型*/
const REQUEST_TYPE_HTTP = 'HTTP';
/**WebSocket请求类型*/
const REQUEST_TYPE_WEB_SOCKET = 'WebSocket';
/**TCP请求类型*/
const REQUEST_TYPE_TCP = 'TCP';
/**证书类型,默认证书*/
const REQUEST_CERT_DEFAULT = 'DEFAULT';
/**证书类型pem*/
const REQUEST_CERT_PEM = 'PEM';
/**证书类型PFX*/
const REQUEST_CERT_PFX = 'PFX';
/**证书类型JKS*/
const REQUEST_CERT_JKS = 'JKS';

/**普通日志*/
const LOG_INFO = 'INFO';
/**成功日志*/
const LOG_SUCCESS = 'SUCCESS';
/**异常日志*/
const LOG_ERROR = 'ERROR';

/** 取消操作 */
const WS_COMMAND_CANCEL = -1;
/** 提交测试*/
const WS_COMMAND_SUBMIT_TEST = 1;
/** 开始的前测试,成功data=1,失败data=0 */
const WS_COMMAND_BEFORE_REQUEST_TEST = 2;
/** 测试数据响应 */
const WS_COMMAND_TEST_RESPONSE = 3;
/** 日志输出 */
const WS_COMMAND_TEST_LOG_RESPONSE = 4;
/** 任务已完成 */
const WS_COMMAND_TEST_COMPLETE = 99;
/** 缺少请求参数或存在无效的请求参数 */
const WS_COMMAND_INVALID_PARAMETER = 412;
/** 失败响应 */
const WS_COMMAND_ERROR = 500;
/** 失败响应 */
const WS_COMMAND_JVM_METRIC = 1000;

export default {
	name: 'Orion-Stress-Tester',
	data() {
		return {
			// 是否显示请求统计信息
			isStatistics: false,
			//请求统计的信息
			statisticsInfo: '',
			//是否正在执行中
			isExecuting: false,
			//是否已经执行过
			isExecuted: false,
			//请求设置
			requestConfig: {
				//是否打印响应内容
				printResInfo: true,
				//测试机与URL服务器保持连接
				keepAlive: true,
				//最大建立连接数
				poolSize: null,
				//连接(请求)超时
				timeout: null
			},
			//请求的数据
			requestData: {
				//请求的类型
				requestType: REQUEST_TYPE_HTTP,
				//WebSocket的版本
				webSocketVersion: 'V13',
				//WebSocket子协议
				subProtocols: [],
				//SNI 服务器名称
				serverName: '',
				//是否使用SSL
				isSSL: false,
				//证书的类型
				cert: 'DEFAULT',
				//证书的key
				certKey: null,
				//证书的value
				certValue: null,
				//主机地址,TCP时有效
				host: '',
				//端口号,TCP时有效
				port: null,
				//http请求的类型
				method: 'GET',
				//请求的header数据
				headers: [],
				//请求的url
				url: '',
				//请求的body
				body: null,
				//请求的总次数
				count: null,
				//每次请求多数次
				average: null,
				//请求的间隔
				interval: null
			},
			//请求数据长度
			requestDataLen: 0,
			//请求响应数据长度
			responseDataLen: 0,
			//请求成功的数量
			responseSucceeded: 0,
			//请求失败的数量
			responseFailed: 0,
			//请求统计信息
			// code=状态码
			// count=请求次数
			// total=总用时;
			// max=最大用时;
			// mean=平均用时;
			responseTimeCount: [],
			//服务器性能
			//processors 处理器数量
			//totalMemory 可用内存
			//maxMemory 最大内存
			//freeMemory 剩余内存
			serverMetrics: {
				processors: 0,
				totalMemory: 0,
				maxMemory: 0,
				freeMemory: 0
			},
			//连接服务器的WebSocket
			websocket: null
		};
	},
	methods: {
		/**
		 * 是否显示请求数量统计信息
		 */
		isShowStatistics() {
			if (this.requestData.count > 0 && this.requestData.average > 0 && this.requestData.interval > 0) {
				this.isStatistics = true;
				this.statisticsInfo = this.$t('statisticsInfo')
					.replace('{interval}', this.requestData.interval)
					.replace('{average}', this.requestData.average)
					.replace('{count}', this.requestData.count)
					.replace('{conn}', this.requestConfig.poolSize == null ? this.requestData.count : this.requestConfig.poolSize)
					.replace('{sum}', this.requestData.count * this.requestData.average);
			} else {
				this.isStatistics = false;
			}
		},
		/**
		 * 删除SubProtocols
		 * @param {Object} item
		 */
		removeSubProtocols(item) {
			this.$confirm('确认关闭？')
				.then(res => {
					var index = this.requestData.subProtocols.indexOf(item);
					if (index !== -1) {
						this.requestData.subProtocols.splice(index, 1);
					}
				})
				.catch(err => {});
		},
		/**
		 * 添加SubProtocols
		 */
		addSubProtocols() {
			this.requestData.subProtocols.push({
				value: '',
				key: Date.now()
			});
		},
		/**
		 * 删除header
		 * @param {Object} item
		 */
		removeHeader(item) {
			this.$confirm('确认关闭？')
				.then(res => {
					var index = this.requestData.headers.indexOf(item);
					if (index !== -1) {
						this.requestData.headers.splice(index, 1);
					}
				})
				.catch(err => {});
		},
		/**
		 * 添加header
		 */
		addHeader() {
			this.requestData.headers.push({
				key: '',
				value: ''
			});
		},
		/**
		 * 加载证书的key
		 */
		loadCertKey() {
			var reader = new FileReader();
			var file = this.$refs.readCertKey.files[0];
			reader.readAsText(file);
			var tis = this;
			reader.onload = function(res) {
				try {
					tis.requestData.certKey = res.target.result;
				} catch (err) {
					console.log(err);
				}
			};
		},
		/**
		 * 加载证书的value
		 */
		loadCertValue() {
			var reader = new FileReader();
			var file = this.$refs.readCertValue.files[0];
			reader.readAsText(file);
			var tis = this;
			reader.onload = function(res) {
				try {
					tis.requestData.certValue = res.target.result;
				} catch (err) {
					console.log(err);
				}
			};
		},
		/**
		 * 清楚数据为默认数据
		 */
		clearToDefaultData() {
			//请求数据长度
			this.requestDataLen = 0;
			//请求响应数据长度
			this.responseDataLen = 0;
			//请求成功的数量
			this.responseSucceeded = 0;
			//请求失败的数量
			this.responseFailed = 0;
			//请求统计信息
			this.responseTimeCount = [];
		},
		/**
		 * 截取小数点并四舍五入
		 * @param {Double} num
		 * @param {int} len
		 */
		toFixed(num, len) {
			var index = num.toString().indexOf('.');
			if (index != -1) {
				if (num.toString().substring(index).length > len) {
					return num.toFixed(len);
				} else {
					return num;
				}
			} else {
				return num;
			}
		},
		/**
		 * 执行请求
		 */
		execute() {
			this.$refs.requestTable.validate(valid => {
				if (valid) {
					this.isExecuting = true; //修改是否在执行状态,执行中
					this.isExecuted = true; //标记前端控制台状态为已执行
					var connectMsg = this.$t('consoleConnecting');
					this.consolePrintInfo(LOG_INFO, connectMsg);
					//连接WebSocket并在open后请求数据
					var wshost=process.env.VUE_APP_BASE_API;
					if(wshost==null || wshost=='wlhost'){
						wshost=window.location.host;
					}
					this.websocket = new WebSocket('ws://'+wshost+'/ws/ost');
					this.websocket.onopen = () => {
						var connected = this.$t('consoleConnected');
						this.consolePrintInfo(LOG_SUCCESS, connected);
						var trData = this.requestData;
						//初始化默认数据
						this.clearToDefaultData();

						//加载请求数据
						var reqData = {
							type: trData.requestType,
							url: trData.url.trim(),
							count: trData.count,
							average: trData.average,
							interval: trData.interval
						};
						if (trData.isSSL) {
							reqData.isSSL = true;
							if (trData.cert == REQUEST_CERT_DEFAULT) {
								reqData.cert = REQUEST_CERT_DEFAULT;
							} else if (trData.certKey != null && trData.certKey.trim() != '' && (trData.certValue != null && trData.certValue.trim() != '')) {
								reqData.cert = trData.cert;
								reqData.certKey = trData.certKey.trim();
								reqData.certValue = trData.certValue.trim();
							}
						}

						if (trData.requestType == REQUEST_TYPE_WEB_SOCKET) {
							reqData.webSocketVersion = trData.webSocketVersion;
							if (trData.subProtocols.length > 0) {
								var subPs = [];
								for (var i = 0; i < trData.subProtocols.length; i++) {
									var v = trData.subProtocols[i].value;
									if (v != null && v.trim() != '') {
										subPs.push(v.trim());
									}
								}
								if (subPs.length > 0) {
									reqData.subProtocols = subPs;
								}
							}
						}
						if (trData.requestType == REQUEST_TYPE_TCP) {
							reqData.host = trData.host.trim();
							if (trData.port == '' || trData < 0 || trData > 65535 || isNaN(parseInt(trData.port))) {
								reqData.port = trData.isSSL ? 443 : 80;
							} else {
								reqData.port = parseInt(trData.port);
							}
							if (trData.serverName != null && trData.serverName.trim() != '') {
								reqData.serverName = trData.serverName.trim();
							}
						}

						if (trData.requestType == REQUEST_TYPE_HTTP) {
							reqData.method = trData.method;
						}

						if (trData.headers != null && trData.headers.length != 0 && trData.requestType != REQUEST_TYPE_TCP) {
							var headers = [];
							for (var i = 0; i < trData.headers.length; i++) {
								var h = trData.headers[i];
								if (h.key.trim() != '' && h.value.trim() != '') {
									headers.push(h.trim());
								}
							}
							if (headers.length > 0) {
								reqData.headers = headers;
							}
						}

						if (trData.body != null && trData.body.trim() != '') {
							reqData.body = trData.body.trim();
						}

						reqData.keepAlive = this.requestConfig.keepAlive;
						if (reqData.keepAlive) {
							var vu = this.requestConfig.poolSize;
							if (vu == null || vu == 0 || vu == '' || isNaN(parseInt(vu))) {
								vu = trData.count;
							}
							reqData.poolSize = parseInt(vu);
						}
						if (this.requestConfig.timeout != null && this.requestConfig.timeout > 0) {
							reqData.timeout = parseInt(this.requestConfig.timeout);
						}

						reqData.printResInfo = this.requestConfig.printResInfo;
						console.log(reqData);
						var body = { code: WS_COMMAND_SUBMIT_TEST, data: reqData };
						this.websocket.send(JSON.stringify(body));
					};

					this.websocket.onerror = err => {
						console.log('Connection error:');
						console.log(err);
						this.isExecuting = false; //修改是否在执行状态,执行结束
						this.consolePrintInfo(LOG_ERROR, this.$t('consoleConnectFailed'));
					};
					//关闭事件
					this.websocket.onclose = () => {
						this.isExecuting = false; //修改是否在执行状态,执行结束

						console.log(this.$t('consoleClosed'));
					};
					//响应事件
					this.websocket.onmessage = res => {
						//这里处理后台返回的结果
						var data = JSON.parse(res.data);
						// console.log(data);
						if (data.code == WS_COMMAND_INVALID_PARAMETER) {
							//缺少参数或取消参数的响应
							this.consolePrintInfo(LOG_ERROR, this.$t('commandInvalidParameter') + data.msg);
							this.websocket.close();
						} else if (data.code == WS_COMMAND_BEFORE_REQUEST_TEST) {
							//提交测试前的测试响应
							if (data.data == 1) {
								this.consolePrintInfo(LOG_SUCCESS, this.$t('commandBeforeRequestTestSucceeded'));
							} else {
								this.consolePrintInfo(LOG_ERROR, this.$t('commandBeforeRequestTestFailed') + data.msg);
								this.websocket.close();
							}
						} else if (data.code == WS_COMMAND_JVM_METRIC) {
							var rd = data.data;
							this.serverMetrics.processors = rd.processors;
							this.serverMetrics.totalMemory = this.toFixed(rd.totalMemory / 1048576, 5);
							this.serverMetrics.maxMemory = this.toFixed(rd.maxMemory / 1048576, 5);
							this.serverMetrics.freeMemory = this.toFixed(rd.freeMemory / 1048576, 5);
						} else if (data.code == WS_COMMAND_TEST_LOG_RESPONSE) {
							//请求日志响应
							var resData = data.data;
							var state = resData.state == 1 ? this.$t('succee') : this.$t('fail');
							var msg = this.$t('commandTestResponseCount').replace('{count}', resData.count);
							if (resData.index != 0) {
								msg += ', ' + this.$t('commandTestResponseIndex').replace('{index}', resData.index);
							}

							msg += ', ' + this.$t('commandTestResponseState').replace('{state}', state);
							msg += ', ' + this.$t('commandTestResponseCode').replace('{code}', resData.code);
							if (resData.body != null) {
								msg += ', ' + this.$t('commandTestResponseBody').replace('{body}', resData.body);
							}
							if (this.requestConfig.printResInfo) {
								this.consolePrintInfo(LOG_INFO, msg);
							}
						} else if (data.code == WS_COMMAND_TEST_RESPONSE) {
							//测试统计信息响应
							var resData = data.data;
							//获取http/ws响应长度数据统计
							var vhcbr = resData['vertx.http.client.bytesReceived'];
							if (vhcbr == null) {
								//获取tcp的响应数据长度
								vhcbr = resData['vertx.net.client.bytesReceived'];
							}
							if (vhcbr != null && vhcbr.length > 0) {
								var total = 0;
								for (var i = 0; i < vhcbr.length; i++) {
									total += vhcbr[i].total;
								}
								this.responseDataLen = total;
							}
							//获取http/ws发送数据长度统计
							var vhcbs = resData['vertx.http.client.bytesSent'];
							if (vhcbs == null) {
								//获取tcp发送数据长度统计
								vhcbs = resData['vertx.net.client.bytesSent'];
							}
							if (vhcbs != null && vhcbs.length > 0) {
								var total = 0;
								for (var i = 0; i < vhcbs.length; i++) {
									total += vhcbs[i].total;
								}
								this.requestDataLen = total;
							}
							//获取http/ws响应统计
							var vhcrt = resData['vertx.http.client.responseTime'];
							if (vhcrt != null && vhcrt.length > 0) {
								this.responseTimeCount = [];
								for (var i = 0; i < vhcrt.length; i++) {
									var obj = vhcrt[i];
									var timeCount = {};
									timeCount.code = obj.tags.code;
									timeCount.count = obj.count;
									timeCount.total = this.toFixed(obj.totalTimeMs, 5);
									timeCount.max = this.toFixed(obj.maxMs, 5);
									timeCount.mean = this.toFixed(obj.meanMs, 5);
									this.responseTimeCount.push(timeCount);
								}
							}
							//获取已成功处理数量
							this.responseSucceeded = resData.succeeded;
							//获取处理已失败数量
							this.responseFailed = resData.failed;
						} else if (data.code == WS_COMMAND_TEST_COMPLETE) {
							this.consolePrintInfo(LOG_SUCCESS, this.$t('commandTestComplete'));
						}
					};
					this.$nextTick(() => {
						document.documentElement.scrollTop = document.body.scrollHeight;
					});
				}
			});
		},
		/**
		 * 取消执行
		 */
		executeCancel() {
			this.isExecuting = false;
			if (this.websocket != null) {
				this.websocket.close();
			}
		},
		consolePrintInfo(log, msg) {
			var time = new Date()
				.toISOString()
				.replace('T', ' ')
				.replace('Z', '');
			var el = document.createElement('div');
			if (log == 'SUCCESS') {
				el.style.color = '#28a745';
			} else if (log == 'ERROR') {
				el.style.color = '#dc3545';
			}
			var logEl = document.createElement('p');
			logEl.style.marginBottom = '0';
			logEl.innerText = '[ ' + log + ' ] ' + time;
			el.appendChild(logEl);

			var msgEl = document.createElement('p');
			msgEl.style.marginTop = '3px';
			msgEl.innerText = msg;
			el.appendChild(msgEl);
			document.getElementById('response-body').appendChild(el);
			this.$nextTick(() => {
				this.$refs.responseConsoleBody.scrollTop = this.$refs.responseConsoleBody.scrollHeight;
			});
		}
	},
	watch: {
		'requestData.url'(val) {
			if (val.indexOf('https://') != -1 && this.requestData.requestType == REQUEST_TYPE_HTTP) {
				this.requestData.isSSL = true;
			} else if (val.indexOf('wss://') != -1 && this.requestData.requestType == REQUEST_TYPE_WEB_SOCKET) {
				this.requestData.isSSL = true;
			} else if (this.requestData.requestType != 'TCP') {
				this.requestData.isSSL = false;
			}
		},
		'requestData.requestType'() {
			if (this.requestData.url.indexOf('https://') != -1 && this.requestData.requestType == REQUEST_TYPE_HTTP) {
				this.requestData.isSSL = true;
			} else if (this.requestData.url.indexOf('wss://') != -1 && this.requestData.requestType == REQUEST_TYPE_WEB_SOCKET) {
				this.requestData.isSSL = true;
			} else if (this.requestData.requestType != REQUEST_TYPE_TCP) {
				this.requestData.isSSL = false;
			}
		},
		'requestData.count'() {
			if (this.requestData.count * this.requestData.average > 10000) {
				this.requestConfig.printResInfo = false;
			} else {
				this.requestConfig.printResInfo = true;
			}
			this.isShowStatistics();
		},
		'requestData.average'() {
			if (this.requestData.count * this.requestData.average > 10000) {
				this.requestConfig.printResInfo = false;
			} else {
				this.requestConfig.printResInfo = true;
			}
			this.isShowStatistics();
		},
		'requestData.interval'() {
			this.isShowStatistics();
		},
		'requestConfig.poolSize'() {
			this.isShowStatistics();
		}
	},
	computed: {
		watchResponseHeaderColor() {
			// 获取响应状态
			if (this.responseSucceeded > this.responseFailed) {
				return 'background-color:#e8f6f0;';
			} else if (this.responseSucceeded < this.responseFailed) {
				return 'background-color:#fae7e7;';
			} else {
				return '';
			}
		}
	}
};
</script>

<style>
/* 修改框架样式 */
.el-input__inner::placeholder {
	color: #333 !important;
}

.el-textarea__inner::placeholder {
	color: #333 !important;
}

/* 通用样式 */
a {
	text-decoration: none;
	color: #304455;
}
img {
	border: none;
}
.sm-margin-left-0x {
	margin-left: 0 !important;
}

@media screen and (max-width: 768px) {
	#mobile-bar {
		display: block !important;
	}
	#pc-bar {
		display: none !important;
	}
	#aside {
		padding-top: 40px !important;
		width: 100% !important;
	}
	#main {
		padding-top: 60px !important;
	}
	#response-heander .response-header-label {
		width: 100% !important;
		text-align: left !important;
		justify-content: left !important;
	}
	.sm-margin-left-0x {
		margin-left: 0.6rem !important;
	}
}

/* 手机导航栏 */
#mobile-bar {
	position: fixed;
	display: none;
	top: 0;
	left: 0;
	width: 100%;
	height: 40px;
	background-color: #fff;
	z-index: 20;
	box-shadow: 0 0 2px rgba(0, 0, 0, 0.25);
}
#mobile-bar .menu-button {
	position: absolute;
	top: 8px;
	left: 12px;
	height: 25px;
	width: 25px;
	z-index: 2;
}

#mobile-bar .logo {
	position: absolute;
	top: 10px;
	text-align: center;
	width: 100%;
}

/* 电脑导航栏 */
#pc-bar {
	position: fixed;
	width: 100%;
	top: 0;
	width: 100%;
	box-shadow: 0 0 1px rgba(0, 0, 0, 0.25);
	transition: background-color 0.3s ease-in-out;
	background-color: #fff;
	height: 40px;
	padding: 10px 60px;
	z-index: 10;
}
#logo {
	display: inline-block;
	font-size: 1.5em;
	line-height: 40px;
	color: #273849;
	font-family: 'Dosis', 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif;
	font-weight: 500;
}
#logo img {
	vertical-align: middle;
	margin-right: 6px;
	width: 40px;
	height: 40px;
}

/* #nav {
	position: fixed;
}
#nav {
	list-style-type: none;
	margin: 0;
	padding: 0;
	position: absolute;
	right: 30px;
	top: 10px;
	height: 40px;
	line-height: 40px;
}
#nav li {
	display: inline-block;
	position: relative;
	margin: 0 0.6em;
	cursor: pointer;
}
#nav li a:hover {
	padding-bottom: 2px;
	border-bottom: 2px solid #42b983;
}
 */

#main {
	color: #333;
	padding-top: 80px;
	max-width: 1024px;
	margin-left: auto;
	margin-right: auto;
	margin-bottom: 10px;
	scrollbar-width: thin;
}

#response-heander {
	background-color: #f5f7fa;
	border-radius: 5px 5px 0 0;
	padding: 10px 5px;
}
#response-heander .response-header-box {
	display: flex;
	flex-wrap: wrap;
}
#response-heander .response-header-label {
	width: 105px;
	margin-right: 10px;
	display: flex;
	text-align: right;
	justify-content: right;
	align-items: center;
}
#response-heander .response-header-body {
	display: flex;
	flex-wrap: wrap;
	justify-content: left;
	align-items: flex-end;
}

#response-heander .response-header-body .small {
	line-height: 34px;
	margin-left: 0.6rem;
}

#response-charts-line {
	background-color: #f5f7fa;
	padding-top: 10px;
}

#response-body {
	background-color: #18181a;
	color: white;
	padding: 5px;
	height: 40vh;
	border-radius: 0 0 5px 5px;
	overflow-y: auto;
	scrollbar-width: thin;
}
</style>
