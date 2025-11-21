# vue3 源码解析

- pnpm

```sh
# 只为 @vue/reactivity 包安装 @vue/shared
pnpm install @vue/shared --workspace --filter @vue/reactivity

# https://pnpm.io/zh/workspaces
# 再将 package 改为 *
# "dependencies": {
#   "@vue/shared": "workspace:*"
# }
```