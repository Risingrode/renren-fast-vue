<template>
    <div>
        <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽">
        </el-switch>
        <el-button @click="banchSave" v-if="draggable">批量保存</el-button>
        <el-button @click="banchDelete" type="danger">批量删除</el-button>
        <!-- 解释一下属性 -->
        <el-tree :data="data" :props="defaultProps" :expand-on-click-node="false" show-checkbox :node-key="catId"
            :default-expended-keys="expandedKey" draggable="true" :allow-drop="allowDrop" @node-drop="handleDrop"
            :draggable="draggable" ref="menuTree"><span class="custom-tree-node" slot-scope="{ node, data }">
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
            //父节点id
            pCid: [],
            // 是否开启拖拽
            draggable: false,
            // 主题
            title: "",
            data: [],
            //最大深度
            maxLevel: 0,
            //修改还是添加
            dialogType: "",
            // 默认展开的菜单id
            expandedKey: [],
            // 拖拽节点
            updateNodes: [],
            //表单数据
            category: {
                // 分类名称
                name: "",
                // 父级分类id
                parentCid: 0,
                // 0: 一级分类 1：二级分类 2：三级分类
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                // 产品分类id
                catId: null,
                // 产品单位
                productUnit: "",
                icon: ""
            },
            dialogVisible: false,
            // 树形结构的数据
            defaultProps: {
                children: 'children',
                label: 'name'
            }
        };
    },
    methods: {
        // 批量保存
        banchSave() {
            // 3.发送请求
            // this.$http({
            //     url: this.$http.adornUrl(`/product/category/update/sort`),
            //     method: 'post',
            //     data: this.updateNodes
            // }).then(({ data }) => {
            //     //请求成功
            //     if (data.code == 0) {
            //         this.$message({
            //             message: '修改成功',
            //             type: 'success'
            //         });
            //         this.getMenus();
            //         this.expandedKey = [this.pCId];
            //         this.updateNodes = [];
            //         this.maxLevel = 0;
            //         this.pCId=0;
            //     } else {
            //         this.$message({
            //             message: data.msg,
            //             type: 'error'
            //         });
            //     }
            // }).catch((error) => {
            //     //请求失败
            //     this.$message({
            //         message: '修改失败',
            //         type: 'error'
            //     });
            // });
        },
        // 批量删除
        banchDelete() {
            let checkedNodes = this.$refs.menuTree.getCheckedNodes();
            console.log("被选中的节点:", checkedNodes);
            let catIds = [];
            for (let i = 0; i < checkedNodes.length; i++) {
                catIds.push(checkedNodes[i].catId);
            }
            console.log("被选中的节点id:", catIds);
            // 3.发送请求
            this.$confirm(`是否批量删除【${catIds}】?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/product/category/delete'),
                    method: 'post',
                    data: this.$http.adornData(catIds, false)
                }).then(({ data }) => {
                    this.$message({
                        message: '菜单批量删除成功',
                        type: 'success'
                    });
                    //重新刷新一下
                    this.getMenus()
                });
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });
            });
        },
        // handleNodeClick(data) {
        //     console.log(data);
        // },
        // 拖拽节点
        // 参数说明: draggingNode: 当前拖拽的节点  dropNode: 当前放置的目标节点  dropType: 放置的位置
        handleDrop(draggingNode, dropNode, dropType, ev) {
            console.log("拖拽的父节点");
            console.log(draggingNode, dropNode);
            // 父节点id
            let pCid = 0;
            // 儿子节点
            let siblings = null;
            if (dropType == "before" || dropType == "after") {
                // 当前拖拽的节点的父节点
                pCid = dropNode.parent.data.catId == undefined ? 0 : dropNode.parent.data.catId;
                siblings = dropNode.parent.childNodes;
            } else {
                pCid = dropNode.data.catId;
                siblings = dropNode.childNodes;
            }
            // 获取当前拖拽的节点的父节点id,并且存储起来，用于批量保存的时候方便展开
            this.pCid.push(pCid);
            // 2.获取当前拖拽的节点的排序
            for (let i = 0; i < siblings.length; i++) {
                if (siblings[i].data.catId == draggingNode.data.catId) {
                    // 如果遍历的是当前正在拖拽的节点 还要更新父id的值
                    let catLevel = draggingNode.level;
                    if (siblings[i].level != catLevel) {
                        // 当前节点的层级发生变化
                        catLevel = siblings[i].level;
                        // 修改子节点的层级
                        this.updateChildrenLevel(siblings[i]);
                        this.updateNodes.push({ catId: siblings[i].data.catId, sort: i, parentCid: pCid });
                    } else {
                        this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
                    }
                }
            }
            console.log("需要更新的节点是:", this.updateNodes);
        },
        // 递归遍历修改子节点的层级
        updateChildrenLevel(node) {
            if (node.childNodes.length > 0) {
                for (let i = 0; i < node.childNodes.length; i++) {
                    var cNode = node.childNodes[i].data;
                    this.updateNodes.push({ catId: cNode.catId, catLevel: node.childNodes[i].level });
                }
                this.updateChildrenLevel(node.childNodes[i]);
            }
        },
        // 修改数据
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
        // 获取菜单数据
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
        // 刷新菜单
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
        // 删除
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

        // TODO: 拖动逻辑有点问题
        // 拖动的时候，需要判断当前拖动的节点和父节点的层级总和不能大于3
        allowDrop(draggingNode, dropNode, type) {
            // 被拖动的当前界定啊以及父节点，总层数不能大于3
            console.log("拖动:", draggingNode, dropNode, type);
            this.countNodeLevel(draggingNode);
            console.log(this.maxLevel, "移动情况是:", type);
            //当前正在拖动的节点加上父节点不大于3即可
            let deep = Math.abs(this.maxLevel - draggingNode.data.catLevel + 1);
            console.log("深度:", deep);
            if (type == "inner") {
                return deep + dropNode.level <= 3;
            } else {
                return deep + dropNode.parent.level <= 3;
            }
        },
        countNodeLevel(node) {
            // 找到所有子节点的最大深度
            if (node.children != null && node.childNodes.length > 0) {
                for (let i = 0; i < node.childNodes.length; i++) {
                    if (node.childNodes[i].level > this.maxLevel) {
                        this.maxLevel = node.childNodes[i].level;
                    }
                    this.countNodeLevel(node.childNodes[i]);
                }
            }
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