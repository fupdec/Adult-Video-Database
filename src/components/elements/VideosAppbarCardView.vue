<template>
	<div>
    <v-menu offset-y nudge-bottom="10" :close-on-content-click="false">
      <template v-slot:activator="{ on, attrs }">
        <v-badge :content="getCardSizeIcon()" class="text-uppercase" color="secondary" overlap offset-x="25" offset-y="25">
          <v-btn v-bind="attrs" v-on="on" icon tile>
            <v-tooltip bottom>
              <template v-slot:activator="{ on }">
                <v-icon v-on="on">mdi-card-bulleted</v-icon>
              </template>
              <span>Card size</span>
            </v-tooltip>
          </v-btn>
        </v-badge>
      </template>
      <v-card width="300">
        <v-card-title class="py-1">
          <span class="headline">Card size</span>
          <v-spacer></v-spacer>
          <v-icon>mdi-card-bulleted-settings</v-icon>
        </v-card-title>
        <v-divider></v-divider>
        <v-slider v-model="$store.state.Settings.videoCardSize"
          min="1" max="5" step="1" :tick-labels="cardSizes"
          @input="changeCardSize" class="pa-6"
        />
      </v-card>
    </v-menu>

    <v-tooltip bottom>
      <template v-slot:activator="{ on }">
				<v-badge :value="isChipsColored" class="colored" overlap offset-x="25" offset-y="25">
          <v-btn @click="toggleChipsColored()" v-on="on" icon tile>
            <v-icon>mdi-label</v-icon>
          </v-btn>
        </v-badge>
      </template>
      <span v-if="isChipsColored">Make labels grey</span>
      <span v-else>Make labels colored</span>
    </v-tooltip>

    
    <v-menu bottom offset-y min-width="160">
      <template v-slot:activator="{ on, attrs }">
        <v-btn v-bind="attrs" v-on="on" icon tile>
          <v-icon>mdi-dots-vertical</v-icon>
        </v-btn>
      </template>
      
      <v-list dense class="context-menu">
        <v-list-item link @click="toggleEditBtnVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-account-edit</v-icon> Edit Buttons
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isEditBtnHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item link @click="toggleQualityLabelVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-video-box</v-icon> Quality Label
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isQualityLabelHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item link @click="toggleDurationVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-timer</v-icon> Duration
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isDurationHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item v-if="!$store.state.Settings.ratingAndFavoriteInCard" link @click="toggleRatingVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-star</v-icon> Rating
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isRatingHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item v-if="!$store.state.Settings.ratingAndFavoriteInCard" link @click="toggleFavoriteVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-heart</v-icon> Favorite
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isFavoriteHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item link @click="toggleFileNameVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-alphabetical-variant</v-icon> Filename
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isFileNameHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item link @click="toggleFileInfoVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-filmstrip</v-icon> File Information
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isFileInfoHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item link @click="togglePerformerVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-account</v-icon> Performers
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isPerformersHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item link @click="toggleTagsVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-tag</v-icon> Tags
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isTagsHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>

        <v-list-item link @click="toggleWebsiteVisibilty()">
          <v-list-item-title>
            <v-icon left size="18">mdi-web</v-icon> Websites
          </v-list-item-title>
          <v-icon size="20" class="pl-10" :color="!isWebsiteHidden?'':'rgba(0,0,0,0)'">mdi-check</v-icon>
        </v-list-item>
      </v-list>
    </v-menu>
	</div>
</template>


<script>
export default {
  name: 'VideosAppbarCardView',
  components: {
  },
  mounted() {
    this.$nextTick(function () {
    })
  },
  data: () => ({
    cardSizes: ['XS','S','M','L','XL'],
  }),
  computed: {
    isChipsColored: {
      get() {
        return this.$store.state.Settings.videoChipsColored
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoChipsColored', value})
      },
    },
    isEditBtnHidden: {
      get() {
        return this.$store.state.Settings.videoEditBtnHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoEditBtnHidden', value})
      },
    },
    isFileNameHidden: {
      get() {
        return this.$store.state.Settings.videoFileNameHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoFileNameHidden', value})
      },
    },
    isFileInfoHidden: {
      get() {
        return this.$store.state.Settings.videoFileInfoHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoFileInfoHidden', value})
      },
    },
    isRatingHidden: {
      get() {
        return this.$store.state.Settings.videoRatingHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoRatingHidden', value})
      },
    },
    isFavoriteHidden: {
      get() {
        return this.$store.state.Settings.videoFavoriteHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoFavoriteHidden', value})
      },
    },
    isQualityLabelHidden: {
      get() {
        return this.$store.state.Settings.videoQualityLabelHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoQualityLabelHidden', value})
      },
    },
    isDurationHidden: {
      get() {
        return this.$store.state.Settings.videoDurationHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoDurationHidden', value})
      },
    },
    isPerformersHidden: {
      get() {
        return this.$store.state.Settings.videoPerformersHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoPerformersHidden', value})
      },
    },
    isTagsHidden: {
      get() {
        return this.$store.state.Settings.videoTagsHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoTagsHidden', value})
      },
    },
    isWebsiteHidden: {
      get() {
        return this.$store.state.Settings.videoWebsiteHidden
      },
      set(value) {
        this.$store.dispatch('updateSettingsState', {key:'videoWebsiteHidden', value})
      },
    },
  },
  methods: {
    changeCardSize(value) {
      this.$store.dispatch('updateSettingsState', {key:'videoCardSize', value})
      this.getCardSizeIcon()
    },
    getCardSizeIcon() {
      let size = ''
      switch(this.$store.state.Settings.videoCardSize) {
        case 1: size = 'xs'; break;
        case 2: size = 's'; break;
        case 3: size = 'm'; break;
        case 4: size = 'l'; break;
        case 5: size = 'xl'; break;
      }
      return size
    },
    toggleChipsColored() {
      this.isChipsColored = !this.isChipsColored
    },
    toggleEditBtnVisibilty() {
      this.isEditBtnHidden = !this.isEditBtnHidden
    },
    toggleFileNameVisibilty() {
      this.isFileNameHidden = !this.isFileNameHidden
    },
    toggleQualityLabelVisibilty() {
      this.isQualityLabelHidden = !this.isQualityLabelHidden
    },
    toggleDurationVisibilty() {
      this.isDurationHidden = !this.isDurationHidden
    },
    toggleRatingVisibilty() {
      this.isRatingHidden = !this.isRatingHidden
    },
    toggleFavoriteVisibilty() {
      this.isFavoriteHidden = !this.isFavoriteHidden
    },
    toggleFileInfoVisibilty() {
      this.isFileInfoHidden = !this.isFileInfoHidden
    },
    togglePerformerVisibilty() {
      this.isPerformersHidden = !this.isPerformersHidden
    },
    toggleTagsVisibilty() {
      this.isTagsHidden = !this.isTagsHidden
    },
    toggleWebsiteVisibilty() {
      this.isWebsiteHidden = !this.isWebsiteHidden
    },
  },
  watch: {
  },
}
</script>