<style lang="less">
    @import "./main.less";
    .dropa{
        color:#515a6e;
        padding:18px 0;
    }
</style>
<template>
    <div class="main" :class="{'main-hide-text': shrink}">
        <div class="sidebar-menu-con" :style="{width: shrink?'60px':'200px', overflow: shrink ? 'visible' : 'auto'}">
            <scroll-bar ref="scrollBar">
                <shrinkable-menu 
                    :shrink="shrink"
                    @on-change="handleSubmenuChange"
                    :theme="menuTheme" 
                    :before-push="beforePush"
                    :open-names="openedSubmenuArr"
                    :menu-list="menuList">
                    <div slot="top" class="logo-con">
                        <img v-show="!shrink"  src="../images/logo-slide.png" key="max-logo" class="max-logo" />
                        <img v-show="shrink" src="" key="min-logo" />
                    </div>
                </shrinkable-menu>
            </scroll-bar>
        </div>
        <div class="main-header-con" :style="{paddingLeft: shrink?'60px':'200px'}">
            <div class="main-header">
                <div class="navicon-con">
                    <Button :style="{transform: 'rotateZ(' + (this.shrink ? '-90' : '0') + 'deg)'}" type="text" @click="toggleClick">
                        <Icon type="ivu-icon ivu-icon-md-menu" size="32"></Icon>
                        <!-- <Icon custom="_icon88" size="32"></Icon> -->
                        
                    </Button>
                </div>
                 <div class="header-middle-con">
                    <div class="main-breadcrumb">
                        <breadcrumb-nav :currentPath="currentPath"></breadcrumb-nav>
                    </div>
                </div>
                <div class="header-avator-con">
                    <Dropdown transfer @on-click="handleClickLanguage" align="middle" >
                        <a href="javascript:void(0)" class="dropa">
                            <!-- <img src="../images/China.png" style="position: relative;top: 10px" v-show="language== 'zh-CN'">
                            <img src="../images/England.png" style="position: relative;top: 10px" v-show="language=='en-US'"> -->
                            <!-- <Icon type="ivu-icon ivu-icon-md-arrow-dropdown" size="15"></Icon> -->
                            

                            <span v-show="language== 'zh-CN'"><Icon custom="_icon22" size="25"></Icon>&nbsp;简体中文</span>
                            <span v-show="language== 'en-US'"><Icon custom="_icon22" size="25"></Icon>&nbsp;English</span>
                            <Icon type="ivu-icon ivu-icon-md-arrow-dropdown" size="15"></Icon>
                        </a>
                        <DropdownMenu slot="list">
                            <DropdownItem name="China" :disabled="language== 'zh-CN'" :selected="language== 'zh-CN'">
                                <!-- <img src="../images/China.png" style="vertical-align:middle;"> -->
                                <span>简体中文</span>
                            </DropdownItem>
                            <DropdownItem name="England" :disabled="language== 'en-US'" :selected="language== 'en-US'">
                                <!-- <img src="../images/England.png" style="vertical-align:middle;"> -->
                                <span>English</span>
                            </DropdownItem>
                        </DropdownMenu>
                    </Dropdown> 
                    <!-- <full-screen v-model="isFullScreen" @on-change="fullscreenChange"></full-screen> -->
                    <lock-screen></lock-screen>
                    
                    <div class="user-dropdown-menu-con">
                        <Row type="flex" justify="end" align="middle" class="user-dropdown-innercon">
                            <Dropdown transfer trigger="click" @on-click="handleClickUserDropdown">
                                <a href="javascript:void(0)">
                                    <span class="main-user-name">{{ userName }}</span>
                                    <Icon type="ivu-icon ivu-icon-md-arrow-dropdown" size="15"></Icon>
                                </a>
                                <DropdownMenu slot="list">
                                    <DropdownItem name="ownSpace">{{$t('personalCenter')}}</DropdownItem>
                                    <DropdownItem name="logout" divided>{{$t('logout')}}</DropdownItem>
                                </DropdownMenu>
                            </Dropdown>
                            <img class="unlock-avator-img" src="../images/user_icon.png" style="margin-left: 10px;width: 32px;height: 32px;border-radius: 50%;">
                            <!-- <Avatar src="../images/Thailand.png" style="background: #619fe7;margin-left: 10px;"></Avatar> -->
                        </Row>
                    </div>
                </div>
            </div>
            <div class="tags-con">
                <tags-page-opened :pageTagsList="pageTagsList"></tags-page-opened>
            </div>
        </div>
        <div class="single-page-con" :style="{left: shrink?'60px':'200px'}">
            <div class="single-page">
                <keep-alive :include="cachePage">
                    <router-view></router-view>
                </keep-alive>
            </div>
        </div>
    </div>
