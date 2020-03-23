<template>
	<div class="login-container">
		<h3 class="title">教师后台</h3>
		<el-form :model="loginForm" :rules="rules" ref="loginForm" label-position="top" label-width="80px" class="login-form">
			<el-form-item label="学校">
				<el-select v-model="loginForm.universityId">
					<el-option v-for="university in universities" :key="university.id" :label="university.name" :value="university.id"></el-option>
				</el-select>
			</el-form-item>
			<el-form-item prop="account" label="工号">
				<el-input size="large" type="text" v-model="loginForm.account" placeholder="工号"></el-input>
			</el-form-item>
			<el-form-item prop="password" label="密码">
				<el-input size="large" type="password" v-model="loginForm.password" placeholder="密码"></el-input>
			</el-form-item>
			<el-form-item style="width:100%;">
				<el-button size="large" type="primary" style="width:100%;" @click.native.prevent="login" :loading="logining">登录</el-button>
				<router-link to="/password">设置登录密码</router-link>
				<router-link to="/resetpass">修改登录密码</router-link>
			</el-form-item>
		</el-form>
	</div>
</template>

<script>
	import md5 from 'js-md5'
	import resources from '../resources'
	const universitiesQuery = `
{
  	universities {
		id
		name
  	}
}`;

	export default {
		data() {
			return {
				universities: [],
				logining: false,
				loginForm: {
					universityId: '',
					account: '',
					password: '',
					deviceId: '0000000000000000'
				},
				rules: {
					account: [
						{ required: true, message: '请输入账号', trigger: 'blur' },
					],
					password: [
						{ required: true, message: '请输入密码', trigger: 'blur' },
					]
				},
			};
		},
		methods: {
			login() {
				var _this = this;
				let loginParams = {
					username: this.loginForm.account,
					password:  md5(this.loginForm.password),
					universityId: this.loginForm.universityId,
					deviceId: this.loginForm.deviceId,
					expiredHour: 168
				};
				resources.universityId = this.loginForm.universityId;
				this.$refs.loginForm.validate((valid) => {
					if (valid) {
						this.logining = true;
						let url = resources.requestLogin();
						this.$ajax({
							method: 'post',
							url: url,
							timeout: 10000,
							params: loginParams
						}).then((res) => {
							this.logining = false;
							sessionStorage.setItem('universityId', resources.universityId);
							sessionStorage.setItem('token', res.data.obj.token);
							sessionStorage.setItem('userName',this.loginForm.account);
							this.$router.push({ path: '/teachers' });
						}, (resolve, reject, response) => {
							console.log(resolve, reject, response)
							this.logining = false;
							this.$message({
								message: '登录失败',
								type: 'error'
							});
						});
					} else {
						console.log('valid:false!!!');
						return false;
					}
				});
			},
			getUniversities() {
				this.$ajax.post(`${resources.graphQlApi}`, {
					'query': `${universitiesQuery}`
				})
					.then(res => {
						this.universities = res.data.data.universities;
					});
			}
		},
		mounted: function () {
			this.getUniversities();
		}
	}

</script>

<style lang="scss" scoped>
	.login-container {
		-webkit-border-radius: 5px;
		border-radius: 5px;
		-moz-border-radius: 5px;
		background-clip: padding-box;
		margin-top: 140px;
		.login-form {
			margin: 0 auto;
			width: 350px;
			padding: 35px 35px 15px 35px;
			background: #fff;
			border: 1px solid #eaeaea;
			box-shadow: 0 0 25px #cac6c6;
		}
		.title {
			font-size: 28px;
			margin: 0px auto 30px auto;
			text-align: center;
			color: #505458;
		}
		.remember {
			margin: 0px 0px 35px 0px;
		}
		.el-select {
			width: 100%;
		}
	}
</style>