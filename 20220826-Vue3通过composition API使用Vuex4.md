# 20220826-Vue3通过composition API使用Vuex4

## 访问 State 和 Getter[#](https://vuex.vuejs.org/zh/guide/composition-api.html#访问-state-和-getter)

为了访问 state 和 getter，需要创建 `computed` 引用以保留响应性，这与在选项式 API 中创建计算属性等效。

```js
import { computed } from 'vue'
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      // 在 computed 函数中访问 state
      count: computed(() => store.state.count),

      // 在 computed 函数中访问 getter
      double: computed(() => store.getters.double)
    }
  }
}
```



## 访问 Mutation 和 Action[#](https://vuex.vuejs.org/zh/guide/composition-api.html#访问-mutation-和-action)

要使用 mutation 和 action 时，只需要在 `setup` 钩子函数中调用 `commit` 和 `dispatch` 函数。

```js
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      // 使用 mutation
      increment: () => store.commit('increment'),

      // 使用 action
      asyncIncrement: () => store.dispatch('asyncIncrement')
    }
  }
}
```

## 跨模块 使用案例：存储已授权的验证码

> [在带命名空间的模块内访问全局内容（Global Assets）](https://vuex.vuejs.org/zh/guide/modules.html#在带命名空间的模块内访问全局内容（global-assets）)

在页面中存储验证码，数据存储在cache模块中的state中

```js
import { useStore } from 'vuex';
const store = useStore();

// 接口获取数据
const res = await apis.userManagement.getMsgCode(formData.value); 
// 存储已授权验证码
store.commit('cache/SET_PHONE_CODE_CACHE', {...record, result:res.data});
```

退出登录时清除数据，由于清除数据这个操作在user.js模块中，要跨模块操作cache模块的数据。

这里注意当前模块的actions中的方法，解构出的commit，默认只能操作本模块的mutation。

如果要操作其他模块，需要在commit的第三个参数加上{root:true} : `commit('cache/RESET_PHONE_CODE_CACHE', {},{root:true});`

> 若需要在全局命名空间内分发 action 或提交 mutation，将 `{ root: true }` 作为第三参数传给 `dispatch` 或 `commit` 即可。

```js
//user.js
function cleanAll(commit) {
  commit('SET_TOKEN', '');
  commit('SET_ORGANIZATION_ID', '');
  commit('SET_LOGIN_INFO', {});
  commit('SET_LOGIN_INFO', {});
  removeToken();
  removeLoginInfo();
  removeOrganizationId();
}
// 删除存储的数据
const actions = {
    FedLogOut({ commit, state }) {
        return new Promise(resolve => {
          cleanAll(commit);
          commit('cache/RESET_PHONE_CODE_CACHE', {},{root:true});
          if(state.token) {
            apis.login.logout();
          }
          resolve();
        });
     },
}
```

![image-20230209185511082](https://s2.loli.net/2023/02/09/ZansvO3RzFHoABp.png)

官方案例

```js
modules: {
  foo: {
    namespaced: true,

    getters: {
      // 在这个模块的 getter 中，`getters` 被局部化了
      // 你可以使用 getter 的第四个参数来调用 `rootGetters`
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
        rootGetters['bar/someOtherGetter'] // -> 'bar/someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // 在这个模块中， dispatch 和 commit 也被局部化了
      // 他们可以接受 `root` 属性以访问根 dispatch 或 commit
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'
        rootGetters['bar/someGetter'] // -> 'bar/someGetter'

        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}
```

