<script lang="ts">
import {
  createVNode,
  defineComponent,
  Fragment,
  Teleport,
  onMounted,
  onBeforeUnmount,
  onDeactivated,
  onActivated,
  renderSlot,
  toDisplayString,
  withCtx,
} from 'vue'

import throwError from '@element-plus/utils/error'
import { stop } from '@element-plus/utils/dom'
import { renderBlock, PatchFlags } from '@element-plus/utils/vnode'

import usePopper from './use-popper/index'
import defaultProps from './use-popper/defaults'

import { Mask, renderPopper, renderTrigger, renderArrow } from './renderers'

const compName = 'ElPopper'
const UPDATE_VISIBLE_EVENT = 'update:visible'

const emits = [UPDATE_VISIBLE_EVENT, 'after-enter', 'after-leave']

export default defineComponent({
  name: compName,
  props: defaultProps,
  emits,
  setup(props, ctx) {
    if (!ctx.slots.trigger) {
      throwError(compName, 'Trigger must be provided')
    }
    // this is a reference that we need to pass down to child component
    // to obtain the child instance

    // return usePopper(props as IPopperOptions, ctx as SetupContext)
    const popperStates = usePopper(props, ctx)

    const forceDestroy = () => popperStates.doDestroy(true)
    onMounted(popperStates.initializePopper)
    onBeforeUnmount(forceDestroy)
    onActivated(popperStates.initializePopper)
    onDeactivated(forceDestroy)

    return popperStates
  },

  render() {
    const {
      $slots,
      appendToBody,
      class: kls,
      effect,
      hide,
      onPopperMouseEnter,
      onPopperMouseLeave,
      onAfterEnter,
      onAfterLeave,
      popperClass,
      popperId,
      pure,
      showArrow,
      tabIndex,
      transition,
      visibility,
    } = this

    const arrow = renderArrow(showArrow)
    const popper = renderPopper(
      {
        effect,
        name: transition,
        popperClass,
        popperId,
        pure,
        onMouseEnter: onPopperMouseEnter,
        onMouseLeave: onPopperMouseLeave,
        onAfterEnter,
        onAfterLeave,
        visibility,
      },
      [
        renderSlot($slots, 'default', {}, () => {
          return [toDisplayString(this.content)]
        }),
        arrow,
      ],
    )

    const trigger = renderTrigger($slots.trigger?.(), {
      ariaDescribedby: popperId,
      class: kls,
      ref: 'triggerRef',
      tabindex: tabIndex,
      onMouseDown: stop,
      onMouseUp: stop,
      ...this.events,
    })

    return renderBlock(Fragment, null, [
      trigger,
      appendToBody
        ? createVNode(
          Teleport,
          {
            to: 'body',
            key: 0,
          },
          [
            createVNode(
              Mask,
              {
                hide,
                isManualMode: this.isManualMode(),
              },
              {
                default: withCtx(() => [popper]),
              },
              PatchFlags.PROPS,
              ['hide', 'isManualMode'],
            ),
          ],
        )
        : renderBlock(Fragment, { key: 1 }, [popper]),
    ])
  },
})
</script>

<style>
.el-popper__mask {
  position: absolute;
  top: 0px;
  left: 0px;
  width: 100%;
}

.el-popper {
  position: absolute;
  border-radius: 4px;
  padding: 10px;
  z-index: 2000;
  font-size: 12px;
  line-height: 1.2;
  min-width: 10px;
  word-wrap: break-word;
  visibility: visible;
}

.el-popper__arrow,
.el-popper__arrow::before {
  position: absolute;
  width: 10px;
  height: 10px;
  z-index: -1;
}

.el-popper__arrow::before {
  content: ' ';
  transform: rotate(45deg);
  background: #303133;
  box-sizing: border-box;
}

.el-popper[data-popper-placement^='top'] > .el-popper__arrow {
  bottom: -5px;
}

.el-popper[data-popper-placement^='bottom'] > .el-popper__arrow {
  top: -5px;
}

.el-popper[data-popper-placement^='left'] > .el-popper__arrow {
  right: -5px;
}

.el-popper[data-popper-placement^='right'] > .el-popper__arrow {
  left: -5px;
}

.is-dark {
  background: #303133;
  color: #fff;
}
.is-light {
  background: #fff;
  border: 1px solid #303133;
}

.is-dark .el-popper__arrow::before {
  background: #303133;
}

.is-light .el-popper__arrow::before {
  background: #fff;
  border: 1px solid #303133;
}

.is-light[data-popper-placement^='top'] .el-popper__arrow::before {
  border-top-color: transparent;
  border-left-color: transparent;
}

.is-light[data-popper-placement^='bottom'] .el-popper__arrow::before {
  border-bottom-color: transparent;
  border-right-color: transparent;
}

.is-light[data-popper-placement^='left'] .el-popper__arrow::before {
  border-left-color: transparent;
  border-bottom-color: transparent;
}

.is-light[data-popper-placement^='right'] .el-popper__arrow::before {
  border-top-color: transparent;
  border-right-color: transparent;
}

.el-popper__pure {
  padding: 0;
  border: none;
}

.el-popper__pure .el-popper__arrow::before {
  border: none;
}
</style>
