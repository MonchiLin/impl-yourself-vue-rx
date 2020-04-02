<template>
  <div>
    <div>{{count}}</div>
    <!-- callback declared on observableMethods -->
    <button v-on:click="muchMore(500)">Add 500</button>

    <button v-on:click="minus(minusDelta1)"> Minus on Click</button>

    <pre>{{ $data }}</pre>

  </div>
</template>

<script>
  import { Observable, merge } from "rxjs";
  import { scan, share, startWith } from "rxjs/operators";
  import Vue from "vue";

  export default {
    data() {
      return {
        minusDelta1: -1,
        minusDelta2: -1
      }
    },
    observableMethods: {
      muchMore: 'muchMore$',
      minus: 'minus$'
    },
    subscriptions() {
      return {
        count: merge(
          this.muchMore$,
          this.minus$
        ).pipe(
          startWith(0),
          scan((total, change) => total + change)
        )
      }
    },
    mixins: [
      {
        created() {
          const vm = this

          const $createObservableMethod = (methodName) => {
            // 定义一个 subscriber
            const subscriber = (observer) => {
              // 在 vm 上挂载方法 vm["muchMore"] = xx
              // 调用这个方法时就会触发 observer.next
              vm[methodName] = (val) => {
                observer.next(val)
              }
              return () => {
                delete vm[methodName]
              }
            }

            return new Observable(subscriber).pipe(share())
          }

          // 假设我们传入如下结构
          // observableMethods: {
          //    muchMore: 'muchMore$',
          //    minus: 'minus$'
          // }
          const observableMethods = vm.$options.observableMethods
          Object.keys(observableMethods).forEach(key => {
            // 在 vm 上面挂载 muchMore$，muchMore$ 是一个 Observable
            // vm["muchMore$"] = $createObservableMethod("muchMore")
            vm[observableMethods[key]] = $createObservableMethod(key)
          })

          let subscriptions = vm.$options.subscriptions

          if (typeof subscriptions === "function") {
            subscriptions = subscriptions.call(vm)
          }

          vm.$observables = {}

          // subscriptions
          Object.keys(subscriptions)
            .forEach(key => {
              Vue.util.defineReactive(vm, key)
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
