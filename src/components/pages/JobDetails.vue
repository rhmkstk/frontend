<script>
import { mapActions } from 'vuex';
import JobTypeBadge from '../shared/JobTypeBadge';
import JobListing from '../shared/JobListing';
import ApplyLink from '../shared/ApplyLink';
import Loader from '../shared/Loader';
import image from '../../mixins/image';
import jobDetailsMetaInfoMixin from '../../mixins/jobDetailsMetaInfo';

export default {
  props: {
    preview: {
      type: Boolean,
      required: false,
      default: false,
    },
    previewData: {
      type: Object,
      required: false,
      default: () => ({}),
    },
  },
  mixins: [jobDetailsMetaInfoMixin, image],
  components: {
    Loader,
    ApplyLink,
    JobTypeBadge,
    JobListing,
  },
  data() {
    return {
      post: null,
      isLoading: true,
      notFound: false,
      relatedPosts: [],
    };
  },
  computed: {
    postData() {
      return Object.keys(this.previewData).length ? this.previewData : this.post;
    },
    twitterUrl() {
      return `https://twitter.com/${this.postData.company.twitter}`;
    },
    twitterShareUrl() {
      const { twitter } = this.postData.company;
      const twitterHandle = twitter ? `@${twitter}` : '';
      const meta = `https://kodilan.com/ilanlar/${this.postData.slug} ${twitterHandle}`;
      const encoded = encodeURI(
        `kodilan.com'da yayınlanan ${
          this.postData.position
        } başlıklı ilan ilginizi çekebilir. ${meta} @kodilancom`,
      );

      return `https://twitter.com/intent/tweet?text=${encoded.trim()}`;
    },
  },
  methods: {
    ...mapActions(['fetchBySlug', 'fetchRelatedPosts']),
    fetchData() {
      if (!this.preview) {
        this.fetchBySlug(this.$route.params.slug)
          .then((res) => {
            this.isLoading = false;
            this.post = res;

            this.fetchRelatedPosts(this.post)
              .then((posts) => {
                this.relatedPosts = posts;
              });
          })
          .catch(() => {
            this.isLoading = false;
            this.notFound = true;
          });
      }
    },
  },
  watch: {
    $route(from, to) {
      if (to.name === 'JobDetails' && from.params.slug !== to.params.slug) {
        this.fetchData();
      }
    },
  },
  created() {
    this.fetchData();
  },
};
</script>

<template>
  <div
    :class="{ preview: preview }"
    class="job-details"
  >
    <loader v-if="isLoading && !preview" />
    <template v-else>
      <div v-if="preview" class="notification preview warning">
        <p>
          <span>Dikkat!</span>
          Bu sadece bir önizlemedir. İlanınızı yayınlamak için lütfen kaydettiğinizden emin olun.
        </p>
      </div>
      <template v-if="postData">
        <div id="titlebar">
          <div class="container">
            <div class="ten columns">
              <h1>
                {{ postData.position }}
                <job-type-badge :post="postData" />
              </h1>
            </div>
            <div class="six columns">
              <a :href="twitterShareUrl" class="button dark" target="_blank">
                <i class="fa fa-twitter" /> Paylaş
              </a>
            </div>
          </div>
        </div>
        <div class="container">
          <div class="eleven columns">
            <div class="padding-right">
              <div class="company-info">
                <img
                  :src="postData.company.logo"
                  :alt="postData.company.name"
                  ref="logo"
                  @error="handleImageError"
                >
                <div class="content">
                  <h4>{{ postData.company.name }}</h4>
                  <span>
                    <a :href="postData.company.www" target="_blank">
                      <i class="fa fa-link" /> Website
                    </a>
                  </span>
                  <span v-if="postData.company.twitter">
                    <a :href="twitterUrl" target="_blank">
                      <i class="fa fa-twitter" /> @{{ postData.company.twitter.replace('@', '') }}
                    </a>
                  </span>
                  <span v-if="postData.company.linkedin">
                    <a :href="postData.company.linkedin" target="_blank">
                      <i class="fa fa-linkedin" /> {{ postData.company.name }}
                    </a>
                  </span>
                </div>
                <div class="clearfix" />
              </div>
              <div class="job-description" v-html="postData.description" />
            </div>
          </div>
          <div class="five columns">
            <div class="widget">
              <div class="job-overview">
                <ul>
                  <li>
                    <i class="fa fa-user" />
                    <div>
                      <strong>Pozisyon</strong>
                      <span>{{ postData.position }}</span>
                    </div>
                  </li>
                  <li>
                    <i class="fa fa-map-marker" />
                    <div>
                      <strong>Lokasyon</strong>
                      <span>{{ postData.location }}</span>
                    </div>
                  </li>
                  <li>
                    <i class="ln ln-icon-Globe" />
                    <div>
                      <strong>Website</strong>
                      <span>
                        <a :href="postData.company.www" target="_blank">
                          {{ postData.company.www }}
                        </a>
                      </span>
                    </div>
                  </li>
                  <li>
                    <i class="ln ln-icon-Tag-3" />
                    <div>
                      <p>
                        <strong>Etiketler</strong>
                      </p>
                      <span
                        v-for="tag in postData.tags"
                        :key="tag.slug"
                        class="tag"
                      >
                        <router-link :to="`/etiket/${tag.slug}`" class="listing-date">
                          {{ tag.name }}
                        </router-link>
                      </span>
                    </div>
                  </li>
                </ul>
                <apply-link :post="postData" />
              </div>
            </div>
          </div>
          <div class="sixteen columns" v-if="relatedPosts.length">
            <h3 class="margin-bottom-25">
              Benzer ilanlar
            </h3>
            <job-listing :posts="relatedPosts" />
          </div>
        </div>
      </template>
      <div v-else class="notification notice center">
        <p>
          <span>Böyle bir ilan bulunamadı.</span>
        </p>
      </div>
    </template>
  </div>
</template>

<style lang="scss">
.job-details {
  #titlebar .listing-type {
    right: 0;

    a {
      font-size: inherit !important;
      color: inherit !important;
    }
  }

  &.preview {
    width: 1200px;
    margin: 0 auto;
    border: 4px solid #ccc;
    border-radius: 4px;
    pointer-events: none;
    box-sizing: border-box;

    #titlebar,
    .container {
      opacity: 0.66;
    }

    #titlebar {
      .container {
        width: 1060px;
      }
    }

    .notification.preview {
      margin-bottom: 0;
      font-size: 18px;
    }
  }

  .tag {
    display: inline-block;
    margin: 0 5px 5px 0;
  }

  .job-description li {
    line-height: 22px;
    list-style-type: disc;
    list-style-position: inside;
  }
}
</style>
