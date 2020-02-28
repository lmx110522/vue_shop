<template>
    <div class="login_container">
        <div class="login_box">
            <!--            头像区-->
            <div class="avatar_box">
                <img src="http://qiniuyun.donghao.club/00621aa7df2f8237.jpg" alt="头像">
            </div>
            <!--            表单区-->
            <el-form ref="loginFormRef" label-width="0px" :rules="loginRules" :model="loginForm" class="login_form">
                <!--   用户名-->
                <el-form-item prop="username">
                    <el-input v-model="loginForm.username" prefix-icon="el-icon-user-solid"></el-input>
                </el-form-item>
                <!--  密码-->
                <el-form-item prop="password">
                    <el-input v-model="loginForm.password" type="password" prefix-icon="el-icon-lock"></el-input>
                </el-form-item>
                <!--  按钮区-->
                <el-form-item class="button_area">
                    <el-button type="primary" @click="login()">登录</el-button>
                    <el-button type="info" @click="resetLoginForm()">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Login",
        data() {
            return {
                loginForm: {
                    username: 'admin',
                    password: '123456'
                },
                loginRules: {
                    username: [
                        {required: true, message: '请输入用户名', trigger: 'blur'},
                        {min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur'}
                    ],
                    password: {required: true, message: '请输入密码', trigger: 'change'}
                }
            }
        },
        methods: {
            resetLoginForm() {
                // 表单重置
                this.$refs.loginFormRef.resetFields();
            },
            login() {
                // async 函数返回的是一个promise 对象
                this.$refs.loginFormRef.validate(async valid => {
                    // 表单预验证 true false
                    if (!valid) return;
                    const {data: res} = await this.$http.post("login", this.loginForm)
                    console.log(res)
                    if (res.meta.status !== 200) return this.$message.error("登录失败")
                    this.$message({
                        message: "登录成功",
                        type: "success"
                    })
                    window.sessionStorage.setItem("token", res.data.token)
                    await this.$router.push("/home")
                })
            }
        }
    }
</script>

<style lang="less" scoped>
    .login_container {
        height: 100%;
        background: #126376;
    }

    .login_box {
        width: 450px;
        height: 300px;
        background: #fff;
        border-radius: 3px;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);

        .avatar_box {
            height: 130px;
            width: 130px;
            border: 1px solid #eee;
            border-radius: 50%;
            padding: 10px;
            box-shadow: 0 0 10px #ddd;
            position: absolute;
            left: 50%;
            transform: translate(-50%, -50%);
            background: #fff;

            img {
                width: 100%;
                height: 100%;
                border-radius: 50%;
                background: #eee;
            }
        }

        .login_form {
            box-sizing: border-box;
            position: absolute;
            bottom: 0px;
            width: 100%;
            padding: 0px 20px;

            .button_area {
                display: flex;
                justify-content: flex-end;
            }
        }


    }
</style>
