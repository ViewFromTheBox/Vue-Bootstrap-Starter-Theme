<template>
    <div id="homepage">
        <div class="list-posts container">
            <div v-for="post in posts" :key="post.id" v-bind:post="post" class="row">
                <div class="preview-post col-12">
                    <router-link :to="{ name:'post', params: {postSlug: post.slug} }">
                        <h2 class="post-title title is-3" v-html="post.title.rendered"></h2>
                        <p class="post-excerpt" v-html="post.excerpt.rendered"></p>
                        <p><router-link :to="{ name:'post', params: {postSlug: post.slug} }" class="read-more">Read More</router-link></p>
                    </router-link>
                </div>
            </div>
            <div class="pagination row">
                <div v-if="page > 1" class="newer-posts col-lg-6 col-md-6 col-sm-6 columns">
                    <router-link :to="{ name: 'Homepage', query: { page: this.page - 1 }}" class="pagination-previous button" v-on:click.native="previousClick">Previous</router-link>
                </div>
                <div v-if="page < totalPages" class="newer-posts col-lg-6 col-md-6 col-sm-6 columns">
                    <router-link :to="{ name: 'Homepage', query: { page: this.page + 1 }}" class="pagination-next button" v-on:click.native="nextClick">Next</router-link>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Homepage',
  data () {
    if ( this.$route.query.page ) {
    } else {
        this.$route.query.page = 1;
    }
    return {
      title: 'VueJS WordPress Theme',
      posts: {},
      page: this.$route.query.page,
      totalPages: '',
      paginUrl: ''
    }
  },
  created () {
    this.getPosts();
    document.title = this.title
  },
  methods: {
    getPosts () {
      let postsUrl = vuefoundationstarter.root + 'wp/v2/posts';
      if (this.$route.query.page !== undefined) {
        postsUrl += '?page=' + this.$route.query.page;
      }
      axios.get(postsUrl)
      .then(response => {
        this.totalPages = response.headers['x-wp-totalpages'];
        this.posts = response.data
      })
      .catch(e => {
        console.log(e)
      })
    },
    previousClick () {
        this.page = this.page - 1;
        this.getPosts();
        window.scrollTo(0, 0);
    },
    nextClick () {
        this.page = this.page + 1;
        this.getPosts();
        window.scrollTo(0, 0);
    },
  },
  computed: {
      previousRoute () {
          let route = {};
          route['name'] = 'Homepage';
          if (this.page > 1) {
              route['query'] = { page: this.page - 1 }
          }
          return route;
      },
      nextRoute () {
          let route = {};
          route['name'] = 'Homepage';
          route['query'] = { page: this.page + 1 };
          return route;
      },

  },
}
</script>