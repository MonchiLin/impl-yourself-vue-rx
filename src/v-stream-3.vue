<template>
  <div>
    <button
        v-stream:click="{ subject: plus$, data: '额外的数据',options: { once: true, passive: true, capture: true } }"
    >
      增加
    </button>
    <span>  {{ counter }}  </span>
    <button
        v-stream:click="{ subject: minus$, data: '额外的数据',options: { once: true, passive: true, capture: true } }"
    >
      减少
    </button>
    <p>{{ event }}</p>
    <p>{{ someData }}</p>
  </div>
</template>

<script>
  import { fromEvent, Subject, Subscription } from "rxjs";
  import { stringifyEvent } from "./shared";

  export default {
    data() {
      return {
        counter: 0,
        event: "",
        someData: "额外的数据"
      }
    },
    // 通过 domStreams 传入 ["plus$", "minus$"]
    domStreams: ["plus$", "minus$"],
    mixins: [
      {
        created() {
          const vm = this
          // 获取 domStreams，此处的 domStreams 值为 ["plus$", "minus$"]
          const domStreams = vm.$options.domStreams
          domStreams.forEach(key => {
            // 遍历 domStreams，将 vm[key] 赋值为 new Subject
            vm[key] = new Subject()
          })
          // 执行完上面这段 vm["plus$"] 和 vm["minus$"] 的值就变成了 new Subject
        }
      }
    ],
    mounted() {
      this.plus$
        .subscribe(e => {
          this.counter += 1
          this.someData = e.data
          this.event = stringifyEvent(e.event)
        })
      this.minus$
        .subscribe(e => {
          this.counter -= 1
          this.someData = e.data
          this.event = stringifyEvent(e.event)
        })
    },
    directives: {
      stream: {
        bind: function (el, binding, vNode, oldVnode) {
          // handle = { subject, data }
          let handle = binding.value
          const eventName = binding.arg

          // 放在这里方便读者看，这个函数是以鸭子类型的思想来判断对象是否为 observer
          function isObserver(subject) {
            return subject && (
              typeof subject.next === 'function'
            )
          }

          // 处理通过 v-stream="plus$" 传入进来的 subject
          if (isObserver(handle)) {
            // 包装一层，将 v-stream="plus$" 和
            // v-stream="{ subject: plus$, data: someData }" 两种数据结构统一处理
            // 此时 handle 数据结构变成了 { subject, data }
            handle = {subject: handle}
          }

          // 还记得 rxMixin 的功能之一吗？ 没错就是在 vm 上面挂载 _subscription 对象
          // 这里我们单独写主要是为了实现 v-stream
          el._subscription = new Subscription()

          const subject = handle.subject

          // 如果 handle 存在 options 则将 options 传给 fromEvent
          const fromEventArgs = handle.options ? [el, eventName, handle.options] : [el, eventName]
          // 储存 subscription 对象
          el._subscription.add(
            fromEvent(...fromEventArgs)
              // 将 event 和 data 都传递给开发者
              .subscribe(e => subject.next({
                event: e,
                data: handle.data + new Date().getTime()
              }))
          )
        },
        unbind(el, binding) {
          // 取消订阅
          el._subscription.unsubscribe()
        }
      }
    }
  }
</script>
