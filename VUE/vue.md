# Composition API

# Router

注册全局守卫beforeEach，获取vuex的store.state的值时，要写在main.js中。因为写在router/index.js时，store未挂载

#Vuex

setup()获取store的值，要使用计算属性computed(()=>useStore().state.***)