<template>
	<div>
    <v-tooltip bottom>
      <template v-slot:activator="{ on }">
        <v-btn @click="$store.state.Videos.dialogFilterVideos=true" v-on="on" icon tile>
          <v-badge :value="filterBadge" :content="filteredVideosTotal" overlap bottom style="z-index: 5;"> 
          <v-icon>mdi-filter</v-icon> </v-badge>
        </v-btn>
      </template>
      <span>Filter videos</span>
    </v-tooltip>

    <v-tooltip bottom>
      <template v-slot:activator="{ on }">
        <v-btn @click="resetAllFilters" icon tile v-on="on"> 
          <v-icon>mdi-filter-off</v-icon>
        </v-btn>
      </template>
      <span>Reset all filters</span>
    </v-tooltip>

    <v-tooltip bottom>
      <template v-slot:activator="{ on }">
        <v-btn @click="$store.state.Videos.dialogFolderTree = true" icon tile v-on="on"> 
          <v-badge :value="!isTreeEmpty" :content="treeBadgeContent" overlap bottom style="z-index: 5;">
            <v-icon>mdi-file-tree</v-icon>
          </v-badge>
        </v-btn>
      </template>
      <span>Open folder tree</span>
    </v-tooltip>
    
    <v-menu offset-y nudge-bottom="10" :close-on-content-click="false">
      <template #activator="{ on: onMenu }">
        <v-tooltip bottom>
          <template #activator="{ on: onTooltip }">
            <v-badge :icon="sortIcon" overlap offset-x="23" offset-y="44" class="badge-sort">
              <v-btn v-on="{ ...onMenu, ...onTooltip }" icon tile>
                <v-icon>mdi-sort-variant</v-icon>
                <v-icon v-if="sortDirection=='desc'" size="16" class="badge-sort-icon">mdi-arrow-up-thick</v-icon>
                <v-icon v-else size="16" class="badge-sort-icon">mdi-arrow-down-thick</v-icon>
              </v-btn>
            </v-badge>
          </template>
          <span>Sort Videos</span>
        </v-tooltip>
      </template>
      <v-card>
        <v-btn-toggle v-model="sortButtons" mandatory class="group-buttons-sort" color="primary">
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn outlined @click="toggleSortDirection" value="name" v-on="on">
                <v-icon>mdi-alphabetical-variant</v-icon>
                <v-icon right size="14" v-if="sortButtons=='name' && sortDirection=='desc'">
                  mdi-arrow-up-thick
                </v-icon>
                <v-icon right size="14" v-if="sortButtons=='name' && sortDirection=='asc'">
                  mdi-arrow-down-thick
                </v-icon>
              </v-btn>
            </template>
            <span>Sort by Name</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn outlined @click="toggleSortDirection" value="duration" v-on="on">
                <v-icon>mdi-timer-outline</v-icon>
                <v-icon right size="14" v-if="sortButtons=='duration' && sortDirection=='desc'">
                  mdi-arrow-up-thick
                </v-icon>
                <v-icon right size="14" v-if="sortButtons=='duration' && sortDirection=='asc'">
                  mdi-arrow-down-thick
                </v-icon>
              </v-btn>
            </template>
            <span>Sort by Duration</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
            <v-btn outlined @click="toggleSortDirection" value="size" v-on="on">
              <v-icon>mdi-harddisk</v-icon>
              <v-icon right size="14" v-if="sortButtons=='size' && sortDirection=='desc'">
                mdi-arrow-up-thick
              </v-icon>
              <v-icon right size="14" v-if="sortButtons=='size' && sortDirection=='asc'">
                mdi-arrow-down-thick
              </v-icon>
            </v-btn>
            </template>
            <span>Sort by Size</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn outlined @click="toggleSortDirection" value="rating" v-on="on">
                <v-icon>mdi-star-outline</v-icon>
                <v-icon right size="14" v-if="sortButtons=='rating' && sortDirection=='desc'">
                  mdi-arrow-up-thick
                </v-icon>
                <v-icon right size="14" v-if="sortButtons=='rating' && sortDirection=='asc'">
                  mdi-arrow-down-thick
                </v-icon>
              </v-btn>
            </template>
            <span>Sort by Rating</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn outlined @click="toggleSortDirection" value="date" v-on="on">
                <v-icon>mdi-calendar-plus</v-icon>
                <v-icon right size="14" v-if="sortButtons=='date' && sortDirection=='desc'">
                  mdi-arrow-up-thick
                </v-icon>
                <v-icon right size="14" v-if="sortButtons=='date' && sortDirection=='asc'">
                  mdi-arrow-down-thick
                </v-icon>
              </v-btn>
            </template>
            <span>Sort by Date Added</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn outlined @click="toggleSortDirection" value="edit" v-on="on">
                <v-icon>mdi-calendar-edit</v-icon>
                <v-icon right size="14" v-if="sortButtons=='edit' && sortDirection=='desc'">
                  mdi-arrow-up-thick
                </v-icon>
                <v-icon right size="14" v-if="sortButtons=='edit' && sortDirection=='asc'">
                  mdi-arrow-down-thick
                </v-icon>
              </v-btn>
            </template>
            <span>Sort by Date of Editing</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn outlined @click="toggleSortDirection" value="path" v-on="on">
                <v-icon>mdi-folder-outline</v-icon>
                <v-icon right size="14" v-if="sortButtons=='path' && sortDirection=='desc'">
                  mdi-arrow-up-thick
                </v-icon>
                <v-icon right size="14" v-if="sortButtons=='path' && sortDirection=='asc'">
                  mdi-arrow-down-thick
                </v-icon>
              </v-btn>
            </template>
            <span>Sort by Path</span>
          </v-tooltip>
        </v-btn-toggle>
      </v-card>
    </v-menu>

		<v-tooltip bottom>
			<template v-slot:activator="{ on }">
				<v-btn @click="selectAllVideos" icon tile v-on="on">
					<v-icon>mdi-select-all</v-icon>
				</v-btn>
			</template>
			<span>Select all videos</span>
		</v-tooltip>

    <FiltersPresets v-if="$store.state.Bookmarks.dialogFiltersPresets" typeOfPresets="videos"/>
    <DialogFolderTree v-if="$store.state.Videos.dialogFolderTree"/>
    <DialogFilterVideos v-if="$store.state.Videos.dialogFilterVideos"/>
  </div>
