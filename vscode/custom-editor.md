---
theme: smartblue
---

自定义编辑器允许插件创建一个基于`VS Code`标准文本编辑器的高度定制化的私人编辑器，这个编辑器可以用于特殊类型的资源，实际应用方面也有不少实际的应用：

* 资源预览，例如3D模型的shader
* 创建所见即所得的编辑器，如`Markdown`或`XAML`
* 提供`CVS`、`JSON`、`XML`文件数据的可视化渲染
* 为二进制或文本文件提供编辑能力

本文提供自定义编辑器API的概括性描述，并且提供了一个自定义编辑器实现的基础知识。我们会关注两种类型的自定义编辑器，告诉你它们的差异以及适用场景。

尽管自定义编辑器功能非常强大，但实现一个基础的自定义编辑器却并不复杂。实现自定义编辑器对于初学者来说，最复杂的是要接触大量的概念，诸如`webview`以及`text document`等，这点对于初学者来说可能是比较头疼的。

`VS Code`提供了官方的示例 [vscode-extension-samples](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample) 也是很好的学习资料，可以了解`API`的使用方法

本文涉及到的`API`：

* [registerCustomEditorProvider](https://code.visualstudio.com/api/references/vscode-api#window.registerCustomEditorProvider)
* [CustomTextEditorProvider](https://code.visualstudio.com/api/references/vscode-api#CustomTextEditorProvider)

## API基础用法

自定义编辑器是替代`VS Code`的标准文本编辑器来显示特定资源的另一种视图，它包含两个部分：用户交互的视图和插件中用于与底层资源交互的文档模型

视图部分采用`webview`实现，开发者采用HTML、CSS、JavaScript来构建用户交互，`webview`可以通过`VS Code API`直接和组件相互传递信息，可以查看我们上一篇文章 [VS Code插件开发教程（8） Webview](https://juejin.cn/post/6974440642982182920) 来获取个更多关于`webview`的信息

文档模型部分，模型的作用是让插件了解它所使用的资源。`CustomTextEditorProvider`利用`VS Code`的标准 [纯文本文档](https://code.visualstudio.com/api/references/vscode-api#TextDocument) 作为它的文档模型，所有的文件变化都是用`VS Code`的标准文本编辑`API`。`CustomReadonlyEditorProvider`和`CustomEditorProvider`会让你创建自己的文档模型用于哪些非文本文件

自定义编辑器对每一种资源都有一个单独文档模型，不过对于同一个文档可能有多个编辑器实例，例如使用`View: Split editor`功能将同一个文档一分为二

### CustomEditor/CustomTextEditor概览

有两类自定义编辑器:自定义文本编辑器和自定义编辑器。二者之间最大的不同是它们定义文档模型的方式

一个`CustomTextEditorProvider`利用`VS Code`的标准 [纯文本文档](https://code.visualstudio.com/api/references/vscode-api#TextDocument) 作为数据模型，你可以对任何基于文本的文件类型使用`CustomTextEditor`。`CustomTextEditor`使用起来颇为容易，因为VS Code已经知道如何处理文本文件，可以实现多种操作，如保存和备份文件热退出。

`CustomEditorProvider`则是需要插件用自己的文档模型，这意味着你可以利用`CustomEditor`来处理独特的二进制文件如图片等，当然这也意味着插件需要做更多的工作，例如实现保存和后退（如果你的编辑器只用来读取预览文件，这些复杂的功能也可以省略）。

### Contribution point

我们需要利用`customEditors`来配置自定义编辑器所需要的信息，例如告知`VS Code`将哪种文件和自定义编辑器相关联，如下示例：

```json
"contributes": {
    "customEditors": [{
        "viewType": "catEdit.catScratch",
        "displayName": "Cat Scratch",
        "selector": [{
            "filenamePattern": "*.cscratch"
        }],
        "priority": "default"
    }]
}
```

`customEditors`字段的取值是个数组，每个插件可以注册若干自定义编辑器，接下来我们将其逐个字段做介绍：

* `viewType`：自定义编辑器的唯一标识符，`VS Code`据此来将`package.json`中注册的自定义编辑器和具体的代码对应，该取值必须是跨插件唯一的
* `displayName`：在`VS Code's UI`中用来展示自定义编辑器时使用，例如`View: Reopen with`命令的下拉框中显示
* `selector`：定义那种文件会激活自定义编辑器，取值是一个`glob patterns`数组，既可以定义具体的文件名称，如`*.png`匹配全部的`PNG`文件，也可以定义文件夹或路径，如`**/translations/*.json`
* `priority`：可选，标示自定义编辑器何时启用，可取值有两个：
  * `default`：根据`selector`为每个匹配的文件启用自定义编辑器，如果匹配成功了多个编辑器，则交由用户选择使用哪个
  * `option`：默认不启用自定义编辑器，而是让用户决定

### 自定义编辑器的启动

当一个用户打开了自定义编辑器，`VS Code`会触发`onCustomEditor:VIEW_TYPE`激活事件，然后插件必须在代码里通过调用`registerCustomEditorProvider`方法来用指定的`viewType`注册自定义编辑器。`onCustomEditor:VIEW_TYPE`只在`VS Code`创建了编辑器实例的时候才会被触发，用`View: Reopen with`展示编辑器列表的时候不会触发。

## 自定义文本编辑器

### 生命周期


























