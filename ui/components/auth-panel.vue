<template>
  <client-only>
    <v-toolbar-items>
      <v-btn
        v-if="!isAuthenticated"
        color="secondary"
        rounded
        to="/@/auth/register"
        style="border-radius: 28px;"
      >
        Sign up, it's free
      </v-btn>

      <v-divider v-if="!isAuthenticated" vertical />

      <v-btn
        v-if="!isAuthenticated"
        text
        to="/@/auth/login"
      >
        Login
      </v-btn>

      <v-divider vertical />

      <v-menu
        v-if="totalUnreads"
        v-model="messagesMenu"
        open-on-hover
        bottom
        offset-y
      >
        <template v-slot:activator="{ on }">
          <v-btn
            outlined
            small
            dark
            style="background-color: #303030"
            v-on="on"
          >
            <v-badge
              :color="totalMentions ? 'red' : 'orange'"
              left
              bottom
              overlap
            >
              <template v-slot:badge>
                {{ totalUnreads }}
              </template>
              <v-icon>mdi-email</v-icon>
            </v-badge>
          </v-btn>
        </template>
        <v-list>
          <template v-for="(item, index) in messageNotifications">
            <v-list-item
              :key="item.room.path"
              @click="goToRoom(item.room.path)"
            >
              <v-list-item-icon>
                <v-badge
                  :color="item.mentions ? 'red' : 'orange'"
                  left
                  bottom
                  overlap
                >
                  <template v-slot:badge>
                    {{ item.unreads }}
                  </template>
                  <v-icon>mdi-email</v-icon>
                </v-badge>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>
                  <span>{{ item.room.path }}</span>
                </v-list-item-title>
              </v-list-item-content>
              <v-list-item-action v-if="item.mentions">
                <v-list-item-action-text>
                  <v-icon small color="red">
                    fa-at
                  </v-icon>
                </v-list-item-action-text>
              </v-list-item-action>
            </v-list-item>
            <v-divider :key="`message_divider_${index}`" />
          </template>
        </v-list>
      </v-menu>

      <v-menu
        v-if="Object.keys(calls).length"
        v-model="callsMenu"
        open-on-hover
        bottom
        offset-y
      >
        <template v-slot:activator="{ on }">
          <v-btn
            outlined
            small
            dark
            style="background-color: #303030"
            v-on="on"
          >
            <v-badge
              color="red"
              left
              bottom
              overlap
            >
              <template v-slot:badge>
                {{ Object.keys(calls).length }}
              </template>
              <v-icon>fa fa-phone</v-icon>
            </v-badge>
          </v-btn>
        </template>

        <v-list>
          <template v-for="(value, propName, index) in calls">
            <v-list-item
              :key="propName"
              @click="goToRoomCalls(getRoom(propName))"
            >
              <v-list-item-icon>
                <v-badge
                  color="red"
                  left
                  bottom
                  overlap
                >
                  <template v-slot:badge>
                    {{ value.length }}
                  </template>
                  <v-icon>fa fa-phone</v-icon>
                </v-badge>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>
                  <span>{{ getRoom(propName) ? getRoom(propName).path : '' }}</span>
                </v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-divider :key="`call_divider_${index}`" />
          </template>
        </v-list>
      </v-menu>

      <v-menu
        v-if="isAuthenticated"
        v-model="profileMenu"
        open-on-hover
        bottom
        offset-y
      >
        <template v-slot:activator="{ on }">
          <v-btn
            text
            right
            depressed
            style="min-width: 255px"
            v-on="on"
          >
            <v-icon>mdi-dots-vertical</v-icon>
            {{ user.username }}
          &nbsp;
            <v-badge
              :color="isOnline() ? 'green' : 'grey'"
              bordered
              bottom
              left
              dot
              offset-x="10"
              offset-y="10"
            >
              <v-avatar
                size="36px"
              >
                <img
                  v-if="avatarUrl"
                  :src="avatarUrl"
                  alt="Avatar"
                >
                <v-icon
                  v-else
                >
                  fa-user
                </v-icon>
              </v-avatar>
            </v-badge>
          </v-btn>
        </template>
        <v-list>
          <v-list-item @click="goToProfile">
            <v-list-item-title>
              <v-icon>fa-user</v-icon> Profile
            </v-list-item-title>
          </v-list-item>
          <v-list-item @click="toggleTheme">
            <v-list-item-title>
              <v-switch v-model="toggle" label="Toggle Dark Mode" />
            </v-list-item-title>
          </v-list-item>
          <v-divider />
          <v-list-item v-if="permission !== 'N/A'" @click="toggleSubscribe">
            <v-list-item-title>
              <v-icon>{{ subscribeIcon }}</v-icon> {{ subscribeTitle }}
            </v-list-item-title>
          </v-list-item>
          <v-divider />
          <v-list-item @click="goToCreateRoom">
            <v-list-item-title>
              <v-icon>fa-plus</v-icon> Create new room
            </v-list-item-title>
          </v-list-item>
          <v-list-item @click="goToExplore">
            <v-list-item-title>
              <v-icon>fa-search</v-icon> Explore public rooms
            </v-list-item-title>
          </v-list-item>
          <v-divider />
          <v-list-item @click="resetUsername">
            <v-list-item-title>
              <v-icon>fa-fingerprint</v-icon> Reset username
            </v-list-item-title>
          </v-list-item>
          <v-list-item @click="resetPassword">
            <v-list-item-title>
              <v-icon>fa-key</v-icon> Reset password
            </v-list-item-title>
          </v-list-item>
          <v-divider />
          <v-list-item v-if="isAdmin" @click="goToVisits">
            <v-list-item-title>
              <v-icon>fa-eye</v-icon> Visits
            </v-list-item-title>
          </v-list-item>
          <v-list-item v-if="isAdmin" @click="goToReports">
            <v-list-item-title>
              <v-icon>fa-chart-line</v-icon> Reports
            </v-list-item-title>
          </v-list-item>
          <v-list-item @click="logout">
            <v-list-item-title>
              <v-icon>fa-power-off</v-icon> Log out
            </v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>

      <v-divider v-if="user && isAuthenticated && !isActivated" vertical />

      <v-btn v-if="user && isAuthenticated && !isActivated" text @click="resetAccount()">
        Activate
      </v-btn>
    </v-toolbar-items>
  </client-only>
