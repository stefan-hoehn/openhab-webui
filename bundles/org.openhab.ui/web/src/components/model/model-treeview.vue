<template>
  <f7-treeview class="model-treeview">
    <draggable :disabled="!canDragDrop" :list="children" group="model-treeview" animation="150" forceFallBack="true" fallbackOnBody="true" fallbackThreshold="5"
               scrollSensitivity="200" delay="400" delayOnTouchOnly="true" touchStartThreshold="10" invertSwap="true" sort="false"
               @start="onDragStart" @change="onDragChange" @end="onDragEnd" :move="onDragMove">
      <model-treeview-item v-for="node in children"
                           :key="node.item.name" :model="node" :parentNode="model"
                           :includeItemName="includeItemName" :includeItemTags="includeItemTags" :canDragDrop="canDragDrop" :moveState="moveState"
                           @selected="nodeSelected" :selected="selected"
                           @checked="(item, check) => $emit('checked', item, check)"
                           @reload="$emit('reload')" />
    </draggable>
  </f7-treeview>
</template>

<style lang="stylus">
.model-treeview
  --f7-treeview-item-height 40px
  .treeview-item-label
    font-size 10pt
    white-space nowrap
    overflow-x hidden
  .semantic-class
    font-size 8pt
    color var(--f7-list-item-footer-text-color)
</style>

<script>
import ModelTreeviewItem from '@/components/model/treeview-item.vue'
import ModelDragDropMixin from '@/pages/settings/model/model-dragdrop-mixin'
import Draggable from 'vuedraggable'

export default {
  mixins: [ModelDragDropMixin],
  props: ['rootNodes', 'selected', 'includeItemName', 'includeItemTags', 'canDragDrop'],
  emits: ['reload'],
  components: {
    Draggable,
    ModelTreeviewItem
  },
  computed: {
    model: {
      get: function () {
        return {
          class: '',
          children: {
            locations: this.rootNodes.filter(n => n.class.startsWith('Location')),
            equipment: this.rootNodes.filter(n => n.class.startsWith('Equipment')),
            points: this.rootNodes.filter(n => n.class.startsWith('Point')),
            groups: this.rootNodes.filter(n => !n.class && n.item.type === 'Group'),
            items: this.rootNodes.filter(n => !n.class && n.item.type !== 'Group')
          },
          opened: true,
          item: null
        }
      }
    }
  },
  methods: {
    nodeSelected (node) {
      this.$emit('selected', node)
    }
  }
}
</script>
