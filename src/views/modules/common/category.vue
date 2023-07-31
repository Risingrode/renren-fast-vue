<template>
    <div>
        <!-- 点击这个里面的分类，来让attrgroup里面动态刷新内容 -->
        <el-tree :data="menus" :props="defaultProps" :node-key="catId" @node-click="nodeclick" ref="menuTree">
        </el-tree>
    </div>
</template>

<script>
//这里可以导入其他文件(比如：组件，工具 js,第三方插件 js,json文件,图片文件等等)
//例如:import 《组件名称》 from '《组件路径》';

/*
父子组件传值
父组件传递给子组件：子组件通过props接收
子组件传递给父组件：子组件通过$emit触发事件，父组件通过@监听事件
*/

export default {
    //import 引入的组件需要注入到对象中才能使用
    components: {},
    props: {},
    data() {
        return {
            catId: '',
            menus: [],
            expandedKey: [],
            dialogVisible: false,
            defaultProps: {
                children: 'children',
                label: 'name'
            }
        };
    },
    //计算属性 类似于 data 概念
    computed: {},
    //监控 data 中的数据变化
    watch: {},
    //方法集合
    methods: {
        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                if (data && data.code === 0) {
                    this.menus = data.page;
                } else {
                    this.menus = [];
                }
            });
        },
        // 点击分类，触发事件
        // 三个参数分别是：当前点击的节点的数据，节点对象，当前节点的vue实例
        nodeclick(data, node, component) {
            // console.log(data, node, component);
            // 事件名字自定义，可以随便写
            this.$emit('nodeclick', data, node, component);
        }
    },
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