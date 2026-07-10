# 维护指南（供 AI coding agent 与人类维护者）

本仓库是「雷诺曼秘仪 · Lenormand Oracle」：一个**单文件、零依赖、纯静态**的雷诺曼占卜工具。
线上地址：https://lynnnnnnnnnew.github.io/lenormandforyou/ （GitHub Pages，main 分支根目录）。

## 文件结构

```
index.html      # 全部产品：HTML + CSS + JS 都在这一个文件里
牌灵之墓.jpg     # 祭坛照片（Mlle Lenormand 之墓，CC BY-SA 4.0）
README.md       # 面向访客的说明 + 设计决策记录
docs/           # README 用的截图
AGENTS.md       # 本文件
```

## 不可破坏的约定（改任何东西前先读这段）

1. **保持单文件零依赖。** 不引入构建步骤、框架、npm、外部 CSS/JS——唯一允许的外部脚本是页尾的 GoatCounter 统计。新功能直接写进 index.html。
2. **隐私承诺是产品底线。** 占卜内容（问题、牌局）永远只存在用户浏览器的 localStorage，不得上传到任何服务器；除 GoatCounter 匿名计数外不得添加任何数据采集。若改动涉及数据流向，必须同步更新页面底部「隐私公告」和 README 的隐私一节，三处口径一致。
3. **随机性分层不可动摇。** 洗牌只用 `crypto.getRandomValues`（`secureInt` 含拒绝采样修正，保证均匀）；用户的落指坐标/微秒时刻/问题文字/封印语只允许进入**切牌哈希**（`fnv1a`），不允许影响洗牌本身的均匀性。
4. **Prompt 的解读要求语义不得删减。** `buildPrompt()` 里的 7 条要求各有来由，尤其：第 4 条「是非题铁律」（牌性多数决优先于叙事，源于一次真实误读事故）和第 7 条「勿沉迷」提醒。36 张牌的 `CARDS` 数组每张必须带牌性标注（第 7 个字段）。
5. **版权署名不得移除。** 墓照为 Pierre-Yves Beaudouin / Wikimedia Commons / CC BY-SA 4.0，署名出现在页面祭坛小字和 README，两处都要保留。
6. **界面语言为中文，配色为深蓝配金**（CSS 变量在 `:root`，金色 `--gold:#d4af37` 是身份色，不要改紫/改亮）。

## 代码地图（index.html 内）

| 区块 | 内容 |
|---|---|
| `:root` + CSS | 主题变量、面板、卡牌翻转动画、北斗星区、祭坛、隐私公告 |
| `CARDS` | 36 张牌：`[编号, 中文名, 英文名, emoji, 扑克映射, 牌意, 牌性]` |
| `SPREADS` | 四种牌阵：n（张数）、pos（位置名）、guide（给 AI 的阵法解读指引） |
| 天时 | `moonInfo()` 月相（已知新月历元推算）、`shichen()` 十二时辰 |
| 熵池 | 北斗七星点按：`STAR_POS`（勺形坐标%）、顺序点亮、乱序落指计熵不计进度、引导线（点亮第 N 颗即点亮通往 N+1 的线） |
| 洗牌 | `secureInt()`、`drawCards()`（七次 Fisher-Yates + 封印语/熵/问题/时刻哈希切牌） |
| 流程 | 选阵 → 点星 → 按住 ≥1.5s 松手 → `reveal()` 翻牌 → `buildPrompt()` |
| 存档 | localStorage 静默保存（上限 200 条）、下载本次 txt、导出全部历史；**不显示保存提示**（曾有，因造成焦虑感而移除，勿加回） |

## 修改后的测试清单

手动或用无头浏览器（headless Chrome 拍 `--window-size=780,3200 --virtual-time-budget=15000` 可跑完整流程）验证：

- [ ] 七星只能按顺序点亮；乱序点按不推进进度；点亮第 N 颗时出现通往第 N+1 颗的金线
- [ ] 按住抽牌按钮不足 1.5 秒松手：提示重按，不开牌
- [ ] 开牌后：牌数正确、无重复、位置标签正确
- [ ] 占卜全文包含：问题、时间+时辰、月相、熵样本数、切牌位、每张牌的牌性、7 条解读要求
- [ ] 复制按钮、下载本次 txt、导出全部历史均可用；localStorage 记录累积正常
- [ ] 「再占一次」：星、线、进度条、结果面板全部复位
- [ ] 375px 宽度无横向溢出，复制按钮不折行
- [ ] 浏览器控制台无报错

## 部署

```
git push origin main   # GitHub Pages 自动部署，约 1–2 分钟生效
```

验证：`curl -s "https://lynnnnnnnnnew.github.io/lenormandforyou/?v=$(date +%s)" | grep <本次改动的特征串>`

## 已知边界

- GoatCounter 不统计 localhost 访问，本地测试不污染数据
- README 截图由临时 demo 页（复制 index.html + 注入自动播放脚本）无头拍摄，UI 有可见改动时需重拍 docs/ 下两张图
- github.io 在中国大陆访问不稳定，属托管平台特性，非本项目 bug
