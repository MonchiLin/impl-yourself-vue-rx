<template>
  <div>
    <button @click="plus$.next($event)">增加</button>
    <span> >> {{ counter }} << </span>
  </div>
</template>

<script>
  import { map, scan, startWith } from "rxjs/operators";
  import { Subject } from "rxjs";
  import Vue from 'vue'

  const plus$ = new Subject()

  export default {
    data() {
      return {
        plus$: plus$
      }
    },
    // 通过 subscriptions 传入 [counter]
    subscriptions: {
      counter: plus$
        .pipe(
          map(() => 1),
          startWith(0),
          scan((total, change) => total + change)
        )
    },
    mixins: [
      {
        created() {
          const vm = this
          // 获取 subscriptions, 这里我们假设 subscriptions = [{counter: Observable}]
          const subscriptions = vm.$options.subscriptions
          vm.$observables = {}

          Object.keys(subscriptions)
            .forEach(key => {
              // 在 vm 上创建一个响应式的属性，这里等价于 Vue.util.defineReactive(vm, "counter", undefined)
              Vue.util.defineReactive(vm, key, undefined)
              vm.$observables[key] = subscriptions[key]

              // 然后我们订阅 Observable, 并且在发生更新的时候赋值给 vm["counter"]
              subscriptions[key]
                .subscribe(e => {
                  vm[key] = e
                })
            })

        }
      }
    ],
  }
</script>
