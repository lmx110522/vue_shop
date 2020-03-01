<template>
    <div>
        <!--        面包屑导航区-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!--        卡片区-->
        <el-card class="box-card">
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input v-model="queryInfo.query" placeholder="请输入内容" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>
            <el-table :data="userList" style="width: 100%" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column
                        prop="username"
                        label="姓名"
                        width="180">
                </el-table-column>
                <el-table-column
                        prop="email"
                        label="邮箱"
                        width="180">
                </el-table-column>
                <el-table-column
                        prop="mobile"
                        label="电话">
                </el-table-column>
                <el-table-column
                        prop="role_name"
                        label="角色">
                </el-table-column>
                <el-table-column
                        label="状态">
                    <!--                    作用域插槽-->
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">

                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column
                        label="操作" width="249px">
                    <template slot-scope="scope">
                        <el-button type="primary" size="mini" icon="el-icon-edit"
                                   @click="showEditDialg(scope.row.id)"></el-button>
                        <el-button type="danger" size="mini" icon="el-icon-delete"
                                   @click="removeUserById(scope.row.id)"></el-button>
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" size="mini" icon="el-icon-setting"
                                       @click="shareRole(scope.row)"></el-button>
                        </el-tooltip>

                    </template>
                </el-table-column>
            </el-table>
            <!--            分页区-->
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="queryInfo.pagenum"
                    :page-sizes="[1, 2, 5, 10]"
                    :page-size="queryInfo.pagesize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total">
            </el-pagination>
        </el-card>
        <el-dialog
                title="添加用户"
                :visible.sync="addDialogVisible"
                width="50%"
                @close="addDialogClose()">
            <el-form ref="addFormRef" :model="addUserForm" :rules="addUserRules" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addUserForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addUserForm.password" type="password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addUserForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="addUserForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser()">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog
                title="修改用户"
                :visible.sync="editDialogVisible"
                width="50%"
                @close="editDialogClose()">
            <el-form ref="editFormRef" :model="editUserForm" :rules="addUserRules" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="editUserForm.username"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editUserForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="editUserForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUser()">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog
                title="分配角色"
                :visible.sync="roleDialogVisible"
                width="50%" @close="closeRoleDialog()">
            <div>
                <p>当前用户:{{userInfo.username}}</p>
                <p>当前角色:{{userInfo.role_name}}</p>
                <p>
                    <template>
                        <el-select v-model="selectRoleId" placeholder="请选择新角色">
                            <el-option
                                    v-for="item in roleList"
                                    v-if="item.roleName != userInfo.role_name"
                                    :key="item.id"
                                    :label="item.roleName"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </template>
                </p>
            </div>
            <span slot="footer" class="dialog-footer">
    <el-button @click="roleDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="saveRole(userInfo.id)">分 配</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Users",
        data() {
            var checkPhone = (rule, value, callback) => {
                const phoneReg = /^1[3|4|5|7|8][0-9]{9}$/
                if (!value) {
                    return callback(new Error('电话号码不能为空'))
                }


                setTimeout(() => {
                    // Number.isInteger是es6验证数字是否为整数的方法,但是我实际用的时候输入的数字总是识别成字符串
                    // 所以我就在前面加了一个+实现隐式转换

                    if (!Number.isInteger(+value)) {
                        callback(new Error('请输入数字值'))
                    } else {
                        if (phoneReg.test(value)) {
                            callback()
                        } else {
                            callback(new Error('电话号码格式不正确'))
                        }
                    }
                }, 100)
            }
            var checkEmail = (rule, value, callback) => {
                const mailReg = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/
                if (!value) {
                    return callback(new Error('邮箱不能为空'))
                }
                setTimeout(() => {
                    if (mailReg.test(value)) {
                        callback()
                    } else {
                        callback(new Error('请输入正确的邮箱格式'))
                    }
                }, 100)
            }
            return {
                //获取用户列表的对象
                queryInfo: {
                    query: "",
                    pagenum: 1,
                    pagesize: 2
                },
                userList: [],
                total: 0,
                addDialogVisible: false,
                editDialogVisible: false,
                addUserForm: {
                    username: "",
                    password: "",
                    email: "",
                    mobile: ""
                },
                editUserForm: {
                    username: "",
                    email: "",
                    mobile: ""
                },
                addUserRules: {
                    username: [
                        {required: true, message: '请输入活动名称', trigger: 'blur'},
                        {min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur'}
                    ],
                    password: [
                        {required: true, message: '请输入密码', trigger: 'blur'},
                        {min: 3, max: 10, message: '长度在 6 到 15 个字符', trigger: 'blur'}
                    ],
                    phone: [
                        {required: true, message: '请输入电话号码', trigger: 'blur'},
                        {validator: checkPhone, trigger: 'blur'}
                    ],
                    email: [
                        {required: true, message: '请输入邮箱地址', trigger: 'blur'},
                        {validator: checkEmail, trigger: 'blur'}
                    ]
                },
                roleDialogVisible: false,
                userInfo: [],
                roleList: [],
                selectRoleId: ''
            }
        },
        methods: {
            async getUserList() {
                const {data: res} = await this.$http.get('users', {params: this.queryInfo})
                console.log(res)
                if (res.meta.status !== 200) return this.$message.error("获取用户列表失败！")
                this.userList = res.data.users
                this.total = res.data.total
            },
            //监听pageSize改变的事件
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize
                this.getUserList()
            },
            // 监听页码值改变的事件
            handleCurrentChange(newPageNum) {
                this.queryInfo.pagenum = newPageNum
                this.getUserList()
            },
            // 监听switch开关
            async userStateChange(userInfo) {
                const {data: res} = await this.$http.put("users/" + userInfo.id + "/state/" + userInfo.mg_state)
                if (res.meta.status !== 200) {
                    userInfo.mg_state = !userInfo.mg_state
                    return this.$message.error("更改状态失败")
                }
                this.$message.success("更改状态成功")
            },
            addDialogClose() {
                this.$refs.addFormRef.resetFields()
            },
            editDialogClose() {
                this.$refs.editFormRef.resetFields()
            },
            addUser() {
                this.$refs.addFormRef.validate(async vali => {
                    if (!vali) return this.$message.warning("请规范填写用户信息")
                    const {data: ref} = await this.$http.post("users", this.addUserForm);
                    if (ref.meta.status !== 201) {
                        return this.$message.error(ref.meta.msg)
                    }
                    this.$message.success("新增用户成功")
                    this.addDialogVisible = false
                    this.getUserList()
                })
            },
            editUser() {
                this.$refs.editFormRef.validate(async vali => {
                    if (!vali) return this.$message.warning("请规范修改用户信息")
                    const {data: res} = await this.$http.put("users/" + this.editUserForm.id, {
                        email: this.editUserForm.email, mobile: this.editUserForm.mobile
                    });
                    if (res.meta.status !== 200) {
                        return this.$message.error(res.meta.msg)
                    }
                    this.$message.success("修改用户成功")
                    this.editDialogVisible = false
                    this.getUserList()
                })
            },
            async removeUserById(id) {
                const confirmResult = await this.$confirm('是否删除此用户?', '删除用户', {
                    confirmButtonText: '删除',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch((err => err))
                if (confirmResult === "confirm") {
                    const {data: res} = await this.$http.delete("users/" + id)
                    if (res.meta.status !== 200) {
                        return this.$message({
                            type: 'error',
                            message: res.meta.msg + ',删除失败!'
                        });
                    }
                    this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                    this.getUserList()
                    console.log("当前数量:" + this.userList.length)
                    console.log(this.queryInfo.pagenum)
                    if (this.userList.length <= 1 && this.queryInfo.pagenum > 1) {
                        this.queryInfo.pagenum = this.queryInfo.pagenum - 1
                        this.getUserList()
                    }
                } else {
                    return this.$message({
                        type: 'info',
                        message: '取消删除成功!'
                    });
                }
            },
            async showEditDialg(id) {
                const {data: res} = await this.$http.get("users/" + id);
                if (res.meta.status !== 200) return this.$message.error("未找到对应的用户")
                console.log(res)
                this.editUserForm = res.data
                this.editDialogVisible = true
            },
            async shareRole(item) {
                this.userInfo = item
                const {data: res} = await this.$http.get('roles')
                if (res.meta.status !== 200) return this.$message.error("获取角列表失败")
                console.log(res.data)
                this.roleList = res.data
                this.roleDialogVisible = true
            },
            async saveRole(id) {
                const {data: res} = await this.$http.put(`users/${id}/role`, {'rid': this.selectRoleId})
                console.log(res)
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
                this.$message.success("分配角色成功")
                this.getUserList()
                this.roleDialogVisible = false
            },
            closeRoleDialog() {
                this.selectRoleId = ''
                this.userInfo = ''
            }
        },
        created() {
            this.getUserList()
        }
    }
</script>

<style lang="less" scoped>

</style>
