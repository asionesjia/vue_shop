<template>
	<div>
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>用户管理</el-breadcrumb-item>
			<el-breadcrumb-item>用户列表</el-breadcrumb-item>
		</el-breadcrumb>
		<el-card class="box-card">
			<el-row :gutter="20">
				<el-col :span="8">
					<el-input v-model="queryInfo.query" placeholder="请输入内容" class="input-with-select" clearable
						@clear="getUserList">
						<el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
					</el-input>
				</el-col>
				<el-col :span="4">
					<el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
				</el-col>
			</el-row>
			<el-table :data="userList" border stripe>
				<el-table-column type="index"></el-table-column>
				<el-table-column prop="username" label="姓名"></el-table-column>
				<el-table-column prop="email" label="邮箱"></el-table-column>
				<el-table-column prop="mobile" label="电话"></el-table-column>
				<el-table-column prop="role_name" label="角色"></el-table-column>
				<el-table-column label="状态">
					<template slot-scope="scope">
						<el-switch v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949"
							@change="userStateChanged(scope.row)">
						</el-switch>
					</template>
				</el-table-column>
				<el-table-column label="操作" width="180px">
					<template slot-scope="scope">
						<el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">
						</el-button>
						<el-button type="danger" icon="el-icon-delete" size="mini"
							@click="removeUserById(scope.row.id)"></el-button>
						<el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
							<el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
						</el-tooltip>
					</template>
				</el-table-column>
			</el-table>
			<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
				:current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize"
				layout="total, sizes, prev, pager, next, jumper" :total="total">
			</el-pagination>
		</el-card>
		<el-dialog title="添加用戶" :visible.sync="addDialogVisible" width="40%" @close="addDialogClosed">
			<div class="dialogForm">
				<el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
					<el-form-item label="用戶名" prop="username">
						<el-input v-model="addForm.username"></el-input>
					</el-form-item>
					<el-form-item label="密碼" prop="password">
						<el-input v-model="addForm.password"></el-input>
					</el-form-item>
					<el-form-item label="郵箱" prop="email">
						<el-input v-model="addForm.email"></el-input>
					</el-form-item>
					<el-form-item label="手機" prop="mobile">
						<el-input v-model="addForm.mobile"></el-input>
					</el-form-item>
				</el-form>
			</div>
			<span slot="footer" class="dialog-footer">
				<el-button @click="addDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="addUser">确 定</el-button>
			</span>
		</el-dialog>
		<el-dialog title="修改用戶" :visible.sync="editDialogVisible" width="40%" @close="editDialogClosed">
			<div class="dialogForm">
				<el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
					<el-form-item label="用戶名" prop="username">
						<el-input v-model="editForm.username" disabled></el-input>
					</el-form-item>
					<el-form-item label="郵箱" prop="email">
						<el-input v-model="editForm.email"></el-input>
					</el-form-item>
					<el-form-item label="手機" prop="mobile">
						<el-input v-model="editForm.mobile"></el-input>
					</el-form-item>
				</el-form>
			</div>
			<span slot="footer" class="dialog-footer">
				<el-button @click="editDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="editUserInfo">确 定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
	export default {
		data() {
			var checkEmail = (rule, value, cb) => {
				const regEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/
				if (regEmail.test(value)) {
					return cb()
				}
				cb(new Error('請輸入合法的郵箱!'))
			}
			var checkMobile = (rule, value, cb) => {
				const regMobile = /^(13[0-9]|14[5|7]|15[0|1|2|3|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/
				if (regMobile.test(value)) {
					return cb()
				}
				cb(new Error('請輸入合法的手機號!'))
			}

			return {
				queryInfo: {
					query: '',
					pagenum: 1,
					pagesize: 2
				},
				userList: [],
				total: 0,
				addDialogVisible: false,
				addForm: {
					username: '',
					password: '',
					email: '',
					mobile: ''
				},
				addFormRules: {
					username: [{
							required: true,
							message: '請輸入用戶名！',
							trigger: 'blur'
						},
						{
							min: 3,
							max: 10,
							message: '用戶名長度在 3 到 10 之間！',
							trigger: 'blur'
						}
					],
					password: [{
							required: true,
							message: '請輸入密碼！',
							trigger: 'blur'
						},
						{
							min: 6,
							max: 15,
							message: '密碼長度在 6 到 15 之間！',
							trigger: 'blur'
						}
					],
					email: [{
							required: true,
							message: '請輸入郵箱！',
							trigger: 'blur'
						},
						{
							validator: checkEmail,
							trigger: 'blur'
						}
					],
					mobile: [{
							required: true,
							message: '請輸入手機號！',
							trigger: 'blur'
						},
						{
							validator: checkMobile,
							trigger: 'blur'
						}
					]
				},
				editDialogVisible: false,
				editForm: {},
				editFormRules: {
					email: [{
							required: true,
							message: '請輸入郵箱！',
							trigger: 'blur'
						},
						{
							validator: checkEmail,
							trigger: 'blur'
						}
					],
					mobile: [{
							required: true,
							message: '請輸入手機號！',
							trigger: 'blur'
						},
						{
							validator: checkMobile,
							trigger: 'blur'
						}
					]
				}
			}
		},
		created() {
			this.getUserList()
		},
		methods: {
			async getUserList() {
				const {
					data: res
				} = await this.$http.get('users', {
					params: this.queryInfo
				})
				if (res.meta.status !== 200) return this.$message.error('获取用户列表失败！')
				this.userList = res.data.users
				this.total = res.data.total
			},
			handleSizeChange(newSize) {
				this.queryInfo.pagesize = newSize
				this.getUserList()
			},
			handleCurrentChange(newPage) {
				this.queryInfo.pagenum = newPage
				this.getUserList()
			},
			async userStateChanged(userinfo) {
				const {
					data: res
				} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
				if (res.meta.status !== 200) {
					userinfo.mg_state = !userinfo.mg_state
					return this.$message.error('更新用戶狀態是失敗！')
				}
				this.$message.success('更新用戶狀態成功！')
			},
			addDialogClosed() {
				this.$refs.addFormRef.resetFields()
			},
			addUser() {
				this.$refs.addFormRef.validate(async valid => {
					if (!valid) return
					const {
						data: res
					} = await this.$http.post('users', this.addForm)
					if (res.meta.status !== 201) {
						this.$message.error('添加用戶失敗！')
					}
					this.$message.success('添加用戶成功！')
					this.addDialogVisible = false
				})
			},
			async showEditDialog(id) {
				this.editDialogVisible = true
				const {
					data: res
				} = await this.$http.get('users/' + id)
				if (res.meta.status !== 200) {
					return this.$message.error('查詢失敗！')
				}
				this.editForm = res.data
				return this.$message.success('查詢成功！')
			},
			editDialogClosed() {
				this.$refs.editFormRef.resetFields()
			},
			editUserInfo() {
				this.$refs.editFormRef.validate(async vaild => {
					if (!vaild) return
					const {
						data: res
					} = await this.$http.put('users/' + this.editForm.id, {
						email: this.editForm.email,
						mobile: this.editForm.mobile
					})
					if (res.meta.status !== 200) {
						return this.$message.error('更新用户失败！')
					}
					this.getUserList()
					this.editDialogVisible = false
					this.$message.success('更新用户成功！')
				})
			},
			async removeUserById(id) {
				const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning',
					center: true
				}).catch(() => {
					this.$message({
						type: 'info',
						message: '已取消删除'
					});
				});
				if(confirmResult !== 'confirm') {
					return this.$message.info('已取消删除！')
				}
				const {data:res} = await this.$http.delete('users/' + id)
				if(res.meta.status !== 200) return this.$message.error('删除失败！')
				this.$message.success('删除成功！')
				this.getUserList()
			}
		}
	}
</script>

<style lang="less" scoped>
	.el-breadcrumb {
		margin-bottom: 15px;
		font-size: 12px;
	}

	.el-pagination {
		margin-top: 15px;
	}

	.dialogForm {
		margin-right: 30px;
	}
</style>
