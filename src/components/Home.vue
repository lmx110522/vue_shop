<template>
    <el-container class="home_container">
        <!--        头部区-->
        <el-header>
            <div>
                <img src="http://qiniuyun.donghao.club/nyist.png" alt="南工电商管理后台">
                <span>南工电商管理后台</span>
            </div>
            <el-button type="info" @click="logout">退出登录</el-button>
        </el-header>
        <!--        主体-->
        <el-container>
            <!--            侧边栏-->
            <el-aside :width="isCollapse? '64px':'200px'">
                <div class="toggle-button" @click="toggleCollapse()">|||</div>
                <el-menu background-color="#383C66" text-color="#fff" active-text-color="#2B95FF"
                         :unique-opened="true" :collapse="isCollapse" :collapse-transition="false"
                         :router="true" :default-active="activePath">
                    <!--                    一级菜单-->
                    <el-submenu :index="menu.id+''" :key="menu.id" v-for="(menu,index) in menuList">
                        <template slot="title">
                            <i :class="iconList[index]"></i>
                            <span class="menu_word">{{menu.authName}}</span>
                        </template>
                        <!--                        二级菜单-->
                        <el-menu-item :index="'/'+children.path+''" v-for="(children,index) in menu.children"
                                      :key="children.id" @click="saveChooseStatus('/'+children.path+'')">
                            <i class="el-icon-setting"></i>
                            <span slot="title">{{children.authName}}</span>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>
            <!--            主体区-->
            <el-main>
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
    export default {
        name: "Home",
        data() {
            return {
                menuList: [],
                iconList: [
                    "el-icon-user-solid",
                    "el-icon-key",
                    "el-icon-suitcase",
                    "el-icon-notebook-2",
                    "el-icon-s-marketing"],
                isCollapse: false,
                //被激活的链接地址
                activePath: ""
            }
        },
        methods: {
            logout() {
                window.sessionStorage.clear()
                this.$router.push("/login")
            },
            async getMenuList() {
                const {data: res} = await this.$http.get('menus');
                console.log(res)
                if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
                this.menuList = res.data
            },
            toggleCollapse() {
                this.isCollapse = !this.isCollapse
            },
            saveChooseStatus(activePath) {
                this.activePath = activePath
                window.sessionStorage.setItem("activePath", activePath)
            }
        },
        created() {
            this.getMenuList()
            this.activePath = window.sessionStorage.getItem('activePath')
        }
    }
</script>

<style lang="less" scoped>
    .home_container {
        height: 100%;
    }

    .el-header {
        background: #464966;
        display: flex;
        justify-content: space-between;
        padding-left: 0px;
        align-items: center;
        color: #ffffff;
        font-size: 20px;

        div {
            display: flex;
            align-items: center;

            img {
                width: 150px;
                height: 40px;
            }

            span {
                margin-left: 10px;
            }
        }
    }

    .el-aside {
        background: #383C66;

        .menu_word {
            margin-left: 10px;
        }

        .el-menu {
            border: none;
        }

        .toggle-button {
            background: #535cb7e0;
            font-size: 10px;
            line-height: 24px;
            color: #ffffff;
            text-align: center;
            letter-spacing: 0.2em;
            cursor: pointer;
        }
    }

    .el-main {
        background: #EDEFF2;
    }
</style>
