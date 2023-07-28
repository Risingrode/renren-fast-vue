<template>
    <div>
        <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
            show-checkbox :node-key="catId" :default-expended-keys="expandedKey"><span class="custom-tree-node"
                slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                    <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
                        Append
                    </el-button>
                    <el-button v-if="data.children.length == 0" type="text" size="mini" @click="() => remove(node, data)">
                        Delete
                    </el-button>
                    <el-button type="text" size="mini" @click="() => edit(data)">
                        Edit
                    </el-button>
                </span>
            </span>
        </el-tree>
        <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
            <el-form :model="category">
                <el-form-item label="分类名称">
                    <el-input v-model="category.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="图标">
                    <el-input v-model="category.icon" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="计量单位">
                    <el-input v-model="category.productUnit" autocomplete="off"></el-input>
                </el-form-item>
                <!-- <el-form-item label="活动区域">
                    <el-select v-model="category.region" placeholder="请选择活动区域">
                        <el-option label="区域一" value="shanghai"></el-option>
                        <el-option label="区域二" value="beijing"></el-option>
                    </el-select>
                </el-form-item> -->
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCategory">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
    //import 引入的组件需要注入到对象中才能使用
    components: {},
    props: {},
    data() {
        return {
            title: "",
            data: [],
            //修改还是添加
            dialogType: "",
            // 默认展开的菜单id
            expandedKey: [],
            //表单数据
            category: {
                name: "",
                parentCid: 0,
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                catId: null,
                productUnit: "",
                icon: ""
            },
            dialogVisible: false,
            defaultProps: {
                children: 'children',
                label: 'name'
            }
        };
    },
    methods: {
        // handleNodeClick(data) {
        //     console.log(data);
        // },
        edit(data) {
            console.log("要修改的数据");
            this.dialogType = "edit";
            this.dialogVisible = true;
            //发送请求，获取当前节点最新数据
            this.$http({
                url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
                method: 'get',
            }).then(({ data }) => {
                //请求成功
                this.category.name = data.data.name;
                this.category.catId = data.data.catId;
                this.category.icon = data.data.icon;
                this.category.productUnit = data.data.productUnit;
                this.category.parentCid = data.data.parentCid;
            })
        },
        addCategory() {
            console.log("提交的数据是:")
            this.$http({
                url: this.$http.adornUrl('/product/category/save'),
                method: 'post',
                data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
                this.$message({
                    message: '菜单保存成功',
                    type: 'success'
                });
                //关闭对话框
                this.dialogVisible = false;
                this.getMenus();
                this.expandedKey = [this.category.parentCid];
            });
        },
        // 判断是添加还是修改
        submitData() {
            if (this.dialogType == "add") {
                this.addCategory();
            }
            if (this.dialogType == "edit") {
                this.editCategory();
            }
        },
        //修改三级分类数据
        editCategory() {
            this.title = "修改分类";
            var { catId, name, icon, productUnit } = this.category;
            // 前面的key要按照java里面的名字来封装
            // 如果名字都一样，那就直接传
            // var data = { catId: catId, name: name, icon: icon, productUnit: productUnit };
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData({ catId, name, icon, productUnit }, false)
            }).then(({ data }) => {
                this.$message({
                    message: '菜单修改成功',
                    type: 'success'
                });
                this.dialogVisible = false;
                this.getMenus();
                // 设置需要默认展开的菜单
                this.expandedKey = [this.category.parentCid];
            });

        },
        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                if (data && data.code === 0) {

                    this.data = data.page;
                } else {

                    this.data = [];
                }
            });
        },
        append(data) {
            console.log("append", data);
            this.title = "添加分类";
            this.dialogType = "add";
            this.dialogVisible = true;
            this.category.parentCid = data.catId;
            this.category.catLevel = data.catLevel * 1 + 1;

            //清空数组
            this.category.catId = null;
            this.category.icon = "";
            this.category.sort = 0;
            this.category.name = "";
            this.category.showStatus = 1;
            this.category.productUnit = "";
            this.category.parentUnit = "";
        },

        remove(node, data) {
            this.$confirm(`是否删除【${data.name}】菜单?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                var ids = [data.catId]
                this.$http({
                    url: this.$http.adornUrl('/product/category/delete'),
                    method: 'post',
                    data: this.$http.adornData(ids, false)
                }).then(({ data }) => {
                    this.$message({
                        message: '菜单删除成功',
                        type: 'success'
                    });
                    //重新刷新一下
                    this.getMenus()
                    // 设置需要默认展开的菜单
                    this.expandedKey = [node.parent.data.catId]
                });
            }).cache(() => {

            })
        },
    },

    //计算属性 类似于 data 概念
    computed: {},
    //监控 data 中的数据变化
    watch: {},
    //方法集合

    //生命周期 - 创建完成（可以访问当前 this 实例）
    created() {
        this.getMenus();
    },
    //生命周期 - 挂载完成（可以访问 DOM 元素）
    mounted() {

    },
    beforeCreate() { }, //生命周期 - 创建之前
    beforeMount() { }, //生命周期 - 挂载之前
    beforeUpdate() { }, //生命周期 - 更新之前
    updated() { }, //生命周期 - 更新之后
    beforeDestroy() { }, //生命周期 - 销毁之前
    destroyed() { }, //生命周期 - 销毁完成
    activated() { }, //如果页面有 keep-alive 缓存功能，这个函数会触发
}
</script>
<style lang='scss' scoped>
//@import url(); 引入公共 css 类
</style>