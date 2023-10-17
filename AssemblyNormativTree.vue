<template>
  <div class="assembly-normativ-tree" :class="{ children: isChildren }">
    <div v-for="treeItem in treeData" :key="treeItem.classifier_id">
      <AssemblyNormativTreeItem
        :item="treeItem"
        :type="type"
        :selected-item="selectedItem"
        @select-child-node="onSelectChildNode"
        @opened="opened"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ITreeItemNormativ } from '@/api/normativData'

  interface Props {
    tree?: ITreeItemNormativ[]
    type: 'works' | 'resource'
    isChildren?: boolean
    selectedItem: ITreeItemNormativ | null
  }

  interface IEmits {
    (e: 'selectChildNode', data: ITreeItemNormativ): void
    (e: 'openedEl', data: any): void
  }

  const props = withDefaults(defineProps<Props>(), {
    tree: () => [],
  })
  const emit = defineEmits<IEmits>()

  const treeInner = ref<ITreeItemNormativ[]>([])

  const treeData = computed(() =>
    props.tree.length ? props.tree : unref(treeInner)
  )

  function onSelectChildNode(data: ITreeItemNormativ) {
    emit('selectChildNode', data)
  }
  const opened = (isOpen: boolean, item: ITreeItemNormativ) => {
    console.log('isOpen', isOpen, 'item', item.classifier_id)
    emit('openedEl', {
      classifier_id: item.classifier_id,
      isOpen,
    })
  }
</script>

<style lang="sass" scoped>
  @import './assemblyNormativTree'
</style>
