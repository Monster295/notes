# Keep-Alive 与 Iframe 的机制冲突

**问题描述：**

切换tab页签之后，elnExcel表格中的数据会丢失，变成空白

**原因分析：**

1.  **Keep-Alive 与 Iframe 的机制冲突**：
    `elnExcel` 组件在 `noteBook` 中使用，而 `noteBook` 页面（或者其父级布局）使用了 `keep-alive` 来缓存组件状态。当切换 Tab 页签时，组件被 deactivated（停用），随后切回时被 activated（激活）。
    Vue 的 `keep-alive` 在处理包含 `iframe` 的组件时存在一个浏览器层面的特性：**当 iframe 所在的 DOM 节点被移除（或隐藏导致重新渲染）并重新插入文档流时，浏览器会强制重新加载 iframe 的内容**。
    这意味着 `LuckySheetInstance` 中的 iframe 在切换 Tab 后实际上已经重置了，变成了初始的空白页。

2.  **组件初始化逻辑阻断**：
    在 `@src/views/zs-rdm/components/luckysheet/LuckySheetInstance.vue` 组件中，`handleInit` 方法有一个检查：
    ```javascript
    handleInit() {
      if (this.hasInitialized) return; // 阻断逻辑
      this.hasInitialized = true;
      // ... 初始化逻辑
    }
    ```
    因为组件被 `keep-alive` 缓存了，Vue 实例并没有销毁，`this.hasInitialized` 状态仍然保持为 `true`。
    当 iframe 重新加载完成并发送 `FRAME_READY` 事件时，触发 `onFrameReady` -> `handleInit`，但由于 `hasInitialized` 为 `true`，初始化代码直接返回，导致 LuckySheet 没有重新注入数据和配置，从而显示为空白。

**解决方案：**

修改 `LuckySheetInstance.vue` 中的 `onFrameReady` 方法。当接收到 iframe 的 `ready` 事件时（说明 iframe 刚刚加载完成），显式将 `this.hasInitialized` 重置为 `false`，从而允许组件重新执行初始化流程，将数据再次渲染到新的 iframe 中。

**已修复代码：**

我已修改 `@src/views/zs-rdm/components/luckysheet/LuckySheetInstance.vue` 文件：

```javascript
    onFrameReady() {
      // frame准备就绪（可能是首次加载，也可能是keep-alive导致的iframe重载），重置初始化状态
      this.hasInitialized = false
      if (this.lazyLoad) {
        // 如果懒加载开启， 则不会自动初始化
        return
      }
      this.handleInit()
    },
```

这样，每次切换 Tab 切回时，iframe 重载触发 `ready`，组件会重新把数据（`this.value`）注入进去，解决了空白问题。

**注意：**
由于 iframe 重载意味着之前的上下文丢失，如果用户在切换 Tab 前有**未保存的编辑内容**，且这些内容没有实时同步回父组件的 `value` 属性，那么重新初始化后会恢复到最后一次保存的状态（即 `value` 的值）。这是 `iframe` + `keep-alive` 架构下的固有限制。如果需要完全保留未保存的编辑状态，需要实现实时的双向绑定或在切换前同步数据。但目前的修复已解决了“变成空白”这一严重 bug。
