在电脑正常安装Node.js和pnpm后，直接运行pnpm dev若是出现报错，则是因为有些依赖没有安装,以下是一些解决办法：

参考视频及文档：

Node.js安装：[Nodejs安装零基础教程2025](https://www.bilibili.com/video/BV1sbjgzwEBX/?spm_id_from=333.1391.0.0&vd_source=7f406b992b179c7aef591f257da5c339)

Valaxy Theme Sakura文档：[Valaxy Theme Sakura安装指南](https://sakura.valaxy.site/guide/getting-started/installation)

Valaxy官方文档：[Valaxy官方文档](https://valaxy.site/guide/getting-started)

valaxy博客全局美化教程：[valaxy博客全局美化教程（八）](https://daily.yybb.us/posts/valaxy-8/)

## 问题1：主题包未安装
在刚刚创建的 Valaxy 项目后，推荐打开终端，然后执行以下命令来安装主题：
```
pnpm add valaxy-theme-sakura
```

## 问题2：pnpm install部分依赖安装被限制
查看日志信息出现：`[ERROR] Command failed with exit code 1: ... "pnpm" install`，这说明 pnpm 在执行安装后的脚本（比如编译原生代码）时，进程被强制结束了（Exit Code 1 代表异常退出）。

### 哪一步出问题了？

往前看日志，有这行：`[ERR_PNPM_IGNORED_BUILDS] Ignored build scripts: @parcel/watcher@2.5.6, esbuild@0.25.12, typeit@8.8.7, vue-demi@0.14.10`这些被“忽略”的包（如 esbuild、@parcel/watcher）就是导致错误的导火索。esbuild 在安装时需要下载一个针对你电脑的二进制文件。@parcel/watcher 需要编译 C++ 代码。当 pnpm 提示你 Run "pnpm approve-builds" 时，说明你没有允许它们执行安装脚本。但程序又必须要跑这些脚本才能完成安装，两边“僵持”住了，最后 pnpm 判断安装失败，退出了。

### 解决办法：
执行如下命令
```
pnpm approve-builds
```
执行后，终端会弹出一个交互列表，让你用空格键选择允许哪些包运行脚本。推荐做法：按 a 键全选，然后按回车确认，可能还需要输入Y确认，完成确认后，执行
```
pnpm install --force
```
--force会强制重新下载并执行所有安装步骤,或执行
```
pnpm install
```
此时这个问题就解决了。

## 问题3：@vueuse/core 这个包没有被正确解析到 Vite 的依赖中
查看日志，出现`Failed to resolve import "@vueuse/core" from "components/SakuraNavbar.vue"，`表示Vite找不到@vueuse/core包。虽然之前已执行pnpm approve-builds，但可能@vueuse/core并未在dependencies或devDependencies中显式列出，或者pnpm的依赖提升机制导致该包未被正确解析。

### 为什么 pnpm install 后还会报这个错？
在 pnpm 的严格依赖管理模式下，如果一个包（比如 valaxy-theme-sakura）依赖了 @vueuse/core，但你的顶层项目（即 package.json）没有直接声明它，那么 pnpm 不会将这个包提升到 node_modules 的根目录下，而是放在 .pnpm 文件夹里。

Vite 在分析 SakuraNavbar.vue 时，会尝试从 node_modules/@vueuse/core 加载，但找不到（因为根目录下没有）。即使它在 .pnpm 里，Vite 默认也不会去扫描嵌套位置，于是报错。

### 解决办法：
我们可尝试直接安装@vueuse/core作为直接依赖：pnpm add @vueuse/core。

在项目根目录执行：

```bash
pnpm add @vueuse/core
```

这会显式地把 @vueuse/core 写入 package.json 的 dependencies，pnpm 就会把它提升到根 node_modules，Vite 自然能找到。

## 问题4：Vite无法解析 "typeit" 这个包
报错来自 components/SakuraTypewriter.vue 中 import TypeIt from "typeit"。这是一个新的缺失依赖。类似于之前缺失 @vueuse/core 的情况。根据之前的经验，主题 sakura 可能依赖了 typeit，但项目没有显式安装。

### 解决办法：
在项目根目录执行：
```
pnpm add typeit
```

对应完成如上依赖包的安装，执行`pnpm dev`就可以实现界面显示啦。

