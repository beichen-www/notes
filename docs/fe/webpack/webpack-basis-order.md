# webpack 常用指令

1. Entry：入口指示， Webpack 以哪个为入口开始打包，分析构建内部以来图

2. output ：输出指示，Webpack 打包后的资源 bundles 输出到哪里去，以什么命名

3. loader : loader 让 Webpack 能去处理那些非 Javascript 文件 ( Webpack 自身只能理解 Jacascript )，例如：less sass ,img css ...

4. plugins : 插件用于执行范围更广的任务，插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量等

5. Mode：指示 Webpack 使用相应模式的配置。如下所示

|    选项     |                                                                                     描述                                                                                     |            特点            |
| :---------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------: |
| development |                                                   将会 process.env.NODE_ENV 启用 NamedChunksPlugiOn 和 NamedModulesPlugin                                                    |  能让代码本地调试运行环境  |
|  productin  | 将会 process.env.NODE_ENV.，启用 FlagDeoendencyUsagePlugin、 FlagIncudedChunksPlugin、FlaginckudedChunksPugin、OccurrenceOrderPlugin、SideEffectsFlagPlugin 、 UglifyjsPugin | 能让代码优化上线运行的环境 |