</template>


<script>
import vuescroll from 'vuescroll'

export default {
  name: 'VideosAppbarActions',
  components: {
    FiltersPresets: () => import('@/components/elements/FiltersPresets.vue'),
    DialogFolderTree: () => import('@/components/pages/videos/DialogFolderTree.vue'),
    DialogFilterVideos: () => import('@/components/pages/videos/DialogFilterVideos.vue'),
    vuescroll,
  },
  mounted() {
    this.$nextTick(function () {
    })
  },
  data: () => ({
    filtersMenu: false,
    filterPerformersLogicIcon: 'mdi-math-norm',
    filterTagsLogicIcon: 'mdi-math-norm',
  }),
  computed: {
    filterBadge() {
      let filters = _.cloneDeep(this.$store.state.Settings.videoFilters)
      if (filters.length) {
        filters = _.filter(filters, f => {
          if (f.type == null) return false 
          if (f.type=='number'||f.type=='string'||f.type=='date'||f.type=='select'||f.type=='array') {
            if (f.val.length) return true 
            else return false
          } 
          if (f.type == 'boolean') return true
        })
        return filters.length > 0
      } else return false
    },
    filteredVideosTotal() {
      let filters = _.cloneDeep(this.$store.state.Settings.videoFilters)
      if (filters.length) {
        filters = _.filter(filters, f => {
          if (f.type == null) return false 
          if (f.type=='number'||f.type=='string'||f.type=='date'||f.type=='select'||f.type=='array') {
            if (f.val.length) return true 
            else return false
          } 
          if (f.type == 'boolean') return true
        })
        return filters.length
      } else return 0
    },
    sortIcon() {
      if (this.sortButtons=='name') return 'mdi-alphabetical-variant'
      if (this.sortButtons=='duration') return 'mdi-timer-outline'
      if (this.sortButtons=='size') return 'mdi-harddisk'
      if (this.sortButtons=='rating') return 'mdi-star-outline'
      if (this.sortButtons=='date') return 'mdi-calendar-plus'
      if (this.sortButtons=='edit') return 'mdi-calendar-edit'
      if (this.sortButtons=='path') return 'mdi-folder-outline'
      return 'mdi-help'
    },
    sortButtons: {
      get() {
        return this.$store.state.Settings.videoSortBy
      },
      set(value) {
        this.$store.state.Settings.videoSortBy = value
      },
    },
    sortDirection() {
      return this.$store.state.Settings.videoSortDirection
    },
    isTreeEmpty() {
      if (this.$store.state.Videos.tree.length) {
        return false
      } else return true
    },
    treeBadgeContent() {
      return this.$store.state.Videos.tree.length  
    },
    itemId() {
      return this.$route.params.id.replace(/:/g, '')    
    },
    tabId() {
      return this.$route.query.tabId
    },
    isPerformerPage() {
      return this.$route.path.includes('/performer/:')  
    },
    isWebsitePage() {
      return this.$route.path.includes('/website/:')  
    },
  },
  methods: {
    // async pastePath() {
    //   let text = await navigator.clipboard.readText()
    //   let path = this.$store.state.Videos.filters.path
    //   if (path) {
    //     text = path + text
    //   }
    //   this.updateFiltersOfVideos('path', text)
    // },
    // async pastePerformers() {
    //   let text = await navigator.clipboard.readText()
    //   let perfs = text.split(', ')
    //   perfs = this.$store.getters.performers.filter(p=>(perfs.includes(p.name))).value()
    //   perfs = perfs.map(p=>{return p.name})
    //   if (perfs.length) {
    //     this.updateFiltersOfVideos('performers', perfs)
    //   }
    // },
    getItem(itemType) {
      return this.$store.getters[itemType].find({ id: this.itemId }).value()    
    },
    resetAllFilters() {
      if (this.isPerformerPage) {
        let item = this.getItem('performers')
        this.$store.state.Settings.videoFilters = [{
          param: 'performers',
          cond: 'all',
          val: [item.name],
          type: 'array',
          flag: null,
          lock: true,
        },{
          param: 'tags',
          cond: 'one of',
          val: [],
          type: 'array',
          flag: null,
          lock: true,
        },{
          param: 'websites',
          cond: 'one of',
          val: [],
          type: 'array',
          flag: null,
          lock: true,
        }]
      } else if (this.isWebsitePage) {
        let item = this.getItem('websites')
        this.$store.state.Settings.videoFilters = [{
          param: 'websites',
          cond: 'one of',
          val: [item.name],
          type: 'array',
          flag: null,
          lock: true,
        },{
          param: 'performers',
          cond: 'one of',
          val: [],
          type: 'array',
          flag: null,
          lock: true,
        }]
      } else {
        const locked = _.filter(this.$store.state.Settings.videoFilters, {lock: true})
        const defaults = [{param:null, cond:null, val:null, type:null, flag:null, lock:false}]
        this.$store.state.Settings.videoFilters = _.cloneDeep([ ...locked, ...defaults])
      }

      this.$store.dispatch('filterVideos')
      // this.$store.dispatch('saveFiltersOfVideos', this.$route)
    },
    toggleSortDirection() {
      this.$store.state.Settings.videoSortDirection = this.sortDirection=='asc' ? 'desc':'asc'
      setTimeout(()=>{
        this.$store.dispatch('filterVideos')
        // this.$store.dispatch('saveFiltersOfVideos', this.$route)
      },200)
    },
    selectAllVideos() {
      this.$store.state.Videos.selection.clearSelection()
      let selected = this.$store.state.Videos.selection.select('.video-card')
      this.$store.state.Videos.selection.keepSelection()
      this.getSelectedVideos(selected)
      for (let i=0;i<selected.length;++i) {
        selected[i].classList.add("selected")
      }
    },
    getSelectedVideos(selectedVideos){
      let ids = selectedVideos.map(item => (item.dataset.id))
      this.$store.commit('updateSelectedVideos', ids)
    },
  },
}
</script>