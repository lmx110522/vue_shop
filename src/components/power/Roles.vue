<template>
    <div>
        <!--        面包屑导航区-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card class="box-card">
            <el-row>
                <el-col :span="6">
                    <el-button type="primary" size="medium">添加角色</el-button>
                </el-col>
            </el-row>
            <el-table
                    :data="roleList"
                    style="width: 100%" border stripe>
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bd-bottom',index1 == 0?'bd-top':'',{'auth-one':true}]"
                                v-for="(item1,index1) in scope.row.children" :key="item1.id">
                            <!--渲染一级权限-->
                            <el-col :span="5">
                                <el-tag closable @close="deleteRightById(scope.row,item1.id)">{{item1.authName}}
                                </el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级三级权限-->
                            <el-col :span="19">
                                <el-row :class="[index2 == 0? '':'bd-top','auth-one']"
                                        v-for="(item2,index2) in item1.children"
                                        :key="item2.id">
                                    <!-- 渲染二级权限-->
                                    <el-col :span="6">
                                        <el-tag closable @close="deleteRightById(scope.row,item2.id)" type="success">
                                            {{item2.authName}}
                                        </el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <!-- 渲染三级权限-->
                                    <el-col :span="18">
                                        <el-tag closable v-for="(item3,index3) in item2.children" type="warning"
                                                :key="item3.id" @close="deleteRightById(scope.row,item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index">
                </el-table-column>
                <el-table-column
                        label="角色名称"
                        prop="roleName">
                </el-table-column>
                <el-table-column
                        label="角色描述"
                        prop="roleDesc">
                </el-table-column>
                <el-table-column
                        label="操作"
                        prop="desc">
                    <template slot-scope="scope">
                        <el-button size="mini" type="primary" icon="el-icon-edit" @click="editRole(scope.row.id)">编辑
                        </el-button>
                        <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除
                        </el-button>
                        <el-button size="mini" type="warning" icon="el-icon-setting" @click="shareRole(scope.row)">
                            分配权限
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <el-dialog
                title="分配权限"
                :visible.sync="shareDialogVisible"
                @close="closeAuthDialog"
                width="50%">
            <el-tree :data="authList" :props="authProps" show-checkbox node-key="id"
                     :default-expand-all="true" :default-checked-keys="defKeys"
                     ref="tree">
            </el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="shareDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="getCheckedKeys()">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Roles",
        data() {
            return {
                roleList: [],
                shareDialogVisible: false,
                authList: [],
                authProps: {
                    children: 'children',
                    label: 'authName'
                },
                defKeys: [],
                shareId: ''
            }
        },
        methods: {
            editRole(id) {

            },
            deleteRole(id) {

            },
            async getRoleList() {
                const {data: res} = await this.$http.get("roles")
                if (res.meta.status !== 200) return this.$message.error("获取角色列表失败")
                console.log(res.data)
                this.roleList = res.data
            },
            async deleteRightById(item, id) {
                const confirmResult = await this.$confirm('确认删除此项权限?', '删除权限', {
                    confirmButtonText: '删除',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err)
                if (confirmResult === 'confirm') {
                    const {data: res} = await this.$http.delete(`roles/${item.id}/rights/${id}`)
                    if (res.meta.status !== 200) return this.$message.error("删除权限失败")
                    this.$message.success("成功删除权限")
                    item.children = res.data
                } else {
                    this.$message.info("成功取消删除")
                }
            },
            async shareRole(node) {
                this.shareId = node.id
                const {data: res} = await this.$http.get('rights/tree')
                if (res.meta.status !== 200) return this.$message.error("获取权限失败")
                this.authList = res.data;
                this.getLeafKeys(node, this.defKeys)
                this.shareDialogVisible = true
            },
            getLeafKeys(node, arr) {
                if (!node.children) {
                    return arr.push(node.id)
                }
                node.children.forEach(item => this.getLeafKeys(item, arr))
            },
            closeAuthDialog() {
                this.defKeys = []
                this.shareId = ''
            },
            async getCheckedKeys() {
                const checkKeys = this.$refs.tree.getCheckedKeys();
                const checkStr = checkKeys.join(',');
                const {data: res} = await this.$http.post(`roles/${this.shareId}/rights`, {"rids": checkStr})
                console.log(res)
                if (res.meta.status !== 200) return this.$message.error('更新权限失败')
                this.getRoleList()
                this.shareDialogVisible = false
            }
        },
        created() {
            this.getRoleList()
        }
    }
</script>

<style lang="less" scoped>
    .el-tag {
        margin: 7px;
    }

    .auth-one {
        display: flex;
        align-items: center;
    }

    .bd-top {
        border-top: 1px solid #eee;
    }

    .bd-bottom {
        border-bottom: 1px solid #eee;
    }
</style>
