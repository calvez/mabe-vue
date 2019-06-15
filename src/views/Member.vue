<template>
  <div>
    <article>
      <a @click="goBack">Back to All Posts</a>

      <header>
        <img v-if="featured_image" :src="featured_image" />
        <h1 v-html="title"></h1>

        <ul>
          <li>
            <span>Published on {{ date }}</span>
          </li>
          <li>
            <span>
              <a :href="link">View Post at Source</a>
            </span>
          </li>
        </ul>
      </header>

      <PostBody :content="content"></PostBody>
      <template>
        <v-layout>
          <v-flex xs12 sm6 offset-sm3>
            <v-card>
              <v-img
                src="https://cdn.vuetifyjs.com/images/cards/desert.jpg"
                aspect-ratio="2.75"
              ></v-img>

              <v-card-title primary-title>
                <div>
                  <h3 class="headline mb-0">Kangaroo Valley Safari</h3>
                  <div>{{ card_text }}</div>
                </div>
              </v-card-title>

              <v-card-actions>
                <v-btn flat color="orange">Share</v-btn>
                <v-btn flat color="orange">Explore</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
      </template>
    </article>
  </div>
</template>

<script>
import bus from "../bus";
import utils from "../mixins/utils";
import ajax from "../mixins/ajax";
import PostBody from "../components/PostBody";

export default {
  name: "Member",

  mixins: [utils, ajax],

  data() {
    return {
      post: {},
      date: "",
      link: "",
      title: "",
      content: "",
      featured_image: ""
    };
  },

  created: async function() {
    this.post = await this.setPost();
    this.link = this.post.link;
    this.date = this.getFormattedDate(this.post.date);
    this.title = this.post.title.rendered;
    this.content = this.post.content.rendered;
    this.featured_image = await this.getFeaturedImage(this.post.featured_media);

    bus.$emit("toggleLoading", false);
  },

  methods: {
    setPost: function() {
      return new Promise(async resolve => {
        let response;

        try {
          response = await this.get(
            `/mabe_tagok?slug=${this.$route.params.slug}`
          );
        } catch (error) {
          this.$router.push({ name: "four-o-four" });
          return;
        }

        resolve(response.data[0]);
      });
    },

    getFeaturedImage: async function(id) {
      let response;
      try {
        if (post.featured_media <= 0) {
          throw "No featured image.";
        }

        response = await this.get(`/media/${id}`);
      } catch (error) {
        return null;
      }

      return response.data.media_details.sizes["medium"].source_url;
    },
    toggleAll() {
      if (this.selected.length) this.selected = [];
      else this.selected = this.desserts.slice();
    },
    changeSort(column) {
      if (this.pagination.sortBy === column) {
        this.pagination.descending = !this.pagination.descending;
      } else {
        this.pagination.sortBy = column;
        this.pagination.descending = false;
      }
    }
  },

  components: {
    PostBody
  }
};
</script>

<style scoped lang="scss">
article {
  margin: 0 auto;
  background: $gray--light;
  border: 2px solid darken($gray--light, 5%);
  padding: 1rem;

  @include media($small) {
    padding: 3rem;
  }
}

header {
  margin-bottom: 1rem;
}

h1 {
  margin: 2rem 0 0;
}

ul {
  @include media($mobile) {
    display: grid;
    grid-template-columns: auto 1fr;
    grid-gap: 5px;
  }
}

li {
  & + & {
    &:before {
      @include media($mobile) {
        content: "|";
        float: left;
        margin: 0 5px 0 0;
      }
    }
  }

  a {
    color: inherit;
    font-weight: 600;

    &:hover {
      color: $action-color;
    }
  }
}

a {
  display: block;
  margin-bottom: 1rem;
}
</style>
