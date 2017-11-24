<template>
    <div>
        <div class="top_bar_wrapper">
            <div class="top_bar">
                <div class="search_box">
                    <form v-on:submit.prevent="search_post">
                        <input v-on:keyup="search_post" v-model="searchKey" type="text" name="search_input"
                               class="search_input" placeholder="Search post">
                        <button type="submit" class="search_trigger">SEARCH</button>
                    </form>
                </div>
                <a class="create_trigger" v-on:click="openPostForm">Create Post</a>
            </div>
            <!--<div class="user_panel">
                <div class="username" v-on:click="updateForm">{{user.fullname}}</div>
                <div class="drop_box">
                    <div class="inner_wrapper">
                        <form v-on:submit.prevent="updateUser">
                            <input v-model="userUpdate.fullname" type="text" class="input_form" name="username">
                            <div class="margin-10"></div>
                            <input v-model="userUpdate.email" type="text" class="input_form" name="email">
                            <button class="action_trigger" type="submit">Save</button>
                        </form>
                    </div>
                </div>
            </div>-->
        </div>
        <div class="post_list_wrapper">
            <div class="post_list" v-if="post_list.length !== 0">
                <div class="section_left">
                    <div class="eachPost animated slideInUp" v-for="(post,index) in post_list"
                         v-if="isEven(index) === true">
                        <div class="dropBox">
                            <div class="dropBox_inner">
                                <div class="icon_trigger"></div>
                                <div class="dropMenu">
                                    <div class="drop_child" v-on:click="openEditor(post.id , index)">Edit Post</div>
                                    <div class="drop_child" v-on:click="callRemover(post.id , index)">Remove Post</div>
                                </div>
                            </div>
                        </div>
                        <div class="post_title">
                            <router-link :to="'/posts/'+post.id">{{post.title}}</router-link>
                        </div>
                        <div class="margin-10"></div>
                        <div class="post_content">{{post.body}}</div>
                    </div>
                </div>
                <div class="section_right">
                    <div class="eachPost animated slideInUp" v-for="(post,index) in post_list"
                         v-if="isEven(index) === false">
                        <div class="dropBox">
                            <div class="dropBox_inner">
                                <div class="icon_trigger"></div>
                                <div class="dropMenu">
                                    <div class="drop_child" v-on:click="openEditor(post.id , index)">Edit Post</div>
                                    <div class="drop_child" v-on:click="callRemover(post.id , index)">Remove Post</div>
                                </div>
                            </div>
                        </div>
                        <div class="post_title">
                            <router-link :to="'/posts/'+post.id">{{post.title}}</router-link>
                        </div>
                        <div class="margin-10"></div>
                        <div class="post_content">{{post.body}}</div>
                    </div>
                </div>
            </div>
            <div class="post_list" v-if="post_list.length === 0">
                <div class="margin-10"></div>
                <div class="margin-10"></div>
                <div class="ground_loader_wrapper">
                    <div class="bg-bar"></div>
                    <div class="bg-bar"></div>
                    <div class="sm-bar"></div>
                    <div class="bg-bar"></div>
                    <div class="alert_msg">Post not found</div>
                    <div class="bg-bar"></div>
                    <div class="sm-bar"></div>
                    <div class="bg-bar"></div>
                    <div class="bg-bar"></div>
                </div>
            </div>
        </div>


        <create_form></create_form>
        <edit_form :editForm="editForm" :dataLoad="dataLoad"></edit_form>
        <div class="delete_confirm_wrapper" :class="{_active: postController.removePost.on === true}">
            <div class="delete_confirm">
                <div class="delete_confirm_inner">
                    <div class="loader_wrapper" :class="{on: postController.removePost.loader === true}">
                        <div class="loader_inner">
                            <div class="sk-cube-grid">
                                <div class="sk-cube sk-cube1"></div>
                                <div class="sk-cube sk-cube2"></div>
                                <div class="sk-cube sk-cube3"></div>
                                <div class="sk-cube sk-cube4"></div>
                                <div class="sk-cube sk-cube5"></div>
                                <div class="sk-cube sk-cube6"></div>
                                <div class="sk-cube sk-cube7"></div>
                                <div class="sk-cube sk-cube8"></div>
                                <div class="sk-cube sk-cube9"></div>
                            </div>
                        </div>
                    </div>
                    <div class="help_slogan">Selected Post will be remove</div>
                    <div class="margin-10"></div>
                    <div class="margin-10"></div>
                    <div class="action_section">
                        <a class="action_trigger" v-on:click="closeRemover">close</a>
                        <a class="action_trigger danger" v-on:click="delete_post()">Remove</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import create_form from './create.vue';
    import edit_form from './edit.vue';

    export default {
        name: 'app',
        data() {
            return {
                post_type: 1,
                searchKey: '',
                editForm: {
                    title: '',
                    body: ''
                },
                dataLoad: true,
                deletePostInfo: {
                    post_id: '',
                    index: '',
                }
            }
        },
        created() {
            this.getPostList()
        },
        computed: {
            postController() {
                return this.$store.getters.postController;
            },
            post_list() {
                return this.$store.getters.post_list;
            },
        },
        components: {create_form, edit_form},
        methods: {
            isEven: function (index) {
                if (index % 2 == 0) {
                    return true;
                }
                else {
                    return false;
                }
            },
            search_post: function (event) {
                this.post_list = null;
                this.post_type = 2;
                if (this.searchKey.length > 0) {
                    this.axios.get('https://jsonplaceholder.typicode.com/posts?title_like=' + this.searchKey).then((response) => {
                        this.$store.commit('post_list', response.data)
                    })
                } else {
                    this.getPostList()
                }
            },
            getPostList: function () {
                this.$store.dispatch('getPostList');
            },
            delete_post: function () {
                this.postController.removePost.loader = true;
                let params = {
                    post_id: this.deletePostInfo.post_id,
                    index: this.deletePostInfo.index,
                };
                console.log(params)
                this.$store.dispatch('removePost', params);
            },
            callRemover: function (post_id, index) {
                this.deletePostInfo.post_id = post_id;
                this.deletePostInfo.index = index;
                this.postController.removePost.on = true;
            },
            openPostForm: function () {
                this.$store.commit('toggleCreateForm', true)
            },
            openEditor: function (post_id, index) {
                this.editForm.title = this.post_list[index].title;
                this.editForm.body = this.post_list[index].body;
                let params = {
                    post_id: post_id,
                    index: index,
                    status: true
                };
                this.$store.commit('toggleEditForm', params)
                let THIS = this;
                setTimeout(function () {
                    THIS.dataLoad = true;
                }, 500);
            },
            closeRemover: function () {
                this.deletePostInfo.post_id = '';
                this.deletePostInfo.index = '';
                this.postController.removePost.on = false;
            }
            /*updateForm: function (event) {
                let trigger = event.currentTarget;
                let targer = trigger.parentElement.querySelector('.drop_box');
                if (targer.classList.contains('active')){
                    targer.classList.remove('active')
                }else {
                    const fullname = this.user.fullname;
                    const email = this.user.email;
                    this.userUpdate.fullname = fullname;
                    this.userUpdate.email = email;
                    targer.classList.add('active')
                }
            },
            updateUser: function (event) {
                this.$store.dispatch('setUser', this.userUpdate);
                let trigger = event.currentTarget;
                let dropBox = trigger.parentElement.parentElement;
                dropBox.classList.remove('active');
                this.userUpdate.fullname = '';
                this.userUpdate.email = '';
            }*/
        }
    }
</script>
<style>
    @import '../assets/css/home/style.css';
    @import "../assets/css/loader/style.css";
</style>
