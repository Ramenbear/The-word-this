<template>
    <vue-drawer-layout
            ref="drawerLayout">
        <div class="drawer" slot="drawer">
            <div class="text">
                <p class="user">{{name}}</p>
                <ul class="drawerList">
                    <li @click="modifyPassword">
                        <img src="../../assets/password.png" width="15px"/>
                        修改密码
                    </li>
                    <li @click="quit">
                        <img src="../../assets/exit.png" width="20px"/>
                        安全退出
                    </li>
                </ul>
            </div>
            <a href="javascript:void(0)" class="close"
               @click="handleToggleDrawer">
                <img src="../../assets/return.png" width="18px"/>&nbsp;返回
            </a>
        </div>
        <div class="content" slot="content" ref="viewBox">
            <Header :title="title"></Header>
            <div class="section">
                <ul class="list" v-for="(item,index) in wordAttr">
                    <li v-for=" (word,index) in item">
                        <div>
                            <div>
                                <span class="words">{{word.word}}</span>
                                <span class="syllable">[{{word.pronounce}}]</span></div>
                            <p class="chinese">{{word.description}}</p>
                        </div>
                        <div class="btngroud">
                            <button v-on:click="del(word.id)" class="delbtn">删除</button>
                            <button v-on:click="update(word)" class="modifybtn">修改</button>
                        </div>
                    </li>
                </ul>
            </div>
            <a href="javascript:void(0)" class="btn"
               @click="handleToggleDrawer">
                <img src="../../assets/menu.png" width="20px"/>
            </a>
        </div>
    </vue-drawer-layout>
</template>

<script>
    import {setCookie, getCookie, delCookie} from '../../assets/js/cookie.js'
    import Header from "../header/header"

    export default {
        name: 'Words',
        components: {
            Header
        },
        data() {
            return {
                updWord: {
                    id: 0,
                    word: '',
                    pronounce: '',
                    description: ''
                },
                words: [],
                count: 0,
                page: 1,
                num: null,
                name: '',
                title: "我的单词本", //导航标题内容
                dom: '',
                wordAttr: [] //将获取到的单词列表存到这个数组
            }

        },
        mounted() {
            let uname = getCookie('userName')
            this.name = uname
            if (uname == "") {
                this.$router.push('/')
            }
            let _this = this;
            // 设置一个开关来避免重负请求数据
            let sw = true;
            this.dom = this.$refs.viewBox


            // 注册scroll事件并监听，页面是否滚动到底部
            this.dom.addEventListener('scroll', function () {
                // console.log(document.documentElement.clientHeight + '-----------' + window.innerHeight); // 可视区域高度
                if (document.body.scrollTop + window.innerHeight >= document.body.offsetHeight) {
                    // 如果开关打开则加载数据
                    if (sw == true) {
                        // 将开关关闭
                        sw = false;
                        _this.page++
                        _this.show(_this.page)
                    }
                }
            });
        },
        methods: {
            /**
             * 修改密码的方法
             *
             **/
            modifyPassword: function () {
                this.$router.push({path: "/UpdatePassword"})
            },

            /**
             * 弹出左侧菜单的方法
             *
             **/
            handleToggleDrawer: function () {
                this.$refs.drawerLayout.toggle();
            },

            /**
             * 退出当前程序
             *
             **/
            quit() {
                this.$http.jsonp('http://app.sencha.com.cn/soya/apps/testdb/server/?action=user.logout')
                delCookie('userName')
                this.$router.push('/')
            },
            /**
             * 获取数据
             *
             **/
            show: function (page) {
                console.log(page);
                let _this = this;
                this.$http.jsonp("http://app.sencha.com.cn/soya/apps/testdb/server/?action=wordList.list&perPage=10&page=" + page,
                    {//请求参数

                    }).then(function (res) {
                    _this.words = res.data.data;
                    _this.count = res.data.count;

                    _this.wordAttr.push(_this.words);

                }, function (error) {
                    //console.log(1)
                    console.log(error);
                });
            },
            /**
             * 删除当前的单词信息
             *
             **/
            del: function (id) {
                console.log(id);
                this.$http.jsonp("http://app.sencha.com.cn/soya/apps/testdb/server/?action=wordList.delete&id=" + id,
                    {
                        //请求参数

                    }).then(function (res) {

                    if (res.data.ret == 0) {
                        alert("删除成功！");
                    } else {
                        alert("删除失败!");
                    }

                    this.show(1);

                }, function (error) {
                    //console.log(1)
                    console.log(error);
                });
            },
            /**
             * 修改单词方法
             *
             **/
            update: function (word) {

                console.log(word);

                this.$router.push({
                    /*由于动态路由也是传递params的，所以在 this.$router.push() 方法中
                     * path不能和params一起使用，否则params将无效。需要用name来指定页面。
                     * */
                    name: 'UpdWord',
                    params: {
                        word: word,
                    }
                });
            },

        },
        created() {
            this.show(1); //默认显示第一页的内容
        }
    }
</script>

<style scoped>
    .user {
        height: 100px;
        line-height: 100px;
        font-size: 20px;
        font-weight: bold;
    }

    .section {
        margin-top: 55px;
        margin-bottom: 40px;
    }

    .list li {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        background: #ffffff;
        border-radius: 3px;
        margin: 0 10px;
        padding: 12px;
        text-align: left;
        border: 1px #eee solid;
        margin-top: 12px;

    }

    .content {
        height: 100%;
        overflow: auto;
    }

    .list li .words {
        font-size: 16px;
        font-weight: bold;
        margin-right: 8px;
    }

    .list li .syllable {
        font-size: 12px;
        color: #666;
    }

    .list li .chinese {
        font-size: 12px;
        color: #666;
    }

    .list li .delbtn {
        border-radius: 20px;
        width: 48px;
        height: 19px;
        line-height: 16px;
        color: #41b883;
        border: 1px #41b883 solid;
        font-size: 10px;
        text-align: center;
        display: inline-block;
    }

    .list li .modifybtn {
        border-radius: 20px;
        width: 48px;
        height: 19px;
        line-height: 16px;
        color: #ffffff;
        border: 1px #41b883 solid;
        background: #41b883;
        font-size: 10px;
        text-align: center;
        display: inline-block;
    }

    .btn {
        position: fixed;
        left: 10px;
        top: 0;
        top: 12px;
        z-index: 2;
    }

    .btngroud {
        flex: 0 0 100px;
    }

    .drawerList li {
        display: flex;
        align-items: center;
        margin-left: 20px;
        line-height: 45px;
    }

    .drawerList li img {
        margin-right: 10px;
    }

    button {
        background: none;
    }

    .close {
        position: fixed;
        bottom: 10px;
        right: 35%;
        color: #41b883;
        text-decoration: none;
        font-size: 14px;
        align-items: center;
        display: flex;
        font-weight: bold;

    }

    h1, h2 {
        font-weight: normal;
        text-align: center;
    }

    ul li {
        list-style: none;
    }

    .drawer {
        height: 100%;
    }

    .text {
        width: 70%;
        height: 100%;
        background-color: #ffffff;
        box-shadow: 3px 3px 6px 6px #888888;
        font-family: 微软雅黑;
        font-size: 2.1vh;
    }

    .button {

        display: flex;
        flex-direction: row;

        padding-left: 50%;
        padding-top: 2%;
    }


</style>
