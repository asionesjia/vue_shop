<template>
	<el-container>
	  <el-header>
			<div>
				<img src="../assets/logo.png" alt="">
				<span>电商系统管理后台</span>
			</div>
			<el-button type="info" @click="logout">退出</el-button>
		</el-header>
	  <el-container>
	    <el-aside width="200px" :width="isCollapse ? '64px' : '200px'">
				<div class="toggle-button" @click="toggleCollapse">
					<i :class="iconObj.toggleIcon"></i>
				</div>
				<el-menu
				default-active="2"
				class="el-menu-vertical-demo"
				background-color="#333744"
				text-color="#fff"
				active-text-color="#409EFF"
				:unique-opened="true"
				:collapse="isCollapse"
				:collapse-transition="false"
				:router="true"
				:default-active="activePath"
				>
					<el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
						<template slot="title">
							<i :class="iconObj[item.id]" class="iconfont"></i>
							<span>{{item.authName}}</span>
						</template>
						<el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
							<i class="el-icon-menu"></i>
							<span>{{subItem.authName}}</span>
						</el-menu-item>
					</el-submenu>
				</el-menu>				
			</el-aside>
	    <el-main>
				<router-view></router-view>
			</el-main>
	  </el-container>
	</el-container>
</template>

<script>
	export default {
		data() {
			return {
				menulist: [],
				iconObj: {
					'toggleIcon': 'el-icon-s-fold',
					'125': 'el-icon-user-solid',
					'103': 'el-icon-s-platform',
					'101': 'el-icon-s-goods',
					'102': 'el-icon-s-order',
					'145': 'el-icon-s-marketing'
				},
				isCollapse: false,
				activePath: ''
			}
		},
		created() {
			this.getMenuList(),
			this.activePath = window.sessionStorage.getItem('activePath')
		},
		methods: {
			logout() {
				window.sessionStorage.clear()
				this.$message.success("退出成功")
				this.$router.push("/login")
			},
			async getMenuList() {
				const {data: res} = await this.$http.get('menus')
				if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
				this.menulist = res.data
			},
			toggleCollapse() {
				this.isCollapse = !this.isCollapse
			},
			saveNavState(activePath) {
				window.sessionStorage.setItem('activePath', activePath)
				this.activePath = activePath
			}
		}
	};
</script>

<style lang="less" scoped>
	.el-container {
		height: 100%;
	}
	.el-header {
		background-color: #373D41;
		display: flex;
		justify-content: space-between;
		align-items: center;
		color: #fff;
		font-size: 20px;
		> div {
			display: flex;
			align-items: center;
			span {
				margin-left: 15px;
			}
		}
	}
	.el-aside {
		background-color: #333744;
		.el-menu {
			border-right: none;
		}
	}
	.el-main {
		background-color: #EAEDF1;
	}
	img {
		width: 80px;
		height: 80px;
	}
	.iconfont {
		margin-left: 5px;
		margin-right: 10px;
	}
	.toggle-button {
		background-color: #4A5064;
		font-size: 18px;
		line-height: 24px;
		color: #fff;
		text-align: center;
		cursor: pointer;
	}
</style>
