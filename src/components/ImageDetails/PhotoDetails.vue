<template>
  <div class="photo columns is-desktop is-marginless padding-bottom-xl">
    <div class="column is-three-fifths photo_image-ctr margin-top-normal">
      <a
        class="is-block photo_breadcrumb has-text-left margin-left-normal margin-bottom-normal has-text-grey-dark has-text-weight-semibold caption"
        :href="breadCrumbURL"
        @click.prevent="onGoBackToSearchResults"
        v-if="shouldShowBreadcrumb"
      >
        <i class="icon chevron-left margin-right-small" />
        {{ $t('photo-details.back') }}
      </a>
      <img
        @load="onImageLoad"
        class="photo_image"
        :src="image.url"
        :alt="image.title"
      />

      <legal-disclaimer />

      <div class="margin-bottom-smaller has-text-left">
        <button
          class="button is-text tiny is-paddingless report is-shadowless"
          @click="toggleReportFormVisibility()"
        >
          <span class="has-color-tomato margin-left-small">
            <i class="icon flag margin-right-small"></i>
            {{ $t('photo-details.content-report.title') }}
          </span>
        </button>
      </div>
      <div class="margin-top-small has-text-left">
        <content-report-form
          v-if="isReportFormVisible"
          :imageId="image.id"
          :imageURL="image.foreign_landing_url"
          :providerName="providerName"
        />
      </div>
    </div>
    <div
      role="region"
      aria-label="image details"
      class="column image-info margin-left-xl"
    >
      <div class="margin-top-normal margin-bottom-small">
        <h5 class="b-header">{{ image.title }}</h5>
        <span v-if="image.creator" class="caption has-text-weight-semibold">
          by
          <a
            :aria-label="'author' + image.creator"
            v-if="image.creator_url"
            :href="image.creator_url"
          >
            {{ image.creator }}
          </a>
          <span v-else>{{ image.creator }}</span>
        </span>
      </div>
      <section class="tabs">
        <ul role="tablist">
          <li
            role="tab"
            :aria-selected="activeTab == 0"
            :class="tabClass(0, 'tab')"
          >
            <a href="#panel0" @click.prevent="setActiveTab(0)">
              {{ $t('photo-details.reuse.title') }}
            </a>
          </li>
          <li
            role="tab"
            :aria-selected="activeTab == 1"
            :class="tabClass(1, 'tab')"
          >
            <a href="#panel1" @click.prevent="setActiveTab(1)">
              {{ $t('photo-details.information.title') }}
            </a>
          </li>
          <li
            role="tab"
            :aria-selected="activeTab == 2"
            :class="tabClass(2, 'a')"
            v-if="socialSharingEnabled"
          >
            <a href="#panel2" @click.prevent="setActiveTab(2)">
              {{ $t('photo-details.share') }}
            </a>
          </li>
        </ul>
      </section>
      <section class="photo_info-ctr tabs-content">
        <div :class="tabClass(0, 'tabs-panel')">
          <image-attribution
            :image="image"
            :ccLicenseURL="ccLicenseURL"
            :fullLicenseName="fullLicenseName"
            :attributionHtml="attributionHtml()"
          />
        </div>
        <div :class="tabClass(1, 'tabs-panel')">
          <image-info
            :image="image"
            :ccLicenseURL="ccLicenseURL"
            :fullLicenseName="fullLicenseName"
            :imageWidth="imageWidth"
            :imageHeight="imageHeight"
            :imageType="imageType"
          />
        </div>
        <div :class="tabClass(2, 'tabs-panel')">
          <image-social-share v-if="socialSharingEnabled" :image="image" />
        </div>
      </section>

      <a
        v-if="activeTab < 2"
        :href="image.foreign_landing_url"
        target="_blank"
        rel="noopener"
        class="button is-success margin-bottom-small"
        @click="onPhotoSourceLinkClicked"
      >
        {{ $t('photo-details.weblink') }}
        <i
          class="icon external-link margin-left-normal is-size-6 padding-top-smaller has-text-grey-lighter"
        />
      </a>

      <reuse-survey v-if="activeTab < 2" :image="image" />
    </div>
  </div>
</template>

<script>
import ContentReportForm from '@/components/ContentReport/ContentReportForm'
import { TOGGLE_REPORT_FORM_VISIBILITY } from '@/store/mutation-types'
import {
  SEND_DETAIL_PAGE_EVENT,
  DETAIL_PAGE_EVENTS,
} from '@/store/usage-data-analytics-types'
import attributionHtml from '@/utils/attributionHtml'
import ImageInfo from './ImageInfo'
import ImageAttribution from './ImageAttribution'
import ImageSocialShare from './ImageSocialShare'
import LegalDisclaimer from './LegalDisclaimer'
import ReuseSurvey from './ReuseSurvey'

export default {
  name: 'photo-details',
  props: [
    'image',
    'breadCrumbURL',
    'shouldShowBreadcrumb',
    'query',
    'imageWidth',
    'imageHeight',
    'imageType',
    'socialSharingEnabled',
  ],
  components: {
    ImageInfo,
    ImageAttribution,
    ImageSocialShare,
    LegalDisclaimer,
    ContentReportForm,
    ReuseSurvey,
  },
  data() {
    return {
      activeTab: 0,
    }
  },
  computed: {
    isReportFormVisible() {
      return this.$store.state.isReportFormVisible
    },
    fullLicenseName() {
      const license = this.image.license
      const version = this.image.license_version

      if (license) {
        return license.toLowerCase() === 'cc0'
          ? `${license} ${version}`
          : `CC ${license} ${version}`
      }
      return ''
    },
    ccLicenseURL() {
      return `${this.image.license_url}?ref=ccsearch`
    },
  },
  methods: {
    onGoBackToSearchResults() {
      this.$router.push({
        path: '/search',
        query: this.query,
        params: { location: this.$route.params.location },
      })
    },
    onImageLoad(event) {
      this.$emit('onImageLoaded', event)
    },
    tabClass(tabIdx, tabClass) {
      return {
        [tabClass]: true,
        'is-active': tabIdx === this.activeTab,
      }
    },
    setActiveTab(tabIdx) {
      this.activeTab = tabIdx
    },
    attributionHtml() {
      const licenseURL = `${this.ccLicenseURL}&atype=html`
      return attributionHtml(this.image, licenseURL, this.fullLicenseName)
    },
    toggleReportFormVisibility() {
      this.$store.commit(TOGGLE_REPORT_FORM_VISIBILITY)
    },
    onPhotoSourceLinkClicked() {
      this.$store.dispatch(SEND_DETAIL_PAGE_EVENT, {
        eventType: DETAIL_PAGE_EVENTS.SOURCE_CLICKED,
        resultUuid: this.$props.image.id,
      })
    },
  },
}
</script>

<style lang="scss">
@import '../../styles/photodetails.scss';
@import 'node_modules/bulma/sass/utilities/_all';

@include touch {
  .image-info {
    margin-left: 0 !important;
  }
}
</style>