</template>
<script>
    import Vue from 'vue';
    import shrinkableMenu from './main-components/shrinkable-menu/shrinkable-menu.vue';
    import tagsPageOpened from './main-components/tags-page-opened.vue';
    import breadcrumbNav from './main-components/breadcrumb-nav.vue';
    import fullScreen from './main-components/fullscreen.vue';
    import lockScreen from './main-components/lockscreen/lockscreen.vue';
    import Cookies from 'js-cookie';
    import util from '@/libs/util.js';
    import scrollBar from '@/views/my-components/scroll-bar/vue-scroller-bars';
    
    export default {
        components: {
            shrinkableMenu,
            tagsPageOpened,
            breadcrumbNav,
            fullScreen,
            lockScreen,
            scrollBar
        },
        data () {
            return {
                shrink: false,
                userName: '',
                isFullScreen: false,
                openedSubmenuArr: this.$store.state.app.openedSubmenuArr,
                language:''
            };
        },
        computed: {
            menuList () {
                return this.$store.state.app.menuList;
            },
            pageTagsList () {
                return this.$store.state.app.pageOpenedList; // 打开的页面的页面对象
            },
            currentPath () {
                return this.$store.state.app.currentPath; // 当前面包屑数组
            },
            avatorPath () {
                return localStorage.avatorImgPath;
            },
            cachePage () {
                return this.$store.state.app.cachePage;
            },
            lang () {
                return this.$store.state.app.lang;
            },
            menuTheme () {
                return this.$store.state.app.menuTheme;
            }
        },
        methods: {
            init () {
                let pathArr = util.setCurrentPath(this, this.$route.name);
                this.$store.commit('updateMenulist');
                if (pathArr.length >= 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
                this.userName = Cookies.get('user');
                this.checkTag(this.$route.name);
            },
            toggleClick () {
                this.shrink = !this.shrink;
            },
            handleClickUserDropdown (name) {
                if (name === 'ownSpace') {
                    util.openNewPage(this, 'ownspace_index');
                    this.$router.push({
                        name: 'ownspaceIndex'
                    });
                } else if (name === 'logout') {
                    // 退出登录
                    let url = '/market/portal/user/exit';
                    let ref = this;
                    var params = {};
                    this.$http.post(url, params).then(
                        function(res){
                            if(!!ref && res.resultCode == '0'){
                                ref.$store.commit('logout', ref);
                                ref.$store.commit('clearOpenedSubmenu');
                                ref.$router.push({
                                    name: 'login'
                                });
                            }
                        }
                    );
                }
            },
            checkTag (name) {
                let openpageHasTag = this.pageTagsList.some(item => {
                    if (item.name === name) {
                        return true;
                    }
                });
                if (!openpageHasTag) { //  解决关闭当前标签后再点击回退按钮会退到当前页时没有标签的问题
                    util.openNewPage(this, name, this.$route.params || {}, this.$route.query || {});
                }
            },
            handleSubmenuChange (val) {
                // console.log(val)
            },
            beforePush (name) {
                // if (name === 'accesstest_index') {
                //     return false;
                // } else {
                //     return true;
                // }
                return true;
            },
            fullscreenChange (isFullScreen) {
                // console.log(isFullScreen);
            },
            scrollBarResize () {
                this.$refs.scrollBar.resize();
            },
            handleClickLanguage(name){
                if(name === 'China') {
                    this.changeLanguage('zh-CN');
                }else if (name === 'England') {
                    this.changeLanguage('en-US');
                }else if (name === 'Thailand') {
                    this.changeLanguage('th-TH');
                }
            },
            changeLanguage(flag)
            {
                this.language = flag;
                this.$store.commit('switchLang', flag); // 如果你要自己实现多语言切换，那么只需要执行这行代码即可，修改语言类型
            }
        },
        watch: {
            '$route' (to) {
                this.$store.commit('setCurrentPageName', to.name);
                let pathArr = util.setCurrentPath(this, to.name);
                if (pathArr.length > 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
                this.checkTag(to.name);
                localStorage.currentPageName = to.name;
            },
            lang () {
                util.setCurrentPath(this, this.$route.name); // 在切换语言时用于刷新面包屑
            },
            openedSubmenuArr () {
                setTimeout(() => {
                    this.scrollBarResize();
                }, 300);
            }
        },
        mounted () {
            this.init();
            window.addEventListener('resize', this.scrollBarResize);
        },
        created () {
            // 显示打开的页面的列表
            this.$store.commit('setOpenedList');
            this.language = localStorage.lang || Vue.config.lang || 'th-TH';
        },
        dispatch () {
            window.removeEventListener('resize', this.scrollBarResize);
        }
    };
</script>
