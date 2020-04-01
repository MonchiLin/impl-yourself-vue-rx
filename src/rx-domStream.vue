<template>
  <div>
    <button v-stream:click="plus$">增加</button>
    <span>  {{ counter }}  </span>
    <button v-stream:click="minus$">减少</button>
    <p>{{ event }}</p>
  </div>
</template>

<script>
  import { fromEvent, Subject, Subscription } from "rxjs";
  import { stringifyEvent } from "./shared";

  export default {
    data() {
      return {
        counter: 0,
        event: ""
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
          this.event = stringifyEvent(e)
        })
      this.minus$
        .subscribe(e => {
          this.counter -= 1
          this.event = stringifyEvent(e)
        })
    },
    directives: {
      stream: {
        bind: function (el, binding, vNode, oldVnode) {
          const subject = binding.value
          const eventName = binding.arg
          el._subscription = new Subscription()
          // 储存 subscription 对象
          el._subscription.add(
            fromEvent(el, eventName)
              .subscribe(e => subject.next(e))
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
