<script setup>
    import axios from 'axios';
</script>

<script>
export default {
    data() {
        return {
            moods: ["Happy", "Sad", "Angry"],
            posts: [], 
            entry: "",
            mood: "",
            showEditPost: false,
            editPostId: "",
            id: "",
        }
    },
    computed: {
        baseUrl() {
            if (window.location.hostname=='localhost')
                return 'http://localhost:3000' 
            else {
                const codespace_host = window.location.hostname.replace('5173', '3000')
                return `https://${codespace_host}`;
            }
        }
    },
    created() { 
        axios.get(`${this.baseUrl}/posts`)
            .then(response => {
                this.posts = response.data
                console.log(this.posts)
            })
            .catch(error => {
                this.posts = [{ entry: 'There was an error: ' + error.message }]
            })
    },
    methods: {
        editPost(index, postIndex) {
            this.entry = this.posts[index]['entry'];
            this.mood = this.posts[index]['mood'];
            this.id = postIndex;
            this.showEditPost = true;
        },
        updatePost() {
            const params = {
                    entry: this.entry,
                    mood: this.mood,
            }
            axios.post(`${this.baseUrl}/updatePost?id=${this.id}`, params)
            .then(()=>{
                axios.get(`${this.baseUrl}/posts`)
                .then(response2=>{
                    this.posts = response2.data;
                    this.showEditPost = false;
                    this.entry = '';
                    this.mood='';
                    this.id='';
                })
                .catch(error=>{
                    console.log(error);
                })
            })
        }
    }
}
</script>
<template>
    <div id="demo">
        <h2> View Blog Posts </h2>
        <table class="table m-2">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Entry</th>
                    <th>Mood</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(post, index) in posts" :key="post.id">
                    <td>{{ post.id }}</td>
                    <td>{{ post.entry }}</td>
                    <td>{{ post.mood }}</td>
                    <td><button @click="editPost(index, post.id)">Edit</button></td>
                </tr>
            </tbody>
        </table>
        <!-- TODO Show form for editing post only when edit button is clicked -->
        <div id="editPost" v-if="showEditPost">
            <h3>Edit Post</h3>
            <div id="postContent" class="mx-3">
                <form @submit.prevent="updatePost">
                    <div class="mb-3">
                        <label for="entry" class="form-label">Entry</label>
                        <textarea id="entry" class="form-control" v-model="entry" required></textarea>
                    </div>
                    <div class="mb-3">
                        <label for="mood" class="form-label">Mood</label>
                        <select id="mood" class="form-select" v-model="mood" required>
                            <option value="" disabled>Select Mood</option>
                            <option v-for="mood in moods" :value="mood">{{ mood }}</option>
                        </select>
                    </div>
                    <button type="submit" class="btn btn-primary">Update Post</button>
                </form>
            </div>
        </div>
    </div>
</template>
<style scoped></style>
