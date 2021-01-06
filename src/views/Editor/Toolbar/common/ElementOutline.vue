<template>
  <div class="element-outline">
    <div class="row">
      <div style="flex: 2;">启用边框：</div>
      <div class="switch-wrapper" style="flex: 3;">
        <Switch 
          :checked="hasOutline" 
          @change="checked => toggleOutline(checked)" 
        />
      </div>
    </div>
    <template v-if="hasOutline">
      <div class="row">
        <div style="flex: 2;">边框样式：</div>
        <Select 
          style="flex: 3;" 
          :value="outline.style" 
          @change="value => updateOutline({ style: value })"
        >
          <SelectOption value="solid">实线边框</SelectOption>
          <SelectOption value="dashed">虚线边框</SelectOption>
        </Select>
      </div>
      <div class="row">
        <div style="flex: 2;">边框颜色：</div>
        <Popover trigger="click">
          <template #content>
            <ColorPicker
              :modelValue="outline.color"
              @update:modelValue="value => updateOutline({ color: value })"
            />
          </template>
          <Button class="color-btn" style="flex: 3;">
            <div class="color-block" :style="{ backgroundColor: outline.color }"></div>
            <DownOutlined class="color-btn-icon" />
          </Button>
        </Popover>
      </div>
      <div class="row">
        <div style="flex: 2;">边框粗细：</div>
        <InputNumber 
          :value="outline.width" 
          @change="value => updateOutline({ width: value })" 
          style="flex: 3;" 
        />
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, Ref, ref, watch } from 'vue'
import { useStore } from 'vuex'
import { MutationTypes, State } from '@/store'
import { PPTElement, PPTElementOutline } from '@/types/slides'
import useHistorySnapshot from '@/hooks/useHistorySnapshot'

import ColorPicker from '@/components/ColorPicker/index.vue'
import { Select, Button, Popover, InputNumber, Switch } from 'ant-design-vue'
import { DownOutlined } from '@ant-design/icons-vue'

export default defineComponent({
  name: 'element-outline',
  components: {
    ColorPicker,
    Select,
    SelectOption: Select.Option,
    Button,
    Popover,
    InputNumber,
    Switch,
    DownOutlined,
  },
  setup() {
    const store = useStore<State>()
    const handleElement: Ref<PPTElement> = computed(() => store.getters.handleElement)

    const outline = ref<PPTElementOutline>()
    const hasOutline = ref(false)

    watch(handleElement, () => {
      if(!handleElement.value) return
      outline.value = 'outline' in handleElement.value && handleElement.value.outline || undefined
      hasOutline.value = !!outline.value
    }, { deep: true, immediate: true })

    const { addHistorySnapshot } = useHistorySnapshot()

    const updateOutline = (outlineProps: Partial<PPTElementOutline>) => {
      const props = { outline: { ...outline.value, ...outlineProps } }
      store.commit(MutationTypes.UPDATE_ELEMENT, { id: handleElement.value.id, props })
      addHistorySnapshot()
    }

    const toggleOutline = (checked: boolean) => {
      let props: { outline?: PPTElementOutline } = { outline: undefined }
      if(checked) {
        props = { outline: { width: 2, color: '#000', style: 'solid' } }
      }
      store.commit(MutationTypes.UPDATE_ELEMENT, { id: handleElement.value.id, props })
      addHistorySnapshot()
    }

    return {
      outline,
      hasOutline,
      toggleOutline,
      updateOutline,
    }
  },
})
</script>

<style lang="scss" scoped>
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.color-btn {
  display: flex;
  align-items: center;
  padding: 0 !important;
}
.color-block {
  width: 100px;
  height: 20px;
  background-color: #777;
  margin: 0 8px;
}
.color-btn-icon {
  font-size: 12px;
  margin-top: 2px;
  color: #bfbfbf;
}
.switch-wrapper {
  text-align: right;
}
</style>