<template>
  <div class="assembly-normativ-tree-item">
    <div
      class="assembly-normativ-tree-item__name"
      :class="[{ selected: isSelectedItem }]"
      @click="onClickItem"
    >
      <div class="assembly-normativ-tree-item__icons">
        <div
          v-if="!!item.child_filename"
          class="assembly-normativ-tree-item__icon"
        >
          <IconSprite
            :name="!isOpen ? 'plus' : 'minus'"
            class="assembly-normativ-tree-item__icon-svg"
            :class="[{ open: isOpen }]"
          />
        </div>
        <IconSprite
          name="folder"
          :width="24"
          :height="24"
          class="assembly-normativ-tree-item__icon-folder"
        />
      </div>

      <span class="assembly-normativ-tree-item__code">
        {{ item.full_code }}
      </span>

      <span class="assembly-normativ-tree-item__el-name">{{ item.title }}</span>
    </div>

    <AssemblyNormativTree
      v-if="isOpen"
      :is-root="false"
      :type="type"
      is-children
      :tree="childTrees"
      :selected-item="selectedItem"
      @select-child-node="onSelectChildNode"
    />
  </div>
</template>

<script setup lang="ts">
  import {
    getFileResourcesRequest,
    getFileWorksRequest,
  } from '@/api/normativData'
  import { ITreeItemNormativ } from '@/api/normativData'
  import { usePeriodStore } from '@/stores/period'

  interface IProps {
    item: ITreeItemNormativ
    type: 'works' | 'resource'
    selectedItem: ITreeItemNormativ | null
  }

  interface IEmits {
    (e: 'selectChildNode', data: ITreeItemNormativ): void
    (e: 'opened', data: boolean, item: ITreeItemNormativ): void
  }

  const props = defineProps<IProps>()
  const emit = defineEmits<IEmits>()

  const periodStore = usePeriodStore()
  const { normativPeriodId } = storeToRefs(periodStore)

  const isSelectedItem = computed(
    () =>
      props.selectedItem?.full_code === props.item.full_code &&
      props.selectedItem?.depth === props.item?.depth
  )

  const isOpen = ref(false)

  const childTrees = ref<ITreeItemNormativ[]>([])

  function onClickItem() {
    isOpen.value = !isOpen.value
    emit('opened', isOpen.value, props.item)

    if (
      props.item.child_filename &&
      isOpen.value &&
      !unref(childTrees).length &&
      normativPeriodId.value
    ) {
      ;(props.type === 'resource'
        ? getFileResourcesRequest(
            normativPeriodId.value,
            props.item.child_filename
          )
        : getFileWorksRequest(normativPeriodId.value, props.item.child_filename)
      ).then(({ data }) => {
        if ('results' in data) {
          childTrees.value = data.results as unknown as ITreeItemNormativ[]
        }
      })
    }

    if (props.item.works_leaf_file || props.item.resources_leaf_file) {
      emit('selectChildNode', props.item)
    }
  }

  function onSelectChildNode(data: ITreeItemNormativ) {
    emit('selectChildNode', data)
  }
</script>

<style lang="sass" scoped>

  @import './assemblyNormativTreeItem'
</style>
