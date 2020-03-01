<template>
    <div>
        <!--        面包屑导航区-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card class="box-card">
            <el-table :data="authList" style="width: 100%" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column
                        prop="authName"
                        label="权限名称">
                </el-table-column>
                <el-table-column
                        prop="path"
                        label="路径">
                </el-table-column>
                <el-table-column
                        label="权限等级">
                    <!--                    作用域插槽-->
                    <template slot-scope="scope">
                        <el-tag type="primary" v-if="scope.row.level == 0">一级</el-tag>
                        <el-tag type="success" v-if="scope.row.level == 1">二级</el-tag>
                        <el-tag type="warning" v-if="scope.row.level == 2">三级</el-tag>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
    </div>

</template>

<script>
    export default {
        name: "Rights",
        data() {
            return {
                authList: []
            }
        },
        created() {
            this.getAuthList()
        },
        methods: {
            async getAuthList() {
                const {data: res} = await this.$http.get('rights/list')
                if (res.meta.status !== 200) return this.$message.error("获取权限列表失败")
                console.log(res.data)
                this.authList = res.data
            }
        }
    }
</script>

<style lang="less" scoped>

</style>
