<template>

    <transition name="slide-fade">

        <div class="row rt-main" v-if="loaded === 'true'" >

            <div class="medium-12 small-12 column" v-for="post in posts" :key="post.slug">

                <div class="rt-post">

                    <h2 class="rt-post-title"><router-link :to="{ name: 'post', params: { postSlug:post.slug }}"> {{ post.title.rendered }} </router-link> </h2>

                    <div class="rt-meta">
						<span class="posted-on">
							Posted On
							<span class="date" v-text="formatDate( post )">
							</span>
						</span>
                    </div>

                    <div class="progressive full" v-if="post.featured_image_src['full'][0]">

                        <img class="lazy" :src="post.featured_image_src['full'][0]" />

                    </div>

                    <div class="rt-post-excerpt rt-content" v-html="post.excerpt.rendered" > </div>

                </div>

            </div>

            <div class="pagination row">
                <div v-if="page > 1" class="newer-posts large-6 medium-6 small-6 columns">
                    <router-link :to="{ name: 'Category', query: { page: this.page - 1 }} " class="pagination-previous button" v-on:click.native="previousClick">Previous</router-link>
                </div>
                <div v-if="page < totalPages" class="newer-posts large-6 medium-6 small-6 columns">
                    <router-link :to="{ name: 'Category', query: { page: this.page + 1 }}" class="pagination-next button" v-on:click.native="nextClick">Next</router-link>
                </div>
            </div>

        </div>

    </transition>

</template>

<script>
    import axios from 'axios'

    export default {
        mounted: function() {
            const vm = this;
            if ( vm.$route.params.categorySlug ) {
                vm.getCatId( vm.$route.params.categorySlug );
            }
        },
        data() {
            if ( this.$route.query.page ) {
            } else {
                this.$route.query.page = 1;
            }
            return {
                posts: {},
                loaded: 'false',
                pageTitle: '',
                page: this.$route.query.page,
                totalPages: '',
                totalCount: ''
            };
        },
        methods: {
            getPosts: function( catId ) {
                const vm = this;
                vm.loaded = 'false';
                if (vm.$route.query.page !== undefined) {
                    vm.postsUrl = '&page=' + vm.$route.query.page;
                } else {
                    vm.postsUrl = '';
                }
                axios.get( vuefoundationstarter.root + 'wp/v2/posts?categories=' + catId + vm.postsUrl )
                    .then( ( res ) => {
                        vm.posts = res.data;
                        vm.totalPages = res.headers['x-wp-totalpages'];
                        if (vm.posts === undefined) {
                            vm.$router.push({name: 'NotFound'})
                        }

                        vm.loaded = 'true';
                        vm.pageTitle = 'Category' + ' - ' + vm.catName;
                        vm.$store.commit( 'themeSlugChangeTitle', vm.pageTitle );
                    } )
                    .catch( ( res ) => {
                        //console.log( `Something went wrong : ${ res }` );
                    } );
            },
            getCatId: function( name ) {
                const vm = this;
                vm.loaded = 'false';
                axios.get( vuefoundationstarter.root + 'wp/v2/categories?slug=' + name )
                    .then( ( res ) => {
                        res = res.data[ 0 ];
                        vm.catName = res.name;

                        if (res === undefined) {
                            vm.$router.push({name: 'NotFound'})
                        }

                        vm.totalCount = ( res.data );
                        vm.getPosts( res.id );
                    } )
                    .catch( ( res ) => {
                        //console.log( `Something went wrong : ${ res }` );
                    } );
            },
            formatDate: function( value ) {
                value = value.date;
                if ( value ) {
                    const date = new Date( value );
                    const monthNames = [ "January", "February", "March",
                        "April", "May", "June", "July",
                        "August", "September", "October",
                        "November", "December" ];
                    const day = date.getDate();
                    const monthIndex = date.getMonth();
                    const year = date.getFullYear();
                    return monthNames[ monthIndex ] + ', ' + day + ' ' + year;
                }
            },
            previousClick () {
                this.page = this.page - 1;
                this.getCatId(this.$route.params.categorySlug);
                window.scrollTo(0, 0);
            },
            nextClick () {
                this.page = this.page + 1;
                this.getCatId(this.$route.params.categorySlug);
                window.scrollTo(0, 0);
            },
        }
    };
</script>