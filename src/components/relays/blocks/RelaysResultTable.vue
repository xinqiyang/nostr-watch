<template>

   <div class="pt-0 px-1 sm:px-6 lg:px-8 dark:bg-black/20 rounded-lg">
      <div class="mt-8 flex flex-col">
      <div class="overflow-x-auto">
          <div class="inline-block min-w-full align-middle" v-if="subsectionRelays.length">
            <div class="relative overflow-hidden shadow ring-1 ring-black ring-opacity-5 md:rounded-lg">
                <table class="min-w-full table-auto">
                <thead>
                    <tr>
                      <th scope="col" class="text-left" colspan="2">
                        <span class="mr-3 hidden lg:inline-block">
                          <span 
                            :class="getThemeBtnClass('compact')" 
                            @click="store.prefs.setRowTheme('compact')">compact</span>
                          <span 
                            :class="getThemeBtnClass('comfortable')" 
                            @click="store.prefs.setRowTheme('comfortable')">comfortable</span>
                          <span 
                            :class="getThemeBtnClass('spacious')" 
                            @click="store.prefs.setRowTheme('spacious')">spacious</span>
                          <a
                            v-if="!store.layout.editorExpanded"
                            class="text-left inline-block underline text-xs ml-10" 
                            @click="this.setRandomRelay" 
                            target="_blank"
                            :href="`/relay/${relayClean(randomRelay)}`">
                             random relay
                            </a>
                        </span>
                        <span v-if="subsection != 'favorite' && store.relays.getFavorites.length" class="ml-6 text-slate-600">
                          <input type="checkbox" class=" cursor-pointer relative top-0.5 mr-1" id="relays-pin-favorites" v-model="store.prefs.pinFavorites" /> 
                          <label class="cursor-pointer font-thin text-xs" for="relays-pin-favorites">
                            pin favorites
                          </label>
                        </span>

                        <!-- <span v-if="isLoggedIn() && store.user.kind3" class="ml-6 text-slate-600">
                          <input type="checkbox" @click="handleMine()" class=" cursor-pointer relative top-0.5 mr-1" id="relays-pin-favorites" v-model="store.prefs.mine" /> 
                          <label class="cursor-pointer font-thin text-xs" for="relays-pin-favorites">
                            only mine
                          </label>
                        </span> -->
                      </th>

                     
                      
                      <!-- <th scope="col" class="relative py-3.5 pl-0 pr-0 sm:pr-0" v-if="isLoggedIn()()">
                        <code class="text-xs block">Upvote</code>
                      </th> -->
                      <th v-if="!store.layout.editorIsExpanded && (store.prefs.checkNip11 || subsection === 'nips')" scope="col" class="hidden md:table-cell lg:table-cell xl:table-cell verified">
                        <code class="text-xs block">Pubkey</code>
                      </th>
                      <th scope="col" class="location text-center" v-tooltip:top.tooltip="'Detected location of Relay'">
                        <code class="text-xs block">Location</code>
                      </th>
                      <th v-if="subsection != 'favorite'" scope="col" class="uptime text-center" v-tooltip:top.tooltip="'Detected location of Relay'">
                        <code class="text-xs block">Uptime(12h)</code>
                      </th>
                      <th scope="col" class="latency text-center" v-tooltip:top.tooltip="'Relay Latency on Read'">
                        <code class="text-xs block">Avg. Latency</code>
                      </th>
                      <th v-if="!store.layout.editorIsExpanded || !isLoggedIn()" scope="col" class="hidden md:table-cell lg:table-cell xl:table-cell connect text-center" v-tooltip:top.tooltip="'Relay connection status'">
                        <code class="text-xs block">Connect</code>
                      </th>
                      <th v-if="!store.layout.editorIsExpanded || !isLoggedIn()" scope="col" class="hidden md:table-cell lg:table-cell xl:table-cell first-line:read text-center" v-tooltip:top.tooltip="'Relay read status'">
                        <code class="text-xs block">Read</code>
                      </th>

              
                      <th v-if="(!store.layout.editorIsExpanded || !isLoggedIn())" scope="col" class="hidden md:table-cell lg:table-cell xl:table-cell write text-center" v-tooltip:top.tooltip="'Relay write status'">
                        <code class="text-xs block">Write</code>
                      </th>

                      <th v-if="store.layout.editorIsExpanded && isLoggedIn()" scope="col" class="w-16 hidden md:table-cell lg:table-cell xl:table-cell verified">
                        <!-- <span class="verified-shape-wrapper">
                          <span class="shape verified"></span>
                        </span> -->
                        <code class="text-xs block">Read</code>
                      </th>

                      <th v-if="(store.layout.editorIsExpanded && isLoggedIn())" scope="col" class="w-16 hidden md:table-cell lg:table-cell xl:table-cell verified">
                        <!-- <span class="verified-shape-wrapper">
                          <span class="shape verified"></span>
                        </span> -->
                        <code class="text-xs block">Write</code>
                      </th>

                      <th scope="col" class="relative py-3.5 pl-0 pr-0 sm:pr-0">
                        <code class="text-xs block">Favorite</code>
                      </th>
                    </tr>
                </thead>
                <tbody class="divide-y">
                    <tr 
                      v-for="(relay, index) in subsectionRelays"  
                      :key="generateKey(relay, 'aggregate')" 
                      :class="getResultClass(relay, index)" 
                      class="h-1">

                      <td class="status-indicator w-2 text-left pr-2">
                        <span :class="getAggregateIndicator(relay)" class="block relative">
                        </span>
                      </td>

                      <td class="w-62 relay left-align relay-url text-black/20 dark:text-white/20 hover:text-black/50 hover:dark:text-white/50">
                          <a class="text-black dark:text-white" :href="`/relay/${relayClean(relay)}`">{{ relay.replace('wss://', '') }}</a>
                          <span class="text-inherit flex-1 align-middle hidden lg:inline-block pl-3 m1-3 text-sm whitespace-nowrap truncate" v-if="results[relay]?.topics">
                            {{ getTopics(relay) }}
                          </span>
                      </td>

                      <!-- <td class="w-16 fav text-center" v-if="isLoggedIn()()">
                        <a
                          class=" hover:opacity-100 cursor-pointer opacity-20" 
                          @click="likeRelay(relay)">
                          👍
                        </a>
                      </td> -->

                      <td v-if="!store.layout.editorIsExpanded && (store.prefs.checkNip11 || subsection === 'nips')" class="w-12 verified text-center hidden md:table-cell lg:table-cell xl:table-cell">
                        <!-- {{ this.results[relay]?.pubkeyValid }}
                        {{ this.results[relay]?.info?.pubkey }} -->
                        <span 
                          v-if="this.results[relay]?.pubkeyValid" 
                          v-tooltip:bottom.tooltip="`Valid pubkey was registered by relay: ${this.results[relay].info.pubkey}`" class="cursor-pointer">
                          <svg class="svg-icon fill-green-500" style="width: 1em; height: 1em;vertical-align: middle;;overflow: hidden;" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg"><path d="M969.530368 512l-123.871232 89.4464 62.595072 139.424768-152.020992 15.362048-15.254528 152.062976-139.446272-62.596096L512 969.530368l-89.43616-123.830272-139.435008 62.596096-15.254528-152.062976-152.052736-15.362048 62.595072-139.424768L54.470656 512l123.860992-89.435136-62.595072-139.436032 152.052736-15.361024 15.255552-152.052736 139.435008 62.595072L512 54.469632l89.4464 123.840512 139.424768-62.595072 15.254528 152.052736 152.042496 15.361024L845.574144 422.56384 969.530368 512z"  /></svg>
                        </span> 
                          <!-- <span v-tooltip:top.tooltip="identityList(relay)"> <span class="verified-shape-wrapper cursor-pointer" v-if="Object.entries(results[relay]?.identities).length"><span class="shape verified"></span></span></span> -->
                        <span 
                          v-if="!this.results[relay]?.pubkeyValid && this.results[relay]?.info?.pubkey" class="cursor-pointer" 
                          v-tooltip:bottom.tooltip="`Provided pubkey ${this.results[relay].info.pubkey} is invalid, error: ${this.results[relay].pubkeyError}. Please review NIP-11 specification to fix.`">
                          <svg class="svg-icon fill-red-500/20" style="width: 1em; height: 1em;vertical-align: middle;;overflow: hidden;" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg"><path d="M969.530368 512l-123.871232 89.4464 62.595072 139.424768-152.020992 15.362048-15.254528 152.062976-139.446272-62.596096L512 969.530368l-89.43616-123.830272-139.435008 62.596096-15.254528-152.062976-152.052736-15.362048 62.595072-139.424768L54.470656 512l123.860992-89.435136-62.595072-139.436032 152.052736-15.361024 15.255552-152.052736 139.435008 62.595072L512 54.469632l89.4464 123.840512 139.424768-62.595072 15.254528 152.052736 152.042496 15.361024L845.574144 422.56384 969.530368 512z"  /></svg>
                        </span>
                      </td>

                      <td class="w-24 location text-center">
                        {{ getFlag(relay) }}
                      </td>

                      <td v-if="subsection != 'favorite'" class="w-24 latency text-center">
                        <span class="sm:px-6 text-sm font-bold h-full" :class="getUptimeColor(relay)" v-if="this.results[relay]?.uptime">
                          {{ this.results[relay]?.uptime }}%
                        </span>
                      </td>

                      <!-- <td class="w-24 latency text-center">
                        <div class="px-4 py-5 sm:px-6 flex text-sm font-bold">
                          <span 
                            v-for="heartbeat in this.store.stats.getHeartbeat(relay)"
                            :key="heartbeat[0]"
                            class="mr-0 w-0.5 h-5 flex-1"
                            :class="{
                              'bg-red-700': !heartbeat.latency,
                              'bg-green-500': heartbeat.latency
                            }">
                            </span>
                        </div>
                      </td> -->

                      <!-- <td class="w-24 latency text-center">
                        <span>{{ results[relay]?.latency?.final }}<span v-if="results[relay]?.check?.latency">ms</span></span>
                      </td> -->

                      <td class="w-24 latency text-center text-sm font-bold">
                        <span v-if="results[relay]?.latency?.average">{{ results[relay].latency.average }}<span v-if="results[relay].latency.average">ms</span></span>
                        <span v-if="!results[relay]?.latency?.average" class="font-normal italic text-black/30 dark:text-white/30">timeout</span>
                      </td>

                      <!-- no editor -->
                      <td v-if="!store.layout.editorIsExpanded || !isLoggedIn()" class="w-16 content-center text-center hidden md:table-cell lg:table-cell xl:table-cell" :key="generateKey(relay, 'check.connect')">
                        <span class="m-auto block" :class="getCheckIndicator(relay, 'connect')">
                          &nbsp;
                        </span>
                      </td>

                      <td v-if="!store.layout.editorIsExpanded || !isLoggedIn()" class="w-16 content-center text-center hidden md:table-cell lg:table-cell xl:table-cell" :key="generateKey(relay, 'check.read')">
                        <span class="m-auto block align-middle" :class="getCheckIndicator(relay, 'read')">
                          <span class="align-middle h-max" v-if="isLoggedIn() && store.user.kind3?.[relay]?.read">
                            <svg class="inline-block " :class="getCheckIndicatorPolicy" fill="none" stroke="rgba(0,0,0,0.5)" stroke-width="3" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                              <path stroke-linecap="round" stroke-linejoin="round" d="M9 12.75L11.25 15 15 9.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                            </svg>
                          </span>
                          <span v-else>&nbsp;</span>
                        </span>
                      </td>

                      <td v-if="(!store.layout.editorIsExpanded || !isLoggedIn())" class="w-16 content-center text-center hidden md:table-cell lg:table-cell xl:table-cell" :key="generateKey(relay, 'check.write')">
                        <span class="m-auto block align-middle" :class="getCheckIndicator(relay, 'write')">
                          <span class="align-middle" v-if="isLoggedIn() && store.user.kind3?.[relay]?.write">
                            <svg class="inline-block" :class="getCheckIndicatorPolicy" fill="none" stroke="rgba(0,0,0,0.5)" stroke-width="3" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                              <path stroke-linecap="round" stroke-linejoin="round" d="M9 12.75L11.25 15 15 9.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                            </svg>
                          </span>
                          <span v-else>&nbsp;</span>
                        </span>
                      </td>

                      <!-- editor -->
                      <td v-if="store.tasks.getActiveSlug != 'user/relay/list' 
                                && store.layout.editorIsExpanded 
                                && typeof store.user.kind3?.[relay]?.read !== `undefined`
                                && isLoggedIn()"
                          class="text-center md:table-cell lg:table-cell xl:table-cell">
                        <Switch
                          v-model="store.user.kind3[relay].read" 
                          class="group relative inline-flex h-5 w-10 flex-shrink-0 cursor-pointer items-center justify-center rounded-full focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
                          <span class="sr-only">Use setting</span>
                          <span aria-hidden="true" class="pointer-events-none absolute h-full w-full rounded-md bg-white dark:bg-black/10" />
                          <span aria-hidden="true" :class="[store.user.kind3[relay].read ? 'bg-indigo-600' : 'bg-gray-200 dark:bg-black', 'pointer-events-none absolute mx-auto h-4 w-9 rounded-full transition-colors duration-200 ease-in-out']" />
                          <span aria-hidden="true" :class="[store.user.kind3[relay].read ? 'translate-x-5' : 'translate-x-0', 'pointer-events-none absolute left-0 inline-block h-5 w-5 transform rounded-full border border-gray-200 bg-white shadow ring-0 transition-transform duration-200 ease-in-out']" />
                        </Switch>
                      </td>

                      <td v-if="store.tasks.getActiveSlug != 'user/relay/list' 
                                && store.layout.editorIsExpanded 
                                && typeof store.user.kind3?.[relay]?.write !== `undefined` 
                                && isLoggedIn()"
                        class="text-center md:table-cell lg:table-cell xl:table-cell">
                        <Switch
                          v-model="store.user.kind3[relay].write" 
                          class="group relative inline-flex h-5 w-10 flex-shrink-0 cursor-pointer items-center justify-center rounded-full 
                          focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
                          <span class="sr-only">Use setting</span>
                          <span aria-hidden="true" class="pointer-events-none absolute h-full w-full rounded-md bg-white dark:bg-black/10" />
                          <span aria-hidden="true" :class="[store.user.kind3[relay].write ? 'bg-indigo-600' : 'bg-gray-200 dark:bg-black', 'pointer-events-none absolute mx-auto h-4 w-9 rounded-full transition-colors duration-200 ease-in-out']" />
                          <span aria-hidden="true" :class="[store.user.kind3[relay].write ? 'translate-x-5' : 'translate-x-0', 'pointer-events-none absolute left-0 inline-block h-5 w-5 transform rounded-full border border-gray-200 bg-white shadow ring-0 transition-transform duration-200 ease-in-out']" />
                        </Switch>
                      </td>
                      
                      <td 
                        colspan="2" 
                        v-if="store.layout.editorExpanded && store.tasks.getActiveSlug == 'user/relay/list'" 
                        class="w-auto text-center md:table-cell lg:table-cell xl:table-cell">
                        <svg class="animate-spin mr-1 -mt-0.5 h-4 w-5 text-white inline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                        </svg>
                      </td>

                      <td 
                        colspan="2" 
                        v-if="store.layout.editorExpanded && !store.relays.isFavorite(relay) && store.tasks.getActiveSlug != 'user/relay/list'" 
                        class="w-auto text-center md:table-cell lg:table-cell xl:table-cell">
                      </td>

                      <td  class="w-16 fav text-center">
                        <a
                          class="hover:opacity-100 cursor-pointer" 
                          :class="store.relays.isFavorite(relay) ? 'opacity-100' : 'opacity-10'"
                          @click="store.relays.toggleFavorite(relay)">
                          ❤️
                        </a>
                      </td>
                      
                    </tr>
                </tbody>
                </table>
            </div>
          </div>
          <div class="text-xl block min-w-full align-middle py-8 light:bg-slate-100 rounded-lg" v-if="!store.relays.getFavorites.length && subsection == 'favorite'">
            You have not selected any favorites. To select a favorite, click the heart emoji next to any relay in a relays list. 
            Alternatively, sign-in to sync your relays list. If you do not see a sign-in, download a NIP-04 compatible extension like nos2x or Alby
          </div>
      </div>
      </div>
    </div>
  </template>
  
  <script>
  import { defineComponent, toRefs } from 'vue'
  import { countryCodeEmoji } from 'country-code-emoji';
  import emoji from 'node-emoji';
  import crypto from 'crypto'
  import { Switch } from '@headlessui/vue'

  // import SingleClearnet from '@/components/relays/SingleClearnet.vue'
  
  import RelaysLib from '@/shared/relays-lib.js'
  import UserLib from '@/shared/user-lib.js'
  // import { screenIs } from '@/shared/layout.js'

  import { validateEvent, getEventHash, verifySignature } from 'nostr-tools'
  
  import { setupStore } from '@/store'
  
  const localMethods = {
    setRandomRelay(){
      this.randomRelay = this.store.relays.getShuffledPublic[0]
    },
    handleMine(){
      setTimeout( () => {
        if(this.store.prefs.mine)
          this.store.layout.setActive('relays/find', 'favorite')
      },1)
    },
    async likeRelay(relay){
      const id = this.store.relays.getCanonical(relay)
      const event = {
        created_at: Math.floor(Date.now()/1000),
        kind: 7,
        content: '+',
        tags: [
          ['e', id],
          ['p', this.store.user.getPublicKey]
        ],
        pubkey: this.store.user.getPublicKey
      }
      event.id = getEventHash(event)

      //console.log('like event', event)
      const signedEvent = await window.nostr.signEvent(event)

      let ok = validateEvent(signedEvent)
      let veryOk = await verifySignature(signedEvent)

      if(!ok || !veryOk)
        return
      
      return true
      //console.log('valid event?', ok, veryOk)
    },   
  }
  export default defineComponent({
    name: 'RelaysResultTable',
    components: {
      Switch,
    },
    setup(props){
      const {subsectionProp: subsection} = toRefs(props)
      const {resultsProp: results} = toRefs(props)
      const {relaysProp: relays} = toRefs(props)
      return { 
        store : setupStore(),
        results: results,
        subsection: subsection,
        relays: relays,
      }
    },
    beforeMount(){
      // this.foundNips = this.collateSupportedNips()
    },
    mounted(){
      //console.log('navdata', this.navData, this.navData.filter( item => item.slug == this.subsection )[0], this.navData.filter( item => item.slug == this.subsection ))
      this.activePageData = this.navData.filter( item => item.slug == this.subsection )[0]
      this.setRandomRelay()
    },
    updated(){
      
    },
    beforeUnmount(){
    },
    unmounted(){
      delete this.results
    },
    props: {
      subsectionProp: {
        type: String,
        default(){
          return ""
        }
      },
      resultsProp: {
        type: Object,
        default(){
          return {}
        }
      },
      relaysProp: {
        type: Array,
        default(){
          return []
        }
      },
      relaysCountProp: {
        type: Object,
        default(){
          return {}
        }
      },
    },
    data() {
      return {
        selectedRelays: [],
        timeout: null,
        navData: this.store.layout.getNavGroup('relays/find'),
        activePageData: {},
        randomRelay: "",
        inputDetected: false,
        swearFilter: null,
        foundNips: null
      }
    },
    computed: {
      getTopics(){
        return relay => {
          let topics = ""
          // let topicsArr = this.results[relay].topics.slice(0, 3).filter( topic => topic[0].length <= 32)
          let topicsArr = this.results[relay].topics.filter( topic => {
            //console.log(topic[0], topic[0].length, topic[0].length <= 32)
            return topic[0].length <= 32
          }).slice(0, 3)
          for(let topic in topicsArr){
            topics = `${topics}  #${topicsArr[topic][0]}`
          }
          return topics
        }
      },
      getUptimeColor(){
        return relay => {
          return {
            'text-green-600/100 dark:text-green-600/80': this.results[relay]?.uptime >= 98,
            'text-green-600/80 dark:text-green-400/50': this.results[relay]?.uptime >= 95 && this.results[relay]?.uptime < 98,
            'text-yellow-600 dark:text-yellow-400/90': this.results[relay]?.uptime >= 90 && this.results[relay]?.uptime < 95,
            'text-orange-500': this.results[relay]?.uptime >= 80 && this.results[relay]?.uptime < 90,
            'text-red-400 dark:text-red-600': this.results[relay]?.uptime < 80,
          }
        }
      },
      // getUptimePerc(){
      //   return (relay) => {
      //     const pulses = this.store.stats.getHeartbeat(relay)
      //     if(!pulses || !Object.keys(pulses).length )
      //       return ""
      //     const totalHeartbeats = Object.keys(pulses).length 
      //     const totalOnline = Object.entries(pulses).reduce(
      //         (acc, value) => value[1].latency ? acc+1 : acc,
      //         0
      //     );
      //     const perc = Math.floor((totalOnline/totalHeartbeats)*100)
      //     return `${perc}%`
      //   }
      // },
      subsectionRelays(){
        // return this.getRelays( this.store.relays.getRelays(this.subsection, this.results ) )
        return this.getRelays( this.relays )
      },
      relayGeo(){
        return (relay) => this.store.relays.getGeo(relay)
      },
      getThemeBtnClass(){
        return (key) => {
          return {
            'border-1 border-bottom text-black bg-black/5 dark:text-white dark:bg-black/50 rounded-sm py-1': this.store.prefs.getTheme === key,
            'text-slate-400': this.store.prefs.getTheme !== key,
            'text-xs px-3 mr-1 cursor-pointer': true,
          }
        }
      },
      getResultClass() {
        return (relay, index) => {
          //console.log(this.store.prefs.getTheme)
          return {
            loaded: this.results[relay]?.state == 'complete',
            'bg-slate-100': index % 2,
            'bg-red-50 hover:bg-red-100 dark:bg-red-800/10 dark:hover:bg-red-100/5': this.store.relays.isFavorite(relay),
            'bg-gray-50 hover:bg-slate-200 dark:bg-transparent dark:hover:bg-slate-200/5': !this.store.relays.isFavorite(relay),
            'xl:text-2xl xl:h-16': this.store.prefs.getTheme === 'spacious',
            'xl:text-xl xl:h-9': this.store.prefs.getTheme === 'comfortable',
            // '': this.store.prefs.getTheme === 'compact',
          }
        }
      },
      getCheckIndicator(){
        return (relay, key) => {
          return { 
            'bg-green-500': this.results[relay]?.check?.[key] !== false,
            'bg-red-500': this.results[relay]?.check?.[key] === false,
            'bg-gray-500': 'undefined' === typeof this.results[relay]?.check?.[key],
            // '': this.store.prefs.getTheme === 'spacious',
            // '': this.store.prefs.getTheme === 'comfortable',
            'text-2xl block m-auth h-6 w-6 rounded-xl': this.store.prefs.getTheme === 'spacious',
            'text-xl block m-auth h-5 w-5 rounded-2xl': this.store.prefs.getTheme === 'comfortable',
            'text-xl block m-auth h-4 w-4 rounded-2xl': this.store.prefs.getTheme === 'compact',
            // 'success': this.results[relay]?.check?.[key] !== false,
            // 'failure': this.results[relay]?.check?.[key] === false,
            // 'pending': 'undefined' === typeof this.results[relay]?.check?.[key] 
            }
        } 
      },
      getCheckIndicatorPolicy(){
        return {
          "h-5 w-5 mt-0.5": this.store.prefs.getTheme === 'spacious',
          "h-4 w-4 mt-0.5": this.store.prefs.getTheme === 'comfortable',
          "h-4 w-4": this.store.prefs.getTheme === 'compact',
        }
      },
      getAggregateIndicator(){
        return (relay) => {
          return { 
            'w-4 h-4 bg-green-500': this.results[relay]?.aggregate === 'public',
            'w-4 h-4 bg-orange-500': this.results[relay]?.aggregate === 'restricted',
            'w-4 h-4 bg-red-500': this.results[relay]?.aggregate === 'offline',
            'ml-4': this.store.prefs.getTheme === 'spacious',
            'ml-2': this.store.prefs.getTheme === 'comfortable',
            'ml-1': this.store.prefs.getTheme === 'compact',
            // '': this.store.prefs.getTheme === 'spacious',
            // '': this.store.prefs.getTheme === 'comfortable',
            // 'text-2xl h-16 block m-auth h-6 w-6 rounded-xl': this.store.prefs.getTheme === 'spacious',
            // 'text-xl block m-auth h-5 w-5 rounded-2xl': this.store.prefs.getTheme === 'comfortable',
            // 'text-xl block m-auth h-4 w-4 rounded-2xl': this.store.prefs.getTheme === 'compact',
            // 'success': this.results[relay]?.check?.[key] !== false,
            // 'failure': this.results[relay]?.check?.[key] === false,
            // 'pending': 'undefined' === typeof this.results[relay]?.check?.[key] 
            }
        } 
      },
      generateKey(){
        return (url, key) => crypto.createHash('md5').update(`${url}_${key}`).digest('hex')
      },
      getFlag () {
        return (relay) => this.relayGeo(relay)?.countryCode ? countryCodeEmoji(this.relayGeo(relay)?.countryCode) : emoji.get('shrug');
      },
      identityList () {
        return (relay) => {
          let string = '',
              extraString = '',
              users = Object.entries(this.results[relay]?.identities),
              count = 0

          if(this.results[relay]?.identities) {
            if(this.results[relay]?.identities.serverAdmin) {
              string = `Relay has registered an administrator pubkey: ${this.results[relay]?.identities.serverAdmin}. `
              extraString = "Additionally, "
            }

            const total = users.filter(([key]) => key!='serverAdmin').length,
                  isOne = total==1

            if(total) {
              string = `${string}${extraString}Relay domain contains NIP-05 verification data for:`
              users.forEach( ([key]) => {
                if(key == "serverAdmin") return
                count++
                string = `${string} ${(count==total && !isOne) ? 'and' : ''}  @${key}${(count!=total && !isOne) ? ', ' : ''}`
              })
            }
          }
          return string
        }
      },
      relayClean() {
        return (relay) => relay?.replace('wss://', '')
      },
    },
    methods: Object.assign(RelaysLib, UserLib, localMethods),
  })
  </script>

