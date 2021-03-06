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
        <v-parallax
          dark
          src="https://cdn.vuetifyjs.com/images/backgrounds/vbanner.jpg"
        >
          <v-layout align-center column justify-center>
            <h1 class="display-2 font-weight-thin mb-3">Vuetify.js</h1>
            <h4 class="subheading">Build your application today!</h4>
          </v-layout>
        </v-parallax>
      </template>
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

      <template>
        <div class="text-xs-center">
          <v-rating v-model="rating"></v-rating>
        </div>
      </template>

      <template>
        <v-layout align-center justify-space-around>
          <v-icon>fas fa-lock</v-icon>

          <v-icon>fas fa-search</v-icon>

          <v-icon>fas fa-list</v-icon>

          <v-icon>fas fa-edit</v-icon>

          <v-icon>fas fa-tachometer-alt</v-icon>

          <v-icon>fas fa-circle-notch fa-spin</v-icon>
        </v-layout>
      </template>

      <template>
        <v-data-table
          v-model="selected"
          :headers="headers"
          :items="desserts"
          :pagination.sync="pagination"
          select-all
          item-key="name"
          class="elevation-1"
        >
          <template v-slot:headers="props">
            <tr>
              <th>
                <v-checkbox
                  :input-value="props.all"
                  :indeterminate="props.indeterminate"
                  primary
                  hide-details
                  @click.stop="toggleAll"
                ></v-checkbox>
              </th>
              <th
                v-for="header in props.headers"
                :key="header.text"
                :class="[
                  'column sortable',
                  pagination.descending ? 'desc' : 'asc',
                  header.value === pagination.sortBy ? 'active' : ''
                ]"
                @click="changeSort(header.value)"
              >
                <v-icon small>fas fa-up</v-icon>
                {{ header.text }}
              </th>
            </tr>
          </template>
          <template v-slot:items="props">
            <tr
              :active="props.selected"
              @click="props.selected = !props.selected"
            >
              <td>
                <v-checkbox
                  :input-value="props.selected"
                  primary
                  hide-details
                ></v-checkbox>
              </td>
              <td>{{ props.item.name }}</td>
              <td class="text-xs-right">{{ props.item.calories }}</td>
              <td class="text-xs-right">{{ props.item.fat }}</td>
              <td class="text-xs-right">{{ props.item.carbs }}</td>
              <td class="text-xs-right">{{ props.item.protein }}</td>
              <td class="text-xs-right">{{ props.item.iron }}</td>
            </tr>
          </template>
        </v-data-table>
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
  name: "Post",

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
          response = await this.get(`/posts?slug=${this.$route.params.slug}`);
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
