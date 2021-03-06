<template>
  <vuescroll ref="mainContainer" @handle-scroll="handleScroll">
    <v-toolbar dense>
      <v-spacer></v-spacer>
      <v-tooltip bottom v-if="colors.length">
        <template v-slot:activator="{ on }">
          <v-btn @click="clearColors" icon small tile class="mr-2" v-on="on">
            <v-icon size="16">mdi-cancel</v-icon>
          </v-btn>
        </template>
        <span>Clear all colors</span>
      </v-tooltip>
      <v-btn-toggle v-model="colors" group multiple color="primary" class="mr-4">
        <v-btn v-for="(color,i) in swatches" :key="i" icon small class="ma-0" :value="color">
          <v-icon size="10" :color="color">mdi-circle</v-icon>
        </v-btn>
      </v-btn-toggle>
      <v-btn-toggle v-model="chars" group multiple color="primary">
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-btn icon small class="ma-0" :value="27" v-on="on">#</v-btn>
          </template>
          <span>Symbol</span>
        </v-tooltip>
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-btn icon small class="ma-0" :value="0" v-on="on">0-9</v-btn>
          </template>
          <span>Number</span>
        </v-tooltip>
        <v-btn v-for="(char,i) in alphabet" :key="i" icon small class="ma-0" :value="i+1">
          {{char}}
        </v-btn>
      </v-btn-toggle>
      <v-tooltip bottom v-if="chars.length">
        <template v-slot:activator="{ on }">
          <v-btn @click="clearChars" icon small tile class="ml-2" v-on="on">
            <v-icon size="20">mdi-alphabetical-variant-off</v-icon>
          </v-btn>
        </template>
        <span>Clear all characters</span>
      </v-tooltip>
      <v-spacer></v-spacer>
    </v-toolbar>

    <div class="headline text-h3 text-center my-6">Websites
      <span class="text-h5">({{$store.getters.filteredWebsitesTotal}})</span>
    </div>

    <v-container v-if="filters.length>0" fluid class="d-flex justify-center align-start py-0">
      <v-icon left>mdi-filter</v-icon>
      <v-chip v-for="(filter, i) in filters" :key="i" class="mr-2 mb-2" color="primary" 
        small close :disabled="filter.lock" @click:close="removeFilter(i)">
        {{filter.param}} {{filter.cond}} 
        <span v-if="filter.type=='array'" class="ml-1">{{filter.val.join(', ')}}</span>
        <span v-else class="ml-1">{{filter.val}}</span>
      </v-chip>
    </v-container>
      
    <v-container fluid v-if="!$store.state.Websites.filteredEmpty" class="pagination-container my-6">
      <v-overflow-btn v-model="websitesPerPage" hint="items per page" persistent-hint
        :items="websitesPerPagePreset" dense height="36" solo disable-lookup hide-no-data
        class="items-per-page-dropdown" 
      ></v-overflow-btn>
      <v-spacer></v-spacer>
      <v-pagination
        v-model="websitesCurrentPage"
        :length="websitesPagesSum"
        :total-visible="getNumberOfPagesLimit"
      ></v-pagination>
      <v-spacer></v-spacer>
      <v-overflow-btn v-if="websitesPagesSum > 5"
        v-model="websitesCurrentPage" :items="pages" dense height="36" solo
        class="items-per-page-dropdown width-70 jump-to-page-menu" 
        disable-lookup hint="jump to page" persistent-hint hide-no-data
        :menu-props="{ 
          auto:true, 
          contentClass:'jump-to-page-menu',
          nudgeBottom: -110,
          origin:'center center', 
          transition:'scale-transition'
        }"
      ></v-overflow-btn>
      <div v-else style="min-width:80px;"></div>
    </v-container>

    <div v-if="$store.state.Websites.filteredEmpty" class="text-center"> 
      <div><v-icon size="100" class="ma-10">mdi-close</v-icon></div>
      There are no matching websites for the selected filters.
    </div>
    
    <Loading />

    <v-container fluid class="websites-grid" :class="gapSize">
      <WebsiteCard v-for="(website) in websitesOnPage" :key="website.id" :website="website"/>
    </v-container>

    <v-pagination v-if="!$store.state.Websites.filteredEmpty"
      v-model="websitesCurrentPage" :length="websitesPagesSum"
      :total-visible="getNumberOfPagesLimit" class="mt-6 mb-10"
    ></v-pagination>
    
    <div v-show="$store.state.Settings.navigationSide=='2'" class="py-6"></div>

    <v-btn @click="scrollToTop" v-show="isScrollToTopVisible" 
      class="scroll-to-top" fixed fab color="primary">
      <v-icon>mdi-chevron-up</v-icon>
    </v-btn>

    <v-dialog v-model="$store.state.Websites.dialogDeleteWebsite" scrollable persistent max-width="600">
      <v-card>
        <v-card-title class="headline red--text">Are you sure?
          <v-spacer></v-spacer>
          <v-icon color="red">mdi-delete</v-icon>
        </v-card-title>
        <v-divider></v-divider>
        <vuescroll>
          <v-card-text style="white-space: pre-wrap;">
            <div>You want to delete website<span v-if="selectedWebsitesLength>1">s</span></div>
            {{selectedWebsites(true)}}
          </v-card-text>
        </vuescroll>
        <v-card-actions>
          <v-btn class="ma-4" 
            @click="$store.state.Websites.dialogDeleteWebsite = false">No, Keep it
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="red" class="ma-4"  dark @click="deleteWebsites">
            <v-icon left>mdi-delete-alert</v-icon> Yes, delete
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <DialogEditWebsite v-if="$store.state.Websites.dialogEditWebsite"/>

    <v-menu v-model="$store.state.Websites.menuCard" :position-x="$store.state.x" 
      :position-y="$store.state.y" absolute offset-y z-index="1000" min-width="150">
      <v-list dense class="context-menu">
        <v-list-item class="pr-1" link :disabled="!isSelectedSingleWebsite" @mouseup="addNewTabWebsite">
          <v-list-item-title>
            <v-icon left size="18">mdi-tab-plus</v-icon> Open in a new tab
          </v-list-item-title>
          <v-icon size="22" color="rgba(0,0,0,0)">mdi-menu-right</v-icon>
        </v-list-item>
        <v-divider class="ma-1"></v-divider>
        <v-list-item link @mouseup="$store.state.Websites.dialogEditWebsite = true"
          :disabled="!isSelectedSingleWebsite">
          <v-list-item-title>
            <v-icon left size="18">mdi-pencil</v-icon> Edit website
          </v-list-item-title>
        </v-list-item>
        <v-divider class="ma-1"></v-divider>
        <v-list-item link @mouseup="copyWebsiteNameToClipboard">
          <v-list-item-title>
            <v-icon left size="18">mdi-clipboard-text</v-icon> Copy website name
          </v-list-item-title>
        </v-list-item>
        <v-divider class="ma-1"></v-divider>
        <v-list-item link @mouseup="$store.state.Websites.dialogDeleteWebsite = true">
          <v-list-item-title>
            <v-icon left size="18" color="red">mdi-delete</v-icon> Delete website
          </v-list-item-title>
        </v-list-item>
      </v-list>
    </v-menu>
  </vuescroll>
