<template>
  <div class="loading searchbar-ignore" v-if="!ready">
    <div class="list media-list margin-left searchbar-ignore">
      <ul>
        <li class="item-content skeleton-text searchbar-ignore">
          <div class="item-media searchbar-ignore">
            <div class="skeleton-block searchbar-ignore" style="width: 40px; height: 40px; border-radius: 50%" />
          </div>
          <div class="item-inner searchbar-ignore">
            <div class="item-title-row searchbar-ignore">
              <div class="item-title searchbar-ignore">
                Item Title
              </div>
            </div>
            <div class="item-subtitle searchbar-ignore">
              Item Subtitle
            </div>
            <div class="item-text searchbar-ignore">
              Item text goes here, and it will be rendered as gray box too.
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
  <f7-list media-list v-else inset class="margin-left searchbar-ignore">
    <f7-list-group v-if="links">
      <f7-list-item
        v-for="link in links" :key="link.itemName"
        media-item link
        class="channellist-linkeditem searchbar-ignore"
        @click="configureLink(link)"
        :title="(link.item.label) ? link.item.label : link.item.name"
        :footer="(link.item.label) ? link.item.name : '\xa0'"
        :subtitle="getItemTypeAndMetaLabel(link.item)"
        :after="context.store[link.item.name] ? context.store[link.item.name].displayState || context.store[link.item.name].state : link.item.state">
        <oh-icon v-if="link.item.category" slot="media" :icon="link.item.category" :state="link.item.type === 'Image' ? null : (context.store[link.item.name].state || link.item.state)" height="32" width="32" />
        <span v-else slot="media" class="item-initial">{{ link.item.name[0] }}</span>
        <f7-icon v-if="!link.item.editable" slot="after-title" f7="lock_fill" size="1rem" color="gray" />
        <!-- <f7-button slot="after-start" color="blue" icon-f7="compose" icon-size="24px" :link="`${item.name}/edit`"></f7-button> -->
      </f7-list-item>
    </f7-list-group>
    <f7-list-item class="searchbar-ignore" link color="blue" subtitle="Add Link to Item..." @click="addLink()">
      <f7-icon slot="media" color="green" aurora="f7:plus_circle_fill" ios="f7:plus_circle_fill" md="material:control_point" />
    </f7-list-item>
    <f7-list-button class="searchbar-ignore" color="blue" :title="thing.editable && (channelType.parameterGroups.length || channelType.parameters.length) ? 'Configure Channel' : 'Channel Details'" @click="configureChannel()" />
    <f7-list-button class="searchbar-ignore" v-if="extensible && thing.editable" color="blue" title="Duplicate Channel" @click="duplicateChannel()" />
    <f7-list-button class="searchbar-ignore" v-if="extensible && thing.editable" color="red" title="Remove Channel" @click="removeChannel()" />
  </f7-list>
</template>

<style lang="stylus">
.channellist-linkeditem .item-after
  overflow hidden
  max-width 30%
  span
    overflow hidden
    text-overflow ellipsis
</style>

<script>
import AddLinkPage from '@/pages/settings/things/link/link-add.vue'
import ConfigureLinkPage from '@/pages/settings/things/link/link-edit.vue'
import ConfigureChannelPage from '@/pages/settings/things/channel/channel-edit.vue'
import DuplicateChannelPage from '@/pages/settings/things/channel/channel-duplicate.vue'

import ItemMixin from '@/components/item/item-mixin'

export default {
  mixins: [ItemMixin],
  props: ['channelType', 'channelId', 'channel', 'thing', 'opened', 'extensible', 'context'],
  data () {
    return {
      ready: false,
      loading: false,
      links: [],
      channelKind: ''
    }
  },
  methods: {
    buildLinks () {
      if (this.channel) {
        this.channelKind = this.channel.kind
        let links = []
        let promises = []
        this.loading = true
        this.channel.linkedItems.forEach((itemName) => {
          let link = {
            itemName,
            item: { name: itemName }
          }
          links.push(link)
          const fetchItem = this.$oh.api.get('/rest/items/' + link.itemName + '?metadata=semantics')
          fetchItem.then((i) => {
            link.item = i
          })
          promises.push(fetchItem)
        })
        Promise.all(promises).then(() => {
          this.links = links
          this.ready = true
          this.loading = false
        })
      }
    },
    addLink () {
      this.$f7router.navigate({
        url: 'links/new',
        route: {
          component: AddLinkPage,
          path: 'links/new'
        }
      }, {
        props: {
          thing: this.thing,
          channel: this.thing.channels.find((c) => c.id === this.channelId),
          channelType: this.channelType
        }
      })
    },
    configureLink (link) {
      const path = 'links/' + link.itemName + '/' + this.channelId
      this.$f7router.navigate({
        url: path,
        route: {
          component: ConfigureLinkPage,
          path,
          props: {
          },
          on: {
            pageAfterOut (event, page) {
            }
          }
        }
      }, {
        props: {
          thing: this.thing,
          channel: this.thing.channels.find((c) => c.id === this.channelId),
          channelType: this.channelType,
          item: link.item,
          source: 'thing'
        }
      })
    },
    configureChannel () {
      const self = this
      const path = 'channels/' + this.channelId + '/edit'
      this.$f7router.navigate({
        url: path,
        route: {
          component: ConfigureChannelPage,
          path,
          context: {
            operation: 'edit-channel'
          },
          on: {
            pageAfterOut (event, page) {
              const context = page.route.route.context
              const finalChannel = context.finalChannel
              if (finalChannel) {
                // replace the channel in-place
                const idx = self.thing.channels.findIndex((c) => c.uid === finalChannel.uid)
                self.$set(self.thing.channels, idx, finalChannel)
                self.$emit('channel-updated', true)
              } else {
                self.$emit('channel-updated', false)
              }
            }
          }
        }
      }, {
        props: {
          thing: this.thing,
          channel: this.channel,
          channelType: this.channelType,
          channelId: this.channelId
        }
      })
    },
    duplicateChannel () {
      const self = this
      const path = 'channels/' + this.channelId + '/edit'
      this.$f7router.navigate({
        url: path,
        route: {
          component: DuplicateChannelPage,
          path,
          context: {
            operation: 'duplicate-channel'
          },
          on: {
            pageAfterOut (event, page) {
              const context = page.route.route.context
              const finalChannel = context.finalChannel
              if (finalChannel) {
                self.thing.channels.push(finalChannel)
                self.$emit('channel-updated', true)
              } else {
                self.$emit('channel-updated', false)
              }
            }
          }
        }
      }, {
        props: {
          thing: this.thing,
          channel: this.channel,
          channelType: this.channelType,
          channelId: this.channelId
        }
      })
    },
    removeChannel () {
      const self = this

      if (this.links.length > 0) {
        this.$f7.dialog.alert('Please unlink all items to the channel before removing it')
        return
      }

      this.$f7.dialog.confirm(
        `Are you sure you want to remove the ${self.channel.label} channel from ${this.thing.label}?`,
        'Remove channel',
        () => {
          const idx = self.thing.channels.findIndex((c) => c.uid === self.channel.uid)
          self.thing.channels.splice(idx, 1)
          self.$emit('channel-updated', true)
        })
    }
  },
  watch: {
    opened (isOpen) {
      if (isOpen) {
        this.buildLinks()
      }
    },
    thing () {
      if (this.opened) {
        this.buildLinks()
      }
    }
  }
}
</script>
