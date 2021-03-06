<template>
<div>
    <div class="container m-2">
        <div class="row">
            <div class="col-md-12">
                <banner :user="user"></banner>

                <hr>
            </div>
            <div class="col-md-6 offset-md-1">
                <div v-if="hasPosts">
                    <post v-for="post in posts"
                        :key="post.ID"
                        :content="post.Content"
                        :id="post.ID"
                        :pinned="post.Pinned"
                        :username="post.Username"
                        :isAdmin="isAdmin"
                        :profilePicUrl="post.ProfilePic"
                        @getPosts="getPosts()"
                    ></post>
                </div>

                <div v-else-if="loadingPosts">
                    Loading posts...
                </div>

                <div v-else>
                    <h1>AHHHH</h1>
                    <p>No posts found.</p>
                </div>
            </div>
            <div class ="col-md-4 offset-md-1 border-left">
                <div class="text-center">
                    <h3>Create new post</h3>
                    <textarea v-model="newPost"
                        class="w-100"
                        placeholder="Add new post."
                    ></textarea>

                    <button class="btn btn-primary"
                        @click.prevent="createPost()"
                    >Create post</button>

                    <hr>
                </div>

                <div v-if="isAdmin">
                    <h4>Admin only!</h4>
                    <label for="fname">Search for any user...</label><br>

                    <input type="text"
                        name="fname"
                        class="mt-2 mb-2 w-100"
                        placeholder="First name..."
                        v-model="userSearchFirstName"
                    ><br>

                    <button class="btn btn-primary"
                        @click.prevent="searchUser()"
                    >
                        Search
                    </button>
                    <font-awesome-icon v-if="isLoadingUserSearch"
                        icon="spinner" spin/>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import axios from 'axios'
import apiMixin from '../mixins/api'
import Swal from 'sweetalert2'

export default {
    name: 'Dashboard',
    props: {
        user: Object,
    },

    mixins: [
        apiMixin
    ],

    data: function() {
        return {
            posts: null,
            loadingPosts: false,
            creatingPost: false,
            newPost: null,
            userSearchFirstName: null,
            isLoadingUserSearch: false,
        }
    },

    mounted()
    {
        this.getPosts()
    },

    computed:
    {
        hasPosts()
        {
            return this.posts !== null && this.posts.length > 0
        },

        isAdmin()
        {
            return (this.user && this.user.admin)
        },
    },

    methods:
    {
        getPosts()
        {
            const self = this
            self.loadingPosts = true

            const path = self.getPath(`posts`)

            return axios.get(path)
            .then((res) => {
                self.posts = res.data.posts
            })
            .catch((err) => {
                console.error('Error getting posts', err)
            })
            .finally(() => {
                self.loadingPosts = false
            })
        },

        createPost()
        {
            const self = this
            self.creatingPost = true

            const path = self.getPath('posts')
            const params = {
                UserId: self.user.userId,
                Content: self.newPost
            }

            return axios.post(path, params)
            .then(() => {
                Swal.fire({
                    icon: 'success',
                    title: 'W00t!',
                    text: 'Successfully created new post.'
                })

                setTimeout(() => {
                    self.getPosts()
                }, 1500)
            })
            .catch((err) => {
                Swal.fire({
                    icon: 'error',
                    title: 'Hold up!',
                    text: 'There was a problem. Are you an admin?'
                })

                console.error(err)
            })
            .finally(() => {
                self.creatingPost = false
                self.newPost = null
            })
        },

        searchUser()
        {
            const self = this
            this.isLoadingUserSearch = true

            const FirstName = this.userSearchFirstName
            const path = this.getPath('users/search')
            const params = {
                user: FirstName,
            }

            return axios.post(path, params)
            .then((res) => {
                // code runs when no error
                console.log('response', res)
            })
            .catch((err) => {
                // ahh there's been an error!
                console.error('ahhh error', err)
            })
            .finally(() => {
                // code always runs, regardless of error
                self.isLoadingUserSearch = false
            })
        },

        isLastPost(id)
        {
            return this.hasPosts && (id === this.posts.length - 1)
        },
    }
}
</script>

<style>

</style>