</template>

<script>
import {
  handleSuccess
} from '@/services/ajax-handlers'
// import { colorUtils } from '@/utils/color-utils'
import { storage } from '@/services/storage'
import { base64Utils } from '@/utils/base64-utils'

export default {
  data () {
    return {
      subscription: null,
      toggle: true,
      profileMenu: false,
      messagesMenu: false,
      callsMenu: false
    }
  },
  computed: {
    isAuthenticated () {
      return this.$store.getters['api/auth/isAuthenticated']
    },
    isActivated () {
      return this.$store.getters['api/auth/isActivated']
    },
    avatarUrl () {
      return this.$store.getters['api/auth/avatarUrl']
    },
    user () {
      return this.$store.state.api.auth.user
    },
    isAdmin () {
      return this.$store.state.api.auth.isAdmin
    },
    rooms () {
      return this.$store.state.api.room.rooms
    },
    calls () {
      return this.$store.getters['api/room/calls/callsByRoom']
    },
    isDark () {
      return this.$vuetify.theme.dark
    },
    theme () {
      return this.$vuetify.theme.isDark ? 'dark' : 'light'
    },
    messageNotifications () {
      const self = this
      let result = []
      if (this.user) {
        result = this.rooms
          .map((room) => {
            return {
              room,
              unreads: room && room._id ? self.$store.getters['api/message/unreads'](room._id).length : 0,
              mentions: room && room._id ? self.$store.getters['api/message/mentions'](room._id, self.user._id).length : 0
            }
          })
          .filter(r => r.unreads > 0)
      }
      return result
    },
    totalUnreads () {
      const result = this.messageNotifications.reduce((a, b) => a + b.unreads, 0)
      return result
    },
    totalMentions () {
      return this.messageNotifications.reduce((a, b) => a + b.mentions, 0)
    },
    permission () {
      if (typeof window !== 'undefined' && 'Notification' in window) {
        return Notification.permission
      }
      return 'N/A'
    },
    subscribeIcon () {
      return !this.subscription ? 'fa-bell' : 'fa-bell-slash'
    },
    subscribeTitle () {
      if (this.subscription) {
        return 'Stop Receiving Notifications'
      }
      if (this.permission === 'denied') {
        return 'Notifications are blocked'
      } else {
        return 'Start Receiving Notifications'
      }
    }
  },
  async mounted () {
    const sw = await navigator.serviceWorker.ready
    this.subscription = await sw.pushManager.getSubscription()
  },
  methods: {
    isOnline () {
      return this.$store.getters['api/auth/isOnline'](this.$store.state.api.auth.user._id)
    },
    getRoom (id) {
      return this.$store.getters['api/room/getRoom'](id)
    },
    getUser (id) {
      return this.$store.getters['api/auth/getUser'](id)
    },
    goToProfile () {
      this.$router.push({ path: `/@/${this.user.username}` })
    },
    toggleTheme () {
      this.$vuetify.theme.dark = !this.$vuetify.theme.dark
      storage.setLocal('theme', this.theme)
    },
    goToRoom (room) {
      this.$router.push({ path: `/${room}` })
    },
    goToRoomCalls (room) {
      if (room && room.path) {
        this.$router.push({ path: `/${room.path}/calls` })
      }
    },
    goToCreateRoom () {
      this.$router.push({ path: '/@/room/create' })
    },
    goToExplore () {
      this.$router.push({ path: '/explore/rooms' })
    },
    async resetAccount () {
      await this.$store.dispatch('api/auth/reset', {
        email: this.$store.state.api.auth.user.email,
        type: 'user_activation'
      })
      handleSuccess('Activation link was send. Please check your email for further instructions.', this.$store.commit)
    },
    async resetUsername () {
      await this.$store.dispatch('api/auth/reset', {
        email: this.$store.state.api.auth.user.email,
        type: 'username_reset'
      })
      handleSuccess('Username was reset. Please check your email for further instructions.', this.$store.commit)
    },
    async resetPassword () {
      await this.$store.dispatch('api/auth/reset', {
        email: this.$store.state.api.auth.user.email,
        type: 'password_reset'
      })
      handleSuccess('Password was reset. Please check your email for further instructions.', this.$store.commit)
    },
    async goToVisits () {
      await this.$router.push({ path: '/admin/reports?status=open&panel=3' })
    },
    async goToReports () {
      await this.$router.push({ path: '/admin/reports?panel=3' })
    },
    async logout () {
      await this.$router.push({ path: '/' })
      await this.$store.dispatch('api/auth/logout')
      await this.$store.dispatch('connectWebSocket')
    },
    async toggleSubscribe () {
      if ('Notification' in window) {
        if (Notification.permission !== 'denied') {
          console.log(this.subscription)
          if (!this.subscription) {
            const result = await Notification.requestPermission()
            if (result === 'denied') {
              alert('Notifications blocked. Please enable them in your browser.')
              return
            }
            if (!('serviceWorker' in navigator)) {
              return
            }
            const sw = await navigator.serviceWorker.ready
            const sub = await sw.pushManager.getSubscription()
            if (sub === null) {
              const config = this.$store.state.api.config.config
              this.subscription = await sw.pushManager.subscribe({
                userVisibleOnly: true,
                applicationServerKey: base64Utils.urlB64ToUint8Array(config.webPushSettings.publicKey)
              })
              await this.$store.dispatch('api/subscription/create', this.subscription.toJSON(), { root: true })
            }
            // eslint-disable-next-line no-new
            new Notification('You have subscribed to notifications successfully! You can unsubscribe anytime')
          } else {
            const sw = await navigator.serviceWorker.ready
            const sub = await sw.pushManager.getSubscription()
            if (sub !== null) {
              const payload = sub.toJSON()
              const success = await sub.unsubscribe()
              if (success) {
                await this.$store.dispatch('api/subscription/delete', payload, { root: true })
                this.subscription = null
              }
            }
            // eslint-disable-next-line no-new
            new Notification('You have unsubscribed to notifications successfully! You can subscribe again anytime')
          }
        } else {
          alert('Notifications blocked. Please enable them in your browser.')
        }
      } else {
        alert('Notification is not supported by your browser! 😞')
      }
    }
  }
}
</script>
