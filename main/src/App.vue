<template>
  <div class="layout-wrapper">

    <div class="layout-header">
      <div class="logo">QIANKUN-EXAMPLE</div>
      <ul class="sub-apps">
        <li v-for="item in microApps" :class="{active: item.activeRule === current}" :key="item.name" @click="goto(item)">{{ item.name }}</li>
      </ul>
      <div class="userinfo">主应用的state：{{ JSON.stringify(state) }}</div>
    </div>
    <div id="subapp-viewport"></div>
  </div>
</template>

<script>
import NProgress from 'nprogress'
import microApps from './micro-app'
import store from '@/store'
export default {
  name: 'App',
  data () {
    return {
      isLoading: true,
      microApps,
      current: '/sub-vue/'
    }
  },
  computed: {
    state () {
      // 如果只需要取某个命名空间下的state，比如 user ，可以加上参数
      // return store.getGlobalState('user')

      // 返回所有的state则不需添加参数
      return store.getGlobalState()
    }
  },
  watch: {
    isLoading (val) {
      if (val) {
        NProgress.start()
      } else {
        this.$nextTick(() => {
          NProgress.done()
        })
      }
    }
  },
  components: {},
  methods: {
    goto (item) {
      history.pushState(null, item.activeRule, item.activeRule)
      // this.current = item.name
    },
    bindCurrent () {
      const path = window.location.pathname
      if (this.microApps.findIndex(item => item.activeRule === path) >= 0) {
        this.current = path
      }
    },
    listenRouterChange () {
      const _wr = function (type) {
        const orig = history[type]
        return function () {
          const rv = orig.apply(this, arguments)
          const e = new Event(type)
          e.arguments = arguments
          window.dispatchEvent(e)
          return rv
        }
      }
      history.pushState = _wr('pushState')

      window.addEventListener('pushState', this.bindCurrent)
      window.addEventListener('popstate', this.bindCurrent)

      this.$once('hook:beforeDestroy', () => {
        window.removeEventListener('pushState', this.bindCurrent)
        window.removeEventListener('popstate', this.bindCurrent)
      })
    }
  },
  created () {
    this.bindCurrent()
    NProgress.start()
  },
  mounted () {
    this.listenRouterChange()
  }
}
</script>

<style lang="scss">
html, body{
  margin: 0 !important;
  padding: 0;
}
.github-corner:hover .octo-arm{animation:octocat-wave 560ms ease-in-out}@keyframes octocat-wave{0%,100%{transform:rotate(0)}20%,60%{transform:rotate(-25deg)}40%,80%{transform:rotate(10deg)}}@media (max-width:500px){.github-corner:hover .octo-arm{animation:none}.github-corner .octo-arm{animation:octocat-wave 560ms ease-in-out}}
  .layout-wrapper{
    .layout-header{
      height: 50px;
      width: 100%;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
      line-height: 50px;
      position: relative;
     .logo {
        float: left;
        margin: 0 50px;
      }
      .sub-apps {
        list-style: none;
        margin: 0;
        li{
          list-style: none;
          display: inline-block;
          padding: 0 20px;
          cursor: pointer;
          &.active{
            color: #42b983;
            text-decoration: underline;
          }
        }
      }
      .userinfo{
        position: absolute;
        right: 100px;
        top: 0;
      }
    }
  }
</style>
