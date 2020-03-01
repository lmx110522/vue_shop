<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card class="box-card">
            <el-row>
                <el-col :span="8">
                    <el-button type="primary" size="medium" @click="showCartDialog()">添加分类</el-button>
                </el-col>
            </el-row>
            <tree-table :data="cartList" class="tree-table" :columns="columns"
                        :show-index="true" :selection-type="false"
                        :expand-type="false" border>
                <template slot="isEnable" scope="scope">
                    <span class="el-icon-success primary" style="color: lightgreen" v-if="scope.row.cat_deleted"></span>
                    <span class="el-icon-error" style="color: red" v-if="!scope.row.cat_deleted"></span>
                </template>
                <template slot="sort" scope="scope">
                    <el-tag type="primary" size="mini" v-if="scope.row.cat_level == 0">一级</el-tag>
                    <el-tag type="success" size="mini" v-if="scope.row.cat_level == 1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-if="scope.row.cat_level == 2">三级</el-tag>
                </template>
                <template slot="operate" scope="scope">
                    <el-button size="mini" type="primary" icon="el-icon-edit" @click="editCart(scope.row)">编辑
                    </el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteCart(scope.row)">删除
                    </el-button>
                </template>
            </tree-table>
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
                title="添加分类"
                :visible.sync="addCartDialogVisible"
                @close="closeCartForm()"
                width="50%">
            <el-form ref="cartForm" :rules="cartRules" :model="cartForm" label-width="80px">
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="cartForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类">
                    <el-cascader
                            placeholder="试试搜索：指南"
                            :options="options"
                            :props="cartProps"
                            v-model="selectedKeys"
                            @change="parentChange()"
                            change-on-select
                            filterable clearable
                            style="width: 100%">
                    </el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
    <el-button @click="addCartDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCart()">确 定</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Cart",
        data() {
            return {
                cartList: [],
                queryInfo: {
                    type: 3,
                    pagenum: 1,
                    pagesize: 5
                },
                total: 0,
                columns: [
                    {
                        label: '分类名称',
                        prop: 'cat_name',
                        width: '400px',
                    },
                    {
                        label: '是否有效',
                        type: 'template',
                        template: 'isEnable'
                    },
                    {
                        label: '排序',
                        type: 'template',
                        template: 'sort'
                    },
                    {
                        label: '操作',
                        prop: 'likes',
                        minWidth: '200px',
                        type: 'template',
                        template: 'operate',
                    },
                ],
                addCartDialogVisible: false,
                cartForm: {
                    'cat_name': "",
                    'cat_pid': "",
                    'cat_level': ''
                },
                cartRules: {
                    cat_name: [
                        {required: true, message: '请输入分类名称', trigger: 'blur'},
                        {min: 3, max: 5, message: '长度在 3 到 10 个字符', trigger: 'blur'}
                    ],
                },
                options: [],
                cartProps: {
                    value: "cat_id",
                    label: 'cat_name',
                    children: 'children',
                    expandTrigger: 'hover'
                },
                selectedKeys: []
            }
        },
        methods: {
            editCart(cartItem) {

            },
            deleteCart(cartItem) {

            },
            async getCartList() {
                const {data: res} = await this.$http.get('categories', {params: this.queryInfo})
                if (res.meta.status !== 200) return this.$message.error("获取类别失败")
                this.cartList = res.data.result;
                this.total = res.data.total
            },
            //监听pageSize改变的事件
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize
                this.getCartList()
            },
            // 监听页码值改变的事件
            handleCurrentChange(newPageNum) {
                this.queryInfo.pagenum = newPageNum
                this.getCartList()
            },
            async getParentCartList() {
                const {data: res} = await this.$http.get('categories', {params: {type: 2}})
                if (res.meta.status !== 200) return this.$message.error("获取父级类别失败")
                this.options = res.data
            },
            showCartDialog() {
                this.getParentCartList()
                this.addCartDialogVisible = true
            },
            parentChange() {
                console.log(this.selectedKeys)
                const len = this.selectedKeys.length
                if (len > 0) {
                    this.cartForm.cat_level = len
                    this.cartForm.cat_pid = this.selectedKeys[len - 1]
                }
            },
            closeCartForm() {
                this.$refs.cartForm.resetFields()
                this.selectedKeys = []
                this.cartForm.cat_pid = ''
                this.cartForm.cat_level = ''
            },
            addCart() {
                this.$refs.cartForm.validate(async vali => {
                    if (!vali) return this.$message.warning('请按要求填写好再提交')
                    const {data: res} = await this.$http.post('categories', this.cartForm)
                    if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
                    this.$message.success("成功添加分类")
                    this.getCartList()
                    this.addCartDialogVisible = false
                })
            }
        },
        created() {
            this.getCartList()
        }
    }
</script>

<style lang="less" scoped>
    .tree-table {
        margin-top: 15px;
    }
</style>
