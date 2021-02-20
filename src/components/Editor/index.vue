<template>
    <div class="editor" id="editor" 
        :class="{ edit: isEdit }" :style="{ width: canvasStyleData.width + 'px', height: canvasStyleData.height + 'px' }">
        <!--页面组件列表展示-->
        <Shape v-for="(item, index) in componentData"
            :defaultStyle="item.style"
            :style="getShapeStyle(item.style, index)"
            :key="item.id"
            :active="item === curComponent"
            :element="item"
            :zIndex="index"
        >
            <component
                v-if="item.component != 'v-text'"
                class="component"
                :is="item.component"
                :style="getComponentStyle(item.style)"
                :propValue="item.propValue"
                @click.native="triggleEvent(curComponent)"
            />

            <component
                v-else
                class="component"
                :is="item.component"
                :style="getComponentStyle(item.style)"
                :propValue="item.propValue"
                @input="handleInput"
                :element="item"
            />
        </Shape>
        <!-- 右击菜单 -->
        <ContextMenu />
        <!-- 标线 -->
        <MarkLine />
    </div>
</template>

<script>
import { mapState } from 'vuex'
import Shape from './Shape'
import getStyle from '@/utils/style'
import ContextMenu from './ContextMenu'
import MarkLine from './MarkLine'

export default {
    props: {
        isEdit: {
            type: Boolean,
            default: true,
        },
    },
    components: { Shape, ContextMenu, MarkLine },
    computed: mapState([
        'componentData',
        'curComponent',
        'canvasStyleData',
    ]),
    methods: {
        getShapeStyle(style, index) {
            const result = { ...style }
            if (result.width) {
                result.width += 'px'
            }

            if (result.height) {
                result.height += 'px'
            }

            if (result.top) {
                result.top += 'px'
            }

            if (result.left) {
                result.left += 'px'
            }

            if (result.rotate) {
                result.transform = 'rotate(' + result.rotate + 'deg)'
            }
            // 按顺序添加 z-index 层级
            result.zIndex = index

            return result
        },

        getComponentStyle(style) {
            return getStyle(style, ['top', 'left', 'width', 'height', 'zIndex', 'rotate'])
        },

        handleInput(element, value) {
            element.propValue = value
            // 根据文本组件高度调整 shape 高度
            this.$store.commit('setShapeStyle', { height: this.getTextareaHeight(element, value) })
        },

        getTextareaHeight(element, text) {
            let { lineHeight, fontSize, height } = element.style
            if (lineHeight === '') {
                lineHeight = 1.5
            }

            const newHeight = text.split('\n').length * lineHeight * fontSize
            return height > newHeight? height : newHeight
        },
        triggleEvent(CurCommponent) {
            console.log('components', CurCommponent)
            let url = CurCommponent.events.redirect
            if (CurCommponent.events.alert) {
                this.$alert(CurCommponent.events.alert, 'alert事件', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    callback: action => {
                        this.$message({
                            type: 'info',
                            message: `action: ${action}`,
                        })
                    },
                })
            }
            if (CurCommponent.events.redirect) {
                window.open(url, '_blank')
            }
        },
      
    },
}
</script>

<style lang="scss" scoped>
.editor {
    position: relative;
    background: #fff;
}
.edit {
    .component {
        outline: none;
        width: 100%;
        height: 100%;
    }
}
</style>