</template>


<script>
import WebsiteCard from "@/components/pages/websites/WebsiteCard.vue"
import Selection from "@simonwep/selection-js";
import vuescroll from 'vuescroll'
import LabelFunctions from '@/mixins/LabelFunctions'

export default {
  name: "WebsitesPage",
  components: {
    WebsiteCard, 
    DialogEditWebsite: () => import("@/components/pages/websites/DialogEditWebsite.vue"),
    vuescroll,
    Loading: () => import('@/components/elements/Loading.vue'),
  },
  mixins: [LabelFunctions],
  mounted() {
    this.$nextTick(function () {
      this.initFilters()
      this.$store.state.Websites.selection = Selection.create({
        boundaries: ['.websites-grid'],
        selectables: ['.website-card'],
      }).on('beforestart', ({inst, selected, oe}) => {
        const targetEl = oe.target.closest('.website-card')
        if (oe.button === 2 && selected.includes(targetEl)) {
          return false
        }
        return (oe.button !== 1);
      }).on('start', ({inst, selected, oe}) => {
        const targetEl = oe.target.closest('.website-card')
        if (oe.button === 2 && selected.includes(targetEl)) {
          return false
        }
        // Remove class if the user isn't pressing the shift or control or ⌘ keys
        if (!oe.shiftKey && !oe.ctrlKey && !oe.metaKey) {
          // Unselect all elements
          for (const el of selected) {
              el.classList.remove('selected');
              inst.removeFromSelection(el);
          }
          // Clear previous selection
          inst.clearSelection();
        }
      }).on('move', ({changed: {removed, added}, inst, selected, oe}) => {
        // Add a custom class to the elements that where selected.
        for (const el of added) {
          el.classList.add('selected');
        }
        // Remove the class from elements that where removed
        // since the last selection
        for (const el of removed) {
          el.classList.remove('selected');
        }
      }).on('stop', ({inst, selected, oe}) => {
        if (oe.shiftKey || oe.ctrlKey || oe.metaKey) {
          let mergedCards = _.union(this.previousSelection, selected)
          let duplicates = _.filter(selected,(v,i,it)=>{return _.find(it, v, i + 1)})
          duplicates.map(duplicated=>{mergedCards = _.reject(mergedCards, duplicated)})
          selected = mergedCards
          this.previousSelection = mergedCards
          for (const el of duplicates) {
            inst.removeFromSelection(el)
          }
        } 
        inst.keepSelection()
        this.getSelectedWebsites(selected)
        let cards = document.querySelectorAll('.website-card')
        for (let i=0;i<cards.length;++i) {
          cards[i].classList.remove("selected")
          void cards[i].offsetWidth
        }
        for (let i=0;i<selected.length;++i) {
          selected[i].classList.add("selected")
        }
      })
    })
  },
  destroyed() {
    this.$store.state.Websites.selection.destroy()
  },
  data: () => ({
    alphabet: [
      'a','b','c','d','e','f','g','h','i','j','k','l','m','n',
      'o','p','q','r','s','t','u','v','w','x','y','z'
    ],
    swatches: [
      "#cc0e00", "#e8004f", "#ae0eff", "#2041f7", "#2196f3", "#00bcd4", "#009688", 
      "#2ac530", "#8bc34a", "#ff9800", "#ff5722", "#795548", "#9b9b9b"
    ],
    websitesPerPagePreset: [20,40,60,80,100,150,200],
    selection: null,
    previousSelection: [],
    isScrollToTopVisible: false,
  }),
  computed: {
    chars: {
      get () {
        return this.$store.state.Settings.websiteFirstChar
      },
      set (value) {
        this.$store.state.Settings.websiteFirstChar = value
        this.$store.dispatch('filterWebsites')
      },
    },
    colors: {
      get () {
        return this.$store.state.Settings.websiteColor
      },
      set (value) {
        this.$store.state.Settings.websiteColor = value
        this.$store.dispatch('filterWebsites')
      },
    },
    getNumberOfPagesLimit() {
      return this.$store.state.Settings.numberOfPagesLimit
    },
    pages() {
      return this.$store.getters.websitesPages
    },
    websitesOnPage() {
      return this.$store.getters.websitesOnPage
    },
    websitesPerPage: {
      get() {
        return this.$store.state.Settings.websitesPerPage
      },
      set(number) {
        this.$store.dispatch('changeWebsitesPerPage', number)
      },
    },
    websitesPagesSum: {
      get() {
        return this.$store.state.Websites.pageTotal
      },
      set(number) {
        this.$store.state.Websites.pageTotal = number
      },
    },
    websitesCurrentPage: {
      get() {
        return this.$store.state.Settings.websitePage
      },
      set(number) {
        this.$store.state.Settings.websitePage = number
        this.$store.dispatch('saveFiltersOfWebsites')
      },
    },
    selectedWebsitesLength() {
      return this.$store.getters.getSelectedWebsites.length
    },
    isSelectedSingleWebsite() {
      return this.$store.getters.getSelectedWebsites.length == 1
    },
    tabId() {
      return this.$route.query.tabId
    },
    gapSize() {
      return `gap-size-${this.$store.state.Settings.gapSize}`
    },
    tab() {
      if (this.tabId === 'default') {
        return undefined
      } else {
        return this.$store.getters.tabsDb.find({id: +this.tabId}).value()    
      }
    },
    filters() {
      return this.$store.state.Settings.websiteFilters
    },
  },
  methods: {
    removeFilter(i) {
      this.filters.splice(i, 1)
      this.$store.dispatch('filterWebsites')
    },
    selectedWebsites(list) {
      let ids = this.$store.getters.getSelectedWebsites
      let websites = this.$store.getters.websites
      if (ids.length!==0) {
        let names = ids.map(i=>(websites.find({id:i}).value().name))
        if (list) {
          return names.map((n,i) => (`${i+1}) ${n}`)).join('\r\n')
        } else {
          return names.join(', ')
        }
      }
    },
    clearChars() {
      this.$store.state.Settings.websiteFirstChar = []
      this.$store.dispatch('filterWebsites')
    },
    clearColors() {
      this.$store.state.Settings.websiteColor = []
      this.$store.dispatch('filterWebsites')
    },
    scrollToTop() {
      this.$refs.mainContainer.scrollTo({y: 0},500,"easeInQuad")
    },
    handleScroll(vertical) {
      if (vertical.scrollTop > 150) {
        this.isScrollToTopVisible = true
      } else this.isScrollToTopVisible = false
    },
    getSelectedWebsites(selectedWebsites){
      let ids = selectedWebsites.map(item => (item.dataset.id))
      this.$store.commit('updateSelectedWebsites', ids)
    },
    copyWebsiteNameToClipboard(){
      navigator.clipboard.writeText(this.selectedWebsites())
    },
    deleteWebsites(){
      this.previousSelection = []
      this.$store.dispatch('deleteWebsites'), 
      this.$store.state.Websites.dialogDeleteWebsite = false
    },
    initFilters(){
      let newFilters
      if (this.tabId === 'default' || typeof this.tab.filters === 'undefined') {
        newFilters = _.cloneDeep(this.$store.getters.settings.get('websiteFilters').value())
        this.$store.state.Settings.websiteSortBy = this.$store.getters.settings.get('websiteSortBy').value()
        this.$store.state.Settings.websiteSortDirection = this.$store.getters.settings.get('websiteSortDirection').value()
        this.$store.state.Settings.websitePage = this.$store.getters.settings.get('websitePage').value()
        this.$store.state.Settings.websiteFirstChar = this.$store.getters.settings.get('websiteFirstChar').value()
        this.$store.state.Settings.websiteColor = this.$store.getters.settings.get('websiteColor').value()
      } else {
        newFilters = _.cloneDeep(this.tab.filters)
        this.$store.state.Settings.websiteSortBy = this.tab.sortBy || 'name'
        this.$store.state.Settings.websiteSortDirection = this.tab.sortDirection || 'asc'
        this.$store.state.Settings.websitePage = this.tab.page || 1
        this.$store.state.Settings.websiteFirstChar = this.tab.firstChar || []
        this.$store.state.Settings.websiteColor = this.tab.color || []
      }
      this.$store.state.Settings.websiteFilters = newFilters
      this.$store.dispatch('filterWebsites', true)
    },
  },
  watch: {
    $route(newRoute) {
      if (!this.$route.path.includes('/websites/:')) return
      this.initFilters()
    },
  }
}
</script>


<style lang="less">
.websites-grid {
  padding: 10px;
  display: grid;
  grid-gap: 10px;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  grid-auto-rows: minmax(260px, auto);
  &.gap-size {
    &-xs {
      grid-gap: 10px;
    }
    &-s {
      grid-gap: 15px;
    }
    &-m {
      grid-gap: 20px;
    }
    &-l {
      grid-gap: 25px;
    }
    &-xl {
      grid-gap: 30px;
    }
  }
}
.website-card.selected {
  position: relative;
  overflow: visible;
  &:after {
    content: '';
    position: absolute;
    width: calc(100% + 8px);
    height: calc(100% + 8px);
    left: -4px;
    top: -4px;
    border-radius: 7px;
    pointer-events: none;
    border: 1.5px solid rgb(122, 122, 122);
  }
}
</style>