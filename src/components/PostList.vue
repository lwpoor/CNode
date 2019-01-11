<template>
	<div class="PostList">
	    <div class="loading" v-if="loading">
	        Loading...
	    </div>
	    <div class="posts" v-else>
			<ul>
				<li v-for="post in posts">
					<router-link :to="{ name: 'user_info', params: { name: post.author.loginname }}" :title="post.author_id">
						<img :src="post.author.avatar_url" :title="post.author.loginname"/>
					</router-link>
					<span>
						{{ post.reply_count }}/{{ post.visit_count }}
					</span>
					<router-link :to="{ name: 'post_content', params: { id: post.id,name:post.author.loginname }}" :title="post.title">
						{{ post.title }}
					</router-link>
					<span class="last_reply">
						{{ post.last_reply_at | formatDate}}
					</span>
				</li>
			</ul>
	    </div>
	</div>
</template>


<script>

  import $ from 'jquery'
  import Vue from 'vue'
  import layer from 'vue-layer'
  Vue.prototype.$layer = layer(Vue);

  var that = null;
  var pos=2;
	export default {
	  name: 'PostList',
	  data () {
	    return {
	      posts:{},
	      loading:false
	    }
	  },
      filters: {
        timeStyle(createTime) {
            return String(createTime).match(/.{10}/)[0];
        }
    },
	  methods: {
      getData() {
        that = this;
        this.$http({
          url: 'https://cnodejs.org/api/v1/topics',
          method: 'get',
          params: {
            page: 1,
            limit: 20,
          }
        })
          .then((response) => {
            if (response.data.success === true) {
              this.posts = response.data.data;
              this.loading = false;
              pos = 2;
            }
          })
          .catch(function (error) {
            this.$layer.msg(error);
          });
      },
    },
	    beforeMount() {
	    	this.loading = true;
	    	this.getData();
	    }
	}


    //-----------------分页开始--------------------------
    $(window).scroll(function () {
    var scrollTop = $(this).scrollTop();
    var scrollHeight = $(document).height();
    var windowHeight = $(this).height();

    if (scrollTop + windowHeight == scrollHeight) {
      //此处是滚动条到底部时候触发的事件，在这里写要加载的数据，或者是拉动滚动条的操作
      that.$http({
        url: 'https://cnodejs.org/api/v1/topics',
        method: 'get',
        params: {
          page: pos,
          limit: 20,
        }
      })
        .then((response) => {
          if (response.data.success === true) {
            that.posts = that.posts.concat(response.data.data);
            that.loading = false;
            pos++;
          }
        })
        .catch(function (error) {
          that.$layer.msg(error);
        });
    }
  })
  //-----------------分页结束--------------------------


</script>

<style scoped>

  @media all and (max-width: 768px){
    .PostList .posts li {
      display: flex;
    }
    .PostList .posts a {
      max-width: 50%;
    }
    .PostList .posts .last_reply {
      right: 0px;
      position: absolute;
    }

  }
  @media all and (min-width: 768px){
    .PostList .posts a {
      max-width: 70%;
    }
  }

	.PostList .posts {
		background-color: white;
		padding: 0.5rem;
	}
	.PostList .posts li {
		list-style: none;
		margin-bottom: 14px;
		border-bottom: 1px solid #E7E7E7;
		line-height: 30px;
	}
	.PostList .posts ul li img {
		width: 1.5rem;
		height: 1.5rem;
	}
	.PostList .posts li span {
		display: inline-block;
		text-align: center;
		width: 70px;
		font-size: 12px;
		margin: 0 10px;
	}
	.PostList .posts a {
		text-decoration: none;
		color: inherit;
	    -o-text-overflow: ellipsis;
	    white-space: nowrap;
	    display: inline-block;
	    vertical-align: middle;
	    overflow: hidden;
	    text-overflow: ellipsis;
	}
	.PostList .posts a:visited {
		color:#858585;
	}
	.PostList .posts .last_reply {
		float: right;
   		font-size: 0.7rem;
   	    margin-top: 0.3rem;
	}
</style>
