<template>
  <div>
    <div v-if="status">
      <div class="hero is-info">
        <div class="hero-body">
          <div class="container">
            <h1 class="title">
              {{ user.username }}      
            </h1>
            <h2 class="subtitle is-6">
              email: {{ user.email }}
              <br />
              name: {{ user.name }}
              <br />
              #posts: {{ user.posts.length }}
              <br />
              #followers: {{ user.followed_num }}
              <br />
              #following: {{ following.length }}
              <br />
              <a
                @click="followingModal = true"
              >
                See Following
              </a>
              |
              <span v-if="user.username != getProfile.username">
                <a
                  v-if="!getProfile.following.includes(user.id)"
                  @click="follow"
                >
                  Follow
                </a>
                <a
                  v-if="getProfile.following.includes(user.id)"
                  @click="unfollow"
                >
                  Unfollow
                </a>
              </span>
              <span v-else>
                <a @click="editProfileModal = true">
                  Edit Profile
                </a>
              </span>
            </h2>
          </div>
        </div>
      </div>
      <div v-if="posts">
        <Post
        v-for="post in posts"
        :key="post.id"
        :post="post"
      />
      </div>
      <div v-else>
        Waiting for posts.
      </div>
    </div>
    <div v-else-if="pending">
      The page is loading.
    </div>
    <div v-else>
      An error has occurred.
    </div>
    <b-modal
      :active.sync="followingModal"
      has-modal-card
      trap-focus
    >
      <FollowingModal :following="following" />
    </b-modal>
    <b-modal
      :active.sync="editProfileModal"
      has-modal-card
      trap-focus
    >
      <editProfileModal :user="user"/>
    </b-modal>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex';
import axios from 'axios';
export default {
  name: 'userPage',
  props: {
    username: String
  },
  data() {
    return {
      user: null,
      posts: [],
      following: [],
      pending: null,
      status: null,
      followingModal: false,
      editProfileModal: false
    }
  },
  computed: {
    ...mapGetters([
      'getToken',
      'getProfile',
      'getApi'
    ])
  },
  methods: {
    ...mapMutations([
      'addFollow',
      'removeFollow'
    ]),
    getUser() {
      const options = {
        url: `${this.getApi}/user?username=${this.username}`,
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Token ${this.getToken}`
        }
      }
      this.pending = true;
      axios(options)
      .then((response) => {
        this.user = response.data;
        this.user.posts.forEach((post) => {
          this.getPost(post);
        });
        this.user.following.forEach((user) => {
          this.getFollowing(user);
        });
        this.status = true;
      })
      .catch((error) => {
        console.log(error);
        this.status = false;
      })
      .finally(() => {
        this.pending = false;
      })
    },
    getPost(id) {
      const options = {
        url: `${this.getApi}/post?id=${id}`,
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Token ${this.getToken}`
        }
      }
      this.pending = true;
      axios(options)
      .then((response) => {
        this.posts.push(response.data);
      })
      .catch((error) => {
        console.log(error);
        this.status = false;
      })
    },
    getFollowing(id) {
      const options = {
        url: `${this.getApi}/user?id=${id}`,
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Token ${this.getToken}`
        }
      }
      this.pending = true;
      axios(options)
      .then((response) => {
        this.following.push(response.data);
      })
      .catch((error) => {
        console.log(error);
        this.status = false;
      })
    },
    follow() {
      const options = {
        url: `${this.getApi}/user/follow?username=${this.username}`,
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Token ${this.getToken}`
        },
      }
      this.pending = true;
      axios(options)
      .then(() => {
        this.status = true;
        this.addFollow(this.user.id);
      })
      .catch((error) => {
        console.log(error);
        this.status = false;
      })
      .finally(() => {
        this.pending = false;
      })
    },
    unfollow() {
       const options = {
        url: `${this.getApi}/user/unfollow?username=${this.username}`,
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Token ${this.getToken}`
        },
      }
      this.pending = true;
      axios(options)
      .then(() => {
        this.status = true;
        this.removeFollow(this.user.id);
      })
      .catch((error) => {
        console.log(error);
        this.status = false;
      })
      .finally(() => {
        this.pending = false;
      })
    }
  },
  created() {
    this.getUser();
  }
}
</script>

<style scoped>
.button {
  margin-right: 1rem;
}
</style>