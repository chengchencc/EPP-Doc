# iGIX 首页 Dashboard

> (图片待补充)



## 介绍

- **页面**：**一堆部件** 和 **布局** 组成的页面
- **部件**：页面上的一小块，由 **元部件** 配置而成
- **元部件**：部件的模板，是代码包（可用 Widget cli 方便地开发，详情见 [《元部件开发》](./iGIX Dashboard 元部件开发.md)）
- **数据源**：数据获取方式
- **数据集**：数据的详细配置



## 页面

iGIX 预制了一个**系统预制（默认）首页** “我的工作中心”。

**用户** 可以通过首页右上角切换、编辑、新建、管理自己的页面。

**管理员** 可以进入 “页面” 菜单中，点击 “新增页面” 创建公共页面，并给页面分配功能组 —— 拥有此功能组的用户，都能在首页列表中看到它。

**管理员** 还可以把某页面设置为所有用户的 **默认首页**（当前暂不支持）。

新建、编辑页面都通过图形化的编辑器 “**页面设计器**” 实现。



## 部件

小部件是页面的组成部分，系统预制了一些常用公共部件。

管理员可以在 “部件” 中点击 “新增部件” 来增加新部件。相比较于设计页面，部件需要更多配置：

1. 根据需要 **选择元部件**，元部件是部件的呈现主体；
2. 若元部件需要 **数据配置**，则可以选择数据集，配置数据；若元部件不需要数据配置，忽略此步；
3. **配置部件属性**，不同元部件有不同配置项。

期间可随时保存。

部件创建完成后，就可以在 “页面设计器” 中拖到页面上啦。



## 元部件

元部件是部件的呈现主体，它决定了部件的功能、可能样貌、可配置项、数据获取方式等。其实质是一块前端代码。

iGIX 预制了一堆常用元部件，应优先考虑使用它们来配置部件。

若预制元部件无法满足需要，或有更好的 idea，可以用元部件脚手架工具开发元部件，参考 [元部件开发](./iGIX Dashboard 元部件开发.md)。

元部件开发完成后，打开 “元部件” 功能中的 “新增元部件”，填写 SU，把元部件的 `release.zip` 文件上传，保存，元部件就新增好咯。

元部件文件都保存在 `<安装盘>\web\runtime\web\metaportlet\<你的SU>` 下，之后若想改变元部件的文件，如 main.js，logo.png 等，直接放进去即可，不需要再次上传。
