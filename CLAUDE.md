# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DataX Web UI — 基于 Vue 2 + Element UI 的 DataX 分布式数据同步平台前端，由 `vue-element-admin` 修改而来。后端项目：[datax-web](https://github.com/WeiYe-Jing/datax-web)。

## Build & Dev Commands

```bash
npm run dev          # 启动开发服务器 (localhost:8082)
npm run build:prod   # 生产构建
npm run build:stage  # 预发布构建
npm run lint         # ESLint 检查 (.js,.vue)
npm run test:unit    # Jest 单元测试
npm run new          # 通过 plop 快速生成组件模板
```

## Architecture

### Tech Stack
- **Vue 2** (2.6.10) + Vue Router 3 + Vuex 3
- **Element UI** 组件库
- Axios HTTP 客户端 (请求/响应拦截器处理 token 鉴权)
- CodeMirror 5 + ECharts 4
- SCSS 样式方案
- ESLint + husky + lint-staged 代码规范

### Directory Structure

```
src/
  api/          # 按业务模块划分的 API 封装 (axios 请求)
  components/   # 通用组件 (Cron, JsonEditor, Pagination 等)
  layout/       # 主布局组件 (Sidebar, Navbar, TagsView, AppMain)
  views/        # 页面组件
    datax/        # 核心业务页面
      jobInfo/         # 任务管理
      json-build/      # 任务构建
      json-build-batch/# 批量构建
      jobTemplate/     # 任务模板
      jobProject/      # 项目管理
      jdbc-datasource/ # 数据源管理
      jobLog/          # 日志管理
      executor/        # 执行器管理
      registry/        # 资源监控
      user/            # 用户管理
    dashboard/    # 运行报表
    login/        # 登录
  store/modules/ # Vuex 状态模块 (app, user, permission, tagsView, settings, errorLog)
  router/        # 路由配置 (constantRoutes + asyncRoutes 权限路由)
  utils/         # 工具函数 (request.js, auth.js, validate.js, permission.js 等)
  styles/        # 全局 SCSS 样式
  icons/         # SVG 图标
  directive/     # 自定义指令
  filters/       # 全局过滤器
  permission.js  # 路由导航守卫 (登录检查/权限路由动态注入)
mock/             # Mock 数据
build/            # 构建脚本
```

### Key Architectural Patterns

1. **权限路由**: `router/index.js` 定义 `constantRoutes`（公开路由）和 `asyncRoutes`（权限路由）。`permission.js` 中的路由守卫根据 token/roles 动态注入权限路由。

2. **API 层**: `src/api/` 下每文件对应一个业务领域，使用 `src/utils/request.js` 封装的 axios 实例，自动注入 Authorization header 并统一处理响应状态码。

3. **状态管理**: Vuex 模块自动注册（`store/index.js` 通过 `require.context` 加载所有模块），getter 集中在 `store/getters.js`。

4. **代理配置**: `vue.config.js` 中 `/dev-api` -> `localhost:8080/api`，请求路径用 `/api/xxx` 即可。

5. **HTTP 响应处理**: 后端返回 `{code, msg, content}` 格式。code=0 取 content.data，code=200 取整个 data，其余直接返回 res。token 过期(code 50008/50012/50014)自动跳转登录页。
