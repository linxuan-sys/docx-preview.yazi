# 📄 docx-preview.yazi

> **在 [Yazi](https://github.com/s-yazi/yazi) 文件管理器里，秒速预览 Word 文档。**

只用把光标停在 `.docx` 文件上，右侧预览面板就会立刻显示文档内容 —— 不用打开 Word / WPS，不用等界面加载。插件会在后台调用 `pandoc` 把 Word 文档实时转成纯文本，再按预览面板的大小分页渲染出来，真正意义上的**零延迟** ⚡

---

## ✨ 特性

- ⚡ **即时预览** —— 光标一停，内容即现，连回车都不用按
- 🔄 **实时转换** —— 每次预览都现调 `pandoc`，磁盘上不留任何临时文件
- 🪶 **轻量纯净** —— 纯 Lua 实现，除 `pandoc` 外不需要任何其他依赖
- 📜 **自动分页** —— 长文档自动按面板高度切页，支持滚动翻页浏览
- 🛡️ **容错友好** —— `pandoc` 缺失、转换失败、空文档等情况都有清晰的中文提示
- 🎯 **无侵入** —— 不干扰 Yazi 原有的预览逻辑，只对 `.docx` 文件生效
- 🌏 **中文友好** —— 中文文档的内容可以正常提取和显示

---

## 📦 依赖

| 依赖 | 是否必需 | 用途 |
| :--- | :---: | :--- |
| [Yazi](https://github.com/s-yazi/yazi) | ✅ | 运行环境（已在 **Yazi 26.9.1** 上测试通过） |
| [pandoc](https://pandoc.org/) | ✅ | 负责把 `.docx` 转换成纯文本（已在 **pandoc 3.12.1** 上测试通过） |

安装 `pandoc`：

```bash
# Arch Linux
sudo pacman -S pandoc

# Debian / Ubuntu
sudo apt install pandoc

# Fedora
sudo dnf install pandoc

# macOS (Homebrew)
brew install pandoc

# Windows (Scoop)
scoop install pandoc
```

---

## 🚀 安装

**方式一：用 Yazi 自带的包管理器（推荐）** 🎉

```bash
ya pkg add linxuan-sys/docx-preview
```

**方式二：手动安装**

```bash
mkdir -p ~/.config/yazi/plugins/docx-preview.yazi
# 把本仓库的 main.lua 复制进去即可
```

---

## ⚙️ 配置

在 `~/.config/yazi/yazi.toml` 里把 `docx-preview` 挂到预览器列表的最前面：

```toml
[plugin]
prepend_previewers = [
	{ url = "*.docx", run = "docx-preview" },
]
```

> 💡 `prepend_previewers` 会把这条规则插到 Yazi 内置预览器之前，保证 `.docx` 文件由本插件接管；
> `url = "*.docx"` 表示匹配所有 Word 文档，也可以换成按 MIME 类型匹配：
> `{ mime = "application/vnd.openxmlformats-officedocument.wordprocessingml.document", run = "docx-preview" }`。

配置保存后重启 Yazi 即可生效 🚀

---

## 💡 使用

1. 用 Yazi 打开任意目录，把光标移动到 `.docx` 文件上 👀
2. 右侧预览面板会立刻显示文档的纯文本内容
3. 长文档用 `J` / `K` 在预览区上下滚动（这是 Yazi 的默认 `seek` 键位，你也可以在 `keymap.toml` 里绑定到自己习惯的按键上）
4. 想用外部程序打开原文件？直接按 `Enter` 走系统默认程序即可

---

## 🧩 工作原理

插件做的事情非常纯粹：

```bash
pandoc -t plain --wrap=auto --columns=100 <你的文档.docx>
```

1. 调用 `pandoc` 把 Word 文档转换成纯文本（`-t plain`）
2. 把结果按行拆分，结合 `job.skip` 和当前面板高度做分页
3. 交给 Yazi 的 `ui.Text` 渲染到预览面板

转换结果全程只走内存（标准输出），**不会产生任何临时文件**，所以既快又干净。滚动预览则由 `seek()` 负责，Yazi 每次滚动都会重新触发一次分页渲染。

---

## ❓ 常见问题

<details>
<summary><b>预览面板出现红色报错 <code>pandoc command failed</code>？</b></summary>

说明系统里没有安装 `pandoc`，或者它不在 `PATH` 里。按上面「依赖」一节装好即可。
</details>

<details>
<summary><b>预览是空白的？</b></summary>

文档本身可能没有可提取的文本内容（例如纯图片、扫描件）。`pandoc` 只能提取文字，提取不到图片。
</details>

<details>
<summary><b>表格、公式、图片会显示吗？</b></summary>

不会哦。预览输出的目标是「纯文本」，复杂排版会被拍平成文字，公式和图片会被直接忽略 🙈
</details>

<details>
<summary><b>支持老的 <code>.doc</code> 格式吗？</b></summary>

不支持。`.doc` 是古老的二进制格式，`pandoc` 无法可靠解析，建议先用 Word/WPS 另存为 `.docx`。
</details>

<details>
<summary><b>想调整换行宽度 / 预览效果？</b></summary>

打开 `main.lua`，把 `--columns=100` 里的数字改成你喜欢的大小即可。
</details>

<details>
<summary><b>中文会乱码吗？</b></summary>

正常不会。只要终端字体和编码设置没问题，中文内容可以正确显示 🌏
</details>

<details>
<summary><b>打开超大的文档会卡吗？</b></summary>

每次预览都会调用一次 `pandoc`，几百页的巨型文档会有一点点延迟，但因为有分页机制，滚动本身很流畅。
</details>

---

## 📁 目录结构

```
docx-preview.yazi/
├── main.lua     # 🧠 插件主体：peek() 负责预览，seek() 负责滚动
├── README.md    # 📖 就是你正在看的这份说明文档
└── LICENSE      # 📄 MIT 许可证
```

---

## 📄 许可证

本项目基于 [MIT License](LICENSE) 开源，可以自由使用、修改和分发。

---

## 🤝 致谢

- [Yazi](https://github.com/s-yazi/yazi) —— 快得离谱的终端文件管理器 🦆
- [pandoc](https://pandoc.org/) —— 文档转换界的瑞士军刀 🔧

---

<div align="center">

**如果这个插件帮到了你，欢迎点个 ⭐ Star 支持一下～**

Made with ❤️ by [linxuan-sys](https://github.com/linxuan-sys)

</div>
