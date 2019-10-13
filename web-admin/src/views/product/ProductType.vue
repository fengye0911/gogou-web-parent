<template>
        <div>
            <el-row>
                <el-col class="col col-left" :span="6">
                    <el-tree :data="producttypes"
                             :props="defaultProps"
                             @node-contextmenu="rightClick"
                             @node-click="handleNodeClick"></el-tree>
                </el-col>
                <el-col class="col col-right" :span="18"></el-col>
            </el-row>
            <div id="perTreeMenu" v-if="tmDisplay" class="tree_menu" :style="{...rightMenu}">
                <ul>
                    <li><i class="el-icon-tickets"></i> 详情</li>
                    <li><i class="el-icon-edit"></i> 编辑</li>
                </ul>
            </div>
            <!--<div v-show="menuVisible">
                <ul id="menu" class="menu">
                    <li class="menu__item" @click="isAddCardPing = true">平级添加</li>
                    <li class="menu__item" @click="addCard">下级添加</li>
                    <li class="menu__item" @click="deleteCard">删除</li>
                </ul>
            </div>-->
        </div>
</template>

<script>
    export default {
        data(){
            return {
                tmDisplay:false,
                menuVisible:false,
                producttypes:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            }
        },
        methods:{
            deleteCard(){

            },
            addCard(){

            },
            rightClick(e,data,node,comp){
                console.log('e:',e,'data',data);
                this.rightMenu = {top:e.pageY+'px',left:e.pageX+'px'};
                this.tmDisplay = true;
                const self = this;
                document.onclick=function(ev){
                    if(ev.target!==document.getElementById('perTreeMenu')){
                        self.tmDisplay = false
                    }
                }
            },
           /* //类型树鼠标右键单击：属性菜单
            rightClick(MouseEvent, object, Node, element) { // 鼠标右击触发事件
                this.menuVisible = true;  // 显示模态窗口，跳出自定义菜单栏
                var menu = document.querySelector('#menu');
                menu.style.left = MouseEvent.clientX - 160 + 'px';
                document.addEventListener('click', this.foo); // 给整个document添加监听鼠标事件，点击任何位置执行foo方法
                menu.style.top = MouseEvent.clientY - 10 + 'px';
                console.log('右键被点击的event:', MouseEvent);
                console.log('右键被点击的object:', object);
                console.log('右键被点击的value:', Node);
                console.log('右键被点击的element:', element);
                console.log('鼠标点击了树形结构图');
            },
            foo() { // 取消鼠标监听事件 菜单栏
                this.menuVisible = false;
                document.removeEventListener('click', this.foo);// 要及时关掉监听，不关掉的是一个坑，不信你试试，虽然前台显示的时候没有啥毛病，加一个alert你就知道了
            },
*/
            //类型树节点单击
            handleNodeClick(data){
                console.log(data);
            },
            //加载类型树
            loadTypeTree(){
                this.$http.get("/product/productType/loadTypeTree")
                    .then(res=>{
                        console.debug(res);
                        this.producttypes=res.data;
                    })
            }
        },
        mounted() {
            this.loadTypeTree();
        }
    }
</script>

<style scoped>
    .col{
        border:1px solid #ccc;
        height: 600px;
        margin-top: 20px;
    }
    .col-left{
        border-right: none;
        overflow-x: hidden;
        overflow-y: scroll;
    }
    .col-left::-webkit-scrollbar {
        display: none;
    }
    .el-tree{
        border:none
    }
    .menu__item {
        display: block;
        line-height: 20px;
        text-align: center;
        margin-top: 10px;
    }
    .menu {
        height: 100px;
        width: 100px;
        position: absolute;
        border-radius: 10px;
        border: 1px solid #999999;
        background-color: #f4f4f4;
    }
    li:hover {
        background-color: #1790ff;
        color: white;
    }

</style>