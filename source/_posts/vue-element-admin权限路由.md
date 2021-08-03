---
title: vue-element-admin权限验证
date: 2021-06-25 19:52:52
tags:
---
> 用[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin)做了好多项目,vue-element-admin项目中有很多可以总结的地方，例如 权限验证的解决方案、mock的解决方案、svg的icon解决方案以及webpack打包的优化等，从中都可以学到很多。本文先从权限的解决方案先总结起，日后有时间再总结下其他的内容。

# 思路
1. 请求后端接口，获取当前用户的权限，确认后端返回权限字段的内容。
2. 设置路由表的mete权限字段(根据后端接口返回的内容)。
3. router.beforeEach的时候将请求的权限和路由表的权限进行对比，筛选出符合的路由，利用`vue-router`的`router.addRoutes`API将符合条件的路由挂载到 router 上。
4. 将符合条件的路由数组渲染到侧边栏的菜单。

## router.js文件 路由表中的mete设置(src/router/index.js)
``` javascript
export const asyncRoutes = [
  {
    path: '/list',
    component: Layout,
    name: 'DataSubscription',
    mete:{
        title: '列表', icon: 'table',
        permission: ['admin','user'] // 根据后端接口返回的内容进行修改，这里只是演示
    }
  },
  // 这里注意，一定要在asyncRoutes最后面设置path: '*'，不然刷新页面会出现404的情况。
  // 如果constantRoutes里面也设置了 path: '*'，需要将constantRoutes里面的删除
  { path: '*', redirect: '/404', hidden: true }
]  
```

## vuex中的permission.js(src/store/modules/permission.js)
主要是三个方法 

1. `hasPermission`方法判断是否有权限
2. `filterAsyncRoutes`方法，递归过滤不符合权限的路由
3. `generateRoutes`方法来生成路由
``` javascript
import { asyncRoutes, constantRoutes } from '@/router'

function hasPermission(permissions, route) {
  if (route.meta && route.meta.permission) {
    // 这里也可以根据业务用其他的条件进行筛选
    return permissions.some(permission => route.meta.permission.include(permission)
  } else {
    return true
  }
}
function filterAsyncRoutes(routes, roles) {
  const res = []
  routes.forEach(route => {
    const tmp = { ...route }
    if (hasPermission(roles, tmp)) {
      if (tmp.children) {
        tmp.children = filterAsyncRoutes(tmp.children, roles)
      }
      res.push(tmp)
    }
  })

  return res
}
function generateRoutes({ commit }, permissions) {
	return new Promise(resolve => {
		let accessedRoutes
		if (permissions[0].includes('*')) {
			// 要根据业务场景进行其他的判断
			// 超级管理员获取全部菜单路由
			accessedRoutes = asyncRoutes || []
		} else {
			// 普通管理员
			accessedRoutes = filterAsyncRoutes(asyncRoutes, permissions)
		}
		commit('SET_ROUTES', accessedRoutes)
		resolve(accessedRoutes)
	})
}
```

## 路由每次进入的时候进行判断(src/permission.js)
``` javascript
const whiteList = ['/login'] // no redirect whitelist

router.beforeEach(async(to, from, next) => {
  // 根据是否有token,判断是否登录
  const hasToken = getToken()

  if (hasToken) {
    if (to.path === '/login') {
      // if is logged in, redirect to the home page
      next({ path: '/' })
      NProgress.done()
    } else {
      const hasGetUserInfo = store.getters.name
      if (hasGetUserInfo) {
        next()
      } else {
        try {
          // 获取用户权限
          const { permissions } = await store.dispatch('user/getInfo')

          // 根据权限进行动态生成路由
          const accessRoutes = await store.dispatch('permission/generateRoutes', permissions)
          // 生成最终路由
          router.addRoutes(accessRoutes)

          // hack method to ensure that addRoutes is complete
          // set the replace: true, so the navigation will not leave a history record
          next({ ...to, replace: true })
        } catch (error) {
          // remove token and go to login page to re-login
          await store.dispatch('user/resetToken')
          Message.error(error || 'Has Error')
          next(`/login?redirect=${to.path}`)
          NProgress.done()
        }
      }
    }
  } else {
    /* has no token*/
    if (whiteList.indexOf(to.path) !== -1) {
      // in the free login whitelist, go directly
      next()
    } else {
      // other pages that do not have permission to access are redirected to the login page.
      next(`/login?redirect=${to.path}`)
      NProgress.done()
    }
  }
})
```
## layout中的侧边栏展示可用路由(src/layout/components/Sidebar/index.vue)
``` javascript
<template>
	<el-scrollbar wrap-class="scrollbar-wrapper">
		<el-menu
			:default-active="activeMenu"
			:collapse="isCollapse"
			:background-color="variables.menuBg"
			:text-color="variables.menuText"
			:unique-opened="false"
			:active-text-color="variables.menuActiveText"
			:collapse-transition="false"
			mode="vertical"
		>
			<sidebar-item v-for="route in permission" :key="route.path" :item="route" :base-path="route.path" />
		</el-menu>
	</el-scrollbar>
</template>
export default {
computed: {
	...mapGetters([
		// 从store中获取
		'permission'
	]),
}
```



