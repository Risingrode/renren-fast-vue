<template>
    <div>
        <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽">
        </el-switch>
        <el-button @click="banchSave" v-if="draggable">批量保存</el-button>
        <el-button @click="banchDelete" type="danger">批量删除</el-button>
        <el-tree :data="data" :props="defaultProps" :expand-on-click-node="false" show-checkbox :node-key="catId"
            :default-expended-keys="expandedKey" draggable="true" :allow-drop="allowDrop" @node-drop="handleDrop"
            :draggable="draggable" ref="menuTree"><span class="custom-tree-node" slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                    <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">Append</el-button>
                    <el-button type="text" size="mini" @click="() => edit(data)">edit</el-button>
                    <el-button v-if="node.childNodes.length == 0" type="text" size="mini"
                        @click="() => remove(node, data)">Delete</el-button>
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
                <el-button type="primary" @click="submitData">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
    //import引入的组件需要注入到对象中才能使用
    components: {},
    props: {},
    data() {
        return {
            pCid: [],
            draggable: false,
            updateNodes: [],
            maxLevel: 0,
            title: "",
            dialogType: "", //edit,add
            category: {
                name: "",
                parentCid: 0,
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                productUnit: "",
                icon: "",
                catId: null
            },
            dialogVisible: false,
            menus: [],
            expandedKey: [],
            defaultProps: {
                children: "children",
                label: "name"
            }
        };
    },

    //计算属性 类似于data概念
    computed: {},
    //监控data中的数据变化
    watch: {},
    //方法集合
    methods: {
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
            this.$confirm(`是否批量删除【${catIds}】菜单?`, "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            })
                .then(() => {
                    this.$http({
                        url: this.$http.adornUrl("/product/category/delete"),
                        method: "post",
                        data: this.$http.adornData(catIds, false)
                    }).then(({ data }) => {
                        this.$message({
                            message: "菜单批量删除成功",
                            type: "success"
                        });
                        this.getMenus();
                    });
                })
                .catch(() => { });
        },
        batchSave() {
            this.$http({
                url: this.$http.adornUrl("/product/category/update/sort"),
                method: "post",
                data: this.$http.adornData(this.updateNodes, false)
            }).then(({ data }) => {
                this.$message({
                    message: "菜单顺序等修改成功",
                    type: "success"
                });
                //刷新出新的菜单
                this.getMenus();
                //设置需要默认展开的菜单
                this.expandedKey = this.pCid;
                this.updateNodes = [];
                this.maxLevel = 0;
                // this.pCid = 0;
            });
        },
        // handleNodeClick(data) {
        //     console.log(data);
        // },
        // 参数说明: draggingNode: 当前拖拽的节点  dropNode: 当前放置的目标节点  dropType: 放置的位置
        handleDrop(draggingNode, dropNode, dropType, ev) {
            console.log("handleDrop: ", draggingNode, dropNode, dropType);
            //1、当前节点最新的父节点id
            let pCid = 0;
            let siblings = null;
            if (dropType == "before" || dropType == "after") {
                pCid =
                    dropNode.parent.data.catId == undefined
                        ? 0
                        : dropNode.parent.data.catId;
                siblings = dropNode.parent.childNodes;
            } else {
                pCid = dropNode.data.catId;
                siblings = dropNode.childNodes;
            }
            this.pCid.push(pCid);

            //2、当前拖拽节点的最新顺序，
            for (let i = 0; i < siblings.length; i++) {
                if (siblings[i].data.catId == draggingNode.data.catId) {
                    //如果遍历的是当前正在拖拽的节点
                    let catLevel = draggingNode.level;
                    if (siblings[i].level != draggingNode.level) {
                        //当前节点的层级发生变化
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
        updateChildNodeLevel(node) {
            if (node.childNodes.length > 0) {
                for (let i = 0; i < node.childNodes.length; i++) {
                    var cNode = node.childNodes[i].data;
                    this.updateNodes.push({ catId: cNode.catId, catLevel: node.childNodes[i].level });

                }
                this.updateChildrenLevel(node.childNodes[i]);
            }
        },
        edit(data) {
            console.log("要修改的数据", data);
            this.dialogType = "edit";
            this.title = "修改分类";
            this.dialogVisible = true;

            this.category.catId = data.catId;


            //发送请求获取当前节点最新的数据
            this.$http({
                url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
                method: "get"
            }).then(({ data }) => {
                //请求成功
                console.log("要回显的数据", data);
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
            var { catId, name, icon, productUnit } = this.category;
            // console.log("修改的数据是：", this.category);
            this.$http({
                url: this.$http.adornUrl("/product/category/update"),
                method: "post",
                data: this.$http.adornData({ catId, name, icon, productUnit }, false)
            }).then(({ data }) => {
                this.$message({
                    message: "菜单修改成功",
                    type: "success"
                });
                //关闭对话框
                this.dialogVisible = false;
                //刷新出新的菜单
                this.getMenus();
                //设置需要默认展开的菜单
                this.expandedKey = [this.category.parentCid];
            });
        },
        //添加三级分类
        addCategory() {
            console.log("提交的三级分类数据", this.category);
            this.$http({
                url: this.$http.adornUrl("/product/category/save"),
                method: "post",
                data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
                this.$message({
                    message: "菜单保存成功",
                    type: "success"
                });
                //关闭对话框
                this.dialogVisible = false;
                //刷新出新的菜单
                this.getMenus();
                //设置需要默认展开的菜单
                this.expandedKey = [this.category.parentCid];
            });
        },

        remove(node, data) {
            var ids = [data.catId];
            this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            })
                .then(() => {
                    this.$http({
                        url: this.$http.adornUrl("/product/category/delete"),
                        method: "post",
                        data: this.$http.adornData(ids, false)
                    }).then(({ data }) => {
                        this.$message({
                            message: "菜单删除成功",
                            type: "success"
                        });
                        //刷新出新的菜单
                        this.getMenus();
                        //设置需要默认展开的菜单
                        this.expandedKey = [node.parent.data.catId];
                    });
                })
                .catch(() => { });

            console.log("remove", node, data);
        }
    },
    //生命周期 - 创建完成（可以访问当前this实例）
    created() {
        this.getMenus();
    },
    //生命周期 - 挂载完成（可以访问DOM元素）
    mounted() { },
    beforeCreate() { }, //生命周期 - 创建之前
    beforeMount() { }, //生命周期 - 挂载之前
    beforeUpdate() { }, //生命周期 - 更新之前
    updated() { }, //生命周期 - 更新之后
    beforeDestroy() { }, //生命周期 - 销毁之前
    destroyed() { }, //生命周期 - 销毁完成
    activated() { } //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>