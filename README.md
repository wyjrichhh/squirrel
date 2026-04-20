    鼠鬚管
    爲物雖微情不淺
    新詩醉墨時一揮
    別後寄我無辭遠

    　　　——歐陽修

今由　[中州韻輸入法引擎／Rime Input Method Engine](https://rime.im)
及其他開源技術強力驅動

【鼠鬚管】輸入法
===
[![Download](https://img.shields.io/github/v/release/rime/squirrel)](https://github.com/rime/squirrel/releases/latest)
[![Build Status](https://github.com/rime/squirrel/actions/workflows/commit-ci.yml/badge.svg)](https://github.com/rime/squirrel/actions/workflows)
[![GitHub Tag](https://img.shields.io/github/tag/rime/squirrel.svg)](https://github.com/rime/squirrel)

式恕堂 版權所無

授權條款：[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)

項目主頁：[rime.im](https://rime.im)

您可能還需要 Rime 用於其他操作系統的發行版：

  * 【中州韻】（ibus-rime、fcitx-rime）用於 Linux
  * 【小狼毫】用於 Windows

---

## 关于本 fork（`feat/ai-inference` 分支）

本分支在上游 `rime/squirrel` 之上添加了 3 处通用前端增强，主要用于配合 [librime-ai-predict](https://github.com/wyjrichhh/librime-ai-predict)（基于 CTranslate2 的神经网络候选预测插件）实现端到端体验：

| Commit | 改动 | 是否绑定 ai_predict |
|---|---|---|
| `chore: store logs under ~/Library/Logs/Squirrel` | 把日志路径迁出 `TMPDIR`，便于在 IMK sandbox 外查看插件日志 | 否，通用增强 |
| `feat: per-comment color overrides via style/comment_color_map` | 主题层支持按候选 comment 文本（如 `AI`）单独配色 | 否，通用 theming API |
| `feat: refresh UI when librime sets ai_predict/ready` | 收到 librime `property` 通知时主动刷新候选栏 | 是，监听了 `ai_predict/ready` 属性 |

> 这 3 个 commit 都是非侵入的增量，未修改任何上游文件的核心行为；本分支的 `librime` 子模块仍指向上游 `rime/librime`，**不依赖 librime fork**。

### 端到端编译并安装（约 15–30 分钟，含模型下载）

> 假设你在一个空目录 `~/work/` 下从零开始；所有命令均按顺序执行即可，**百分百可复现**。

#### 0. 前置依赖

- macOS 13.0+
- Xcode 14.0+（`xcode-select --install`）
- `brew install cmake boost`
- 可访问 GitHub（拉取若干仓库与子模块）

#### 1. 克隆本 fork（含子模块）

```bash
cd ~/work
git clone --recursive -b feat/ai-inference \
    https://github.com/wyjrichhh/squirrel.git
cd squirrel
```

#### 2. 注册 librime-ai-predict 插件并预编译其依赖

```bash
bash librime/install-plugins.sh wyjrichhh/librime-ai-predict
( cd librime/plugins/ai-predict && make deps )
```

第一行用 squirrel/librime 自带的 `install-plugins.sh` 把插件 clone 到 `librime/plugins/ai-predict/`（目录名是脚本剥离 `librime-` 前缀后的结果）。
第二行进入插件目录预编译它依赖的 CTranslate2 静态库（产物：`include/`、`lib/libctranslate2.a`）。

#### 3. 设置 BOOST_ROOT 并编译 squirrel

如果你用 Homebrew 装的 Boost：

```bash
export BOOST_ROOT="$(brew --prefix boost)"
make deps    # 编译 librime（含 ai_predict 插件）+ plum-data + opencc-data
make         # 编译 Squirrel.app
```

> 想用 Universal binary，参考 [INSTALL.md](INSTALL.md) 的 `BUILD_UNIVERSAL=1` 等环境变量。

#### 4. 安装到系统并激活

```bash
sudo make install
```

安装到 `/Library/Input Methods/Squirrel.app`。**注销并重新登录** macOS 让 IMK 重新加载输入法。

#### 5. 下载 AI 模型

```bash
( cd librime/plugins/ai-predict && ./scripts/download_model.sh )
```

模型（约 300MB）会被下载并解压到 `~/Library/Rime/predict_models/zh-base-ct2-int8/`。

#### 6. 启用模块与方案

编辑 `~/Library/Rime/` 下的：

- `default.custom.yaml`：在 `patch.modules` 中加入 `- ai_predict`
- `luna_pinyin.custom.yaml`（或你用的 schema 的 `*.custom.yaml`）：按 [librime-ai-predict/examples/schema.fragment.yaml](https://github.com/wyjrichhh/librime-ai-predict/blob/main/examples/schema.fragment.yaml) 合并 `engine/translators`、`engine/filters` 与 `ai_predict` 段

#### 7. 部署

鼠须管菜单 → 「重新部署」。完成后切到目标方案，输入拼音串（默认有 200ms 防抖窗口）。AI 候选会出现在第 2 位（默认）；触发模式与阈值由插件配置 `ai_predict/min_input_length` 控制（默认 6 字节，详见插件 README）。

> 配置项与候选位策略详见 [librime-ai-predict README](https://github.com/wyjrichhh/librime-ai-predict#模块与配置名避免与官方-predict-冲突)。

### 常见问题

| 现象 | 检查 |
|---|---|
| 编译时 `BOOST_ROOT` 报错 | 确认 `echo $BOOST_ROOT` 非空且目录存在 |
| `make deps` 卡在 librime 编译 | 网络或 git 子模块问题；可单独重试 `cd librime && make deps` |
| 安装后无 AI 候选 | 检查 `~/Library/Logs/Squirrel/` 下日志；确认 schema 的 `engine/translators` 第一位是 `ai_predict_translator` |
| 候选栏不刷新 | 注销重登；或确认本 fork 已正确安装（不是上游版本覆盖） |

---

安裝輸入法
---

本品適用於 macOS 13.0+

初次安裝，如果在部份應用程序中打不出字，請註銷並重新登錄。

使用輸入法
---

選取輸入法指示器菜單裏的【ㄓ】字樣圖標，開始用鼠鬚管寫字。
通過快捷鍵 `` Ctrl+` `` 或 `F4` 呼出方案選單、切換輸入方式。

定製輸入法
---

定製方法，請參考線上 [幫助文檔](https://rime.im/docs/)。

使用系統輸入法菜單：

  * 選中「在線文檔」可打開以上網址
  * 編輯用戶設定後，選擇「重新部署」以令修改生效

安裝輸入方案
---

使用 [/plum/](https://github.com/rime/plum) 配置管理器獲取更多輸入方案。

致謝
---

輸入方案設計：

  * 【朙月拼音】系列

    感謝 CC-CEDICT、Android 拼音、新酷音、opencc 等開源項目

程序設計：

  * 佛振
  * Linghua Zhang
  * Chongyu Zhu
  * 雪齋
  * faberii
  * Chun-wei Kuo
  * Junlu Cheng
  * Jak Wings
  * xiehuc

美術：

  * 圖標設計 佛振、梁海、雨過之後
  * 配色方案 Aben、Chongyu Zhu、skoj、Superoutman、佛振、梁海

本品引用了以下開源軟件：

  * Boost C++ Libraries  (Boost Software License)
  * capnproto (MIT License)
  * darts-clone  (New BSD License)
  * google-glog  (New BSD License)
  * Google Test  (New BSD License)
  * LevelDB  (New BSD License)
  * librime  (New BSD License)
  * OpenCC / 開放中文轉換  (Apache License 2.0)
  * plum / 東風破 (GNU Lesser General Public License 3.0)
  * Sparkle  (MIT License)
  * UTF8-CPP  (Boost Software License)
  * yaml-cpp  (MIT License)

感謝王公子捐贈開發用機。

問題與反饋
---

發現程序有 BUG，或建議，或感想，請反饋到 [Rime 代碼之家討論區](https://github.com/rime/home/discussions)

聯繫方式
---

技術交流，歡迎光臨 [Rime 代碼之家](https://github.com/rime/home)，
或致信 Rime 開發者 <rimeime@gmail.com>。

謝謝
