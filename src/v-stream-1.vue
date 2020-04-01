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
        // 先将 Subject 放在 data 中
        plus$: new Subject(),
        minus$: new Subject(),
        counter: 0,
        event: ""
      }
    },
    created() {
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