<!-- <style lang=“postcss”>
.indicator {
  @apply h-4 w-4 block rounded-lg
}
</style> -->
  
  <style lang='css' scoped>
    

    /* table {
      border-collapse: collapse !important;
    } */
  
    .section-json {
      font-size:13px;
      color: #555;
      cursor:pointer;
    }
  
    ::v-deep(.modal-container) {
      display: flex;
      justify-content: center;
      align-items: center;
    }
    ::v-deep(.modal-content) {
      position: relative;
      display: flex;
      flex-direction: Column;
      max-height: 90%;
      max-width:800px;
      margin: 0 1rem;
      padding: 1rem;
      border: 1px solid #e2e8f0;
      border-radius: 0.25rem;
      background: #fff;
    }
    .modal__title {
      margin: 0 2rem 0 0;
      font-size: 1.5rem;
      font-weight: 700;
    }
    .modal__content {
      flex-grow: 1;
      overflow-y: auto;
    }
    .modal__action {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-shrink: 0;
      padding: 1rem 0 0;
    }
    .modal__close {
      position: absolute;
      top: 0.5rem;
      right: 0.5rem;
    }
  
    .nip-11 a { cursor: pointer }
  
    tr.even {
      background:#f3f3f3;
    }

    tr {
      border-top:none;
    }
    </style>