# 单词大闯关 完整开发流程

> 本文档记录「单词大闯关」从 0 到 1 的完整开发流程、架构决策、功能实现细节和迭代路径，便于后续维护、扩展和交接。

---

## 一、项目概述

### 1.1 产品定位

「单词大闯关」是一款面向小学 3-6 年级学生的**离线单词学习网页应用**，主打：
- **零安装**：单个 HTML 文件，双击即用
- **多模式练习**：拼写、选择、听音、看图等 6 种模式
- **游戏化激励**：得分、连击、星星、奖杯、打卡
- **数据持久化**：错题、打卡、设置、学习统计保存到浏览器本地
- **可扩展性**：支持导入自定义词包（作业单词）

### 1.2 目标用户

- 小学 3-6 年级学生
- 需要投影演示的课堂场景
- 家长辅导孩子课后复习

### 1.3 设计原则

1. **大字体、大按钮**：适合儿童操作和投影
2. **明亮卡通风格**：色彩鲜艳、有小狐狸 mascot
3. **即时反馈**：答对/答错都有音效、动画、语音反馈
4. **无网络依赖**：所有资源内联，无外部 CDN
5. **隐私优先**：数据只存本地 localStorage

---

## 二、技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| 结构 | HTML5 | 单页面应用，3 个 screen 切换 |
| 样式 | CSS3 | 内联样式，Flexbox/Grid 布局，CSS 动画 |
| 逻辑 | Vanilla JavaScript | 无框架，减少依赖 |
| 语音 | Web Speech API | `SpeechSynthesisUtterance` 朗读单词和例句 |
| 音效 | Web Audio API | Oscillator 生成简单音效 |
| 存储 | localStorage | 错题、打卡、设置、统计、自定义词包 |
| 版本 | Git + GitHub | 语义化版本，CHANGELOG 记录 |

---

## 三、文件结构

```
word-challenge/
├── index.html          # 主应用（完整单文件）
├── README.md           # 用户说明
├── CHANGELOG.md        # 版本更新日志
├── DEVELOPMENT.md      # 本开发流程文档
└── .git/               # Git 版本控制
```

> 注：为保持离线可用，CSS 和 JS 全部内联在 `index.html` 中。

---

## 四、架构设计

### 4.1 页面结构

应用包含 3 个主要屏幕（screen）：

1. **homeScreen（首页）**
   - 学习打卡卡片
   - 词包选择下拉框
   - 作业导入面板
   - 学习设置面板
   - 学习统计面板
   - 6 种模式选择卡片
   - 开始闯关按钮

2. **quizScreen（答题页）**
   - 进度条
   - 题号/得分/连击显示
   - 题目区域（动态显示中文/英文/图片/喇叭）
   - 输入框（写单词模式）
   - 选项卡片网格（卡片模式）
   - 反馈区域
   - 学习卡（答对后显示音标、自然拼读、例句）
   - 提交/回首页按钮

3. **resultScreen（结果页）**
   - 得分、正确率、星星评级
   - 今日打卡
   - 错题本及导出/导入/清空
   - 再练错词/回首页按钮

### 4.2 状态管理

使用单一 `appState` 对象管理应用状态：

```javascript
const appState = {
    wordPacks: {},              // 所有词包（内置 + 自定义）
    currentPackKey: "",         // 当前词包 key
    currentMode: "cn2en-write", // 当前模式
    questions: [],              // 本轮题目列表
    currentOptions: [],         // 当前选项（卡片模式）
    currentIndex: 0,            // 当前题号
    score: 0,                   // 当前得分
    streak: 0,                  // 当前连击
    maxStreak: 0,               // 最高连击
    wrongWords: [],             // 本轮/历史错题
    isRetryMode: false,         // 是否错题再练
    audioCtx: null,             // Web Audio 上下文
    currentUtterance: null,     // 当前朗读对象
    hasUserInteracted: false,   // 是否有过用户交互（用于自动播放策略）
    settings: {                 // 设置
        sound: true,
        rate: 0.8,
        questionCount: 8
    },
    stats: {                    // 学习统计
        totalWordsPracticed: new Set(),
        masteredWords: {},
        totalQuizzes: 0,
        totalCorrect: 0,
        totalQuestions: 0
    },
    keyboardSelection: -1       // 键盘选中选项索引
};
```

**设计理由**：
- 单文件应用，不需要 Redux/Vuex 等状态管理库
- 集中式状态便于调试和持久化
- 减少全局变量污染

---

## 五、词库数据设计

### 5.1 单词对象结构

```javascript
{
    en: "apple",           // 英文单词（唯一标识）
    cn: "苹果",            // 中文释义
    icon: "🍎",            // 图片/emoji（看图片选词模式用）
    phonetic: "/ˈæpl/",    // 音标
    phonics: "a-p-p-le",   // 自然拼读拆分
    accepts: ["苹果"]      // 可接受的中文答案数组
}
```

### 5.2 词包结构

```javascript
{
    "grade3-summer": {
        name: "🌱 三年级暑假复习词包",
        words: [/* ... */]
    }
}
```

### 5.3 例句数据

单独维护 `sentences` 对象，以英文单词为 key：

```javascript
const sentences = {
    "apple": { en: "I eat an apple.", cn: "我吃一个苹果。" }
};
```

**设计理由**：
- 例句不是所有模式都必需，单独维护避免词库膨胀
- 自定义导入的单词可以没有例句，应用能优雅降级

### 5.4 自定义词包

- 用户通过 textarea 粘贴作业单词
- 格式支持：空格、逗号、制表符分隔
- 支持 `#` 注释和空行
- 自动生成 icon、音标、自然拼读
- 保存到 localStorage，刷新不丢失

---

## 六、核心功能实现

### 6.1 模式系统

6 种模式对应不同的题目展示和判定逻辑：

| 模式 | 题目显示 | 答案输入方式 | 判定逻辑 |
|------|----------|--------------|----------|
| `cn2en-write` | 中文释义 | 输入框写英文 | 忽略大小写、去空格、完全匹配 |
| `en2cn-write` | 英文单词 | 输入框写中文 | 包含任意 accept 答案即算对 |
| `cn2en-card` | 中文释义 | 4 选 1 卡片 | 选择英文完全匹配 |
| `en2cn-card` | 英文单词 | 4 选 1 卡片 | 选择中文对应即可 |
| `listen` | 播放发音 | 4 选 1 卡片 | 听音选中文 |
| `picture` | emoji 图片 | 4 选 1 卡片 | 看图选英文 |

### 6.2 选项生成

从当前词包中随机抽取 3 个干扰项 + 1 个正确答案，打乱顺序：

```javascript
function generateOptions(correctWord, allWords) {
    const options = [correctWord];
    const others = allWords.filter(w => w.en !== correctWord.en);
    const shuffledOthers = shuffle(others);
    while (options.length < 4 && shuffledOthers.length > 0) {
        const candidate = shuffledOthers.pop();
        if (!options.some(o => o.en === candidate.en)) {
            options.push(candidate);
        }
    }
    return shuffle(options);
}
```

### 6.3 评分算法

- 基础分：答对 +10 分
- 连击加分：第 2 题起每题额外 +2 分
- 全对奖励：全部答对再 +20 分

```javascript
const baseScore = 10;
const streakBonus = streak >= 2 ? (streak - 1) * 2 : 0;
const points = baseScore + streakBonus;
```

### 6.4 星级评定

- 100% 正确：3 星
- ≥80%：2 星
- ≥60%：1 星
- <60%：无星

### 6.5 语音朗读

使用 Web Speech API：

```javascript
function speak(text, lang = "en-US") {
    return new Promise((resolve) => {
        if (!window.speechSynthesis) { resolve(); return; }
        stopSpeaking();
        const utter = new SpeechSynthesisUtterance(text);
        const voice = getPreferredVoice();
        if (voice) utter.voice = voice;
        utter.lang = lang;
        utter.rate = appState.settings.rate;
        utter.pitch = 1.0;
        utter.volume = 1;
        // ...
    });
}
```

**优化点**：
- 选择质量较好的系统语音（如 Google US English、Samantha 等）
- 每次朗读前 `cancel()` 之前的语音，避免重叠
- 语速可在设置中调节

### 6.6 音效反馈

使用 Web Audio API 生成简单波形：

- 答对：C5-E5-G5 三音上升
- 答错：低频下降音
- 胜利：C5-E5-G5-C6 庆祝音

### 6.7 错题本机制

- 答错时自动加入 `wrongWords` 数组
- 去重：相同单词不重复加入
- 持久化：保存到 localStorage
- 再练模式：只从错题中出题，答对后自动移除
- 支持导出、导入、清空

### 6.8 打卡系统

- 每次完成闯关可打卡一次
- 记录日期到 localStorage
- 计算连续打卡天数
- 首页和结果页显示打卡状态

### 6.9 学习统计

记录每个单词的练习情况：

```javascript
appState.stats.masteredWords[wordEn] = {
    count: 3,       // 连续答对次数
    date: "2026-06-28"  // 掌握日期
};
```

- 练习过的单词数
- 已掌握单词数（连续答对 3 次）
- 完成闯关次数
- 总正确率

---

## 七、UI/UX 设计细节

### 7.1 响应式断点

- 桌面端：三列模式卡片、两列选项卡片
- 移动端（<700px）：两列模式卡片、单列选项卡片、调整字体大小

### 7.2 动画使用

- 页面切换：淡入上移
-  mascot：idle 摇摆、happy 跳跃、sad 摇头
- 答对：星星爆炸
- 选项：hover 上浮、正确绿色高亮、错误红色抖动
- 进度条：彩虹渐变 + 星星标记

### 7.3 颜色系统

使用 CSS 变量统一管理：

```css
:root {
    --primary: #ff7043;      /* 主色：橙色 */
    --success: #66bb6a;      /* 成功：绿色 */
    --warning: #ffca28;      /* 警告：黄色 */
    --purple: #ab47bc;       /* 朗读按钮：紫色 */
    /* ... */
}
```

### 7.4 可访问性

- 大按钮、大字体
- 键盘支持（1-4 选择、ESC 返回）
- 明显的颜色对比
- 反馈信息清晰

---

## 八、数据持久化策略

使用 5 个 localStorage key：

| Key | 数据 | 用途 |
|-----|------|------|
| `wordChallengeCheckins` | 打卡日期数组 | 打卡系统 |
| `wordChallengeWrongWords` | 错题数组 | 错题本 |
| `wordChallengeCustomPack` | 自定义词包 | 作业导入 |
| `wordChallengeSettings` | 设置对象 | 用户偏好 |
| `wordChallengeStats` | 统计对象 | 学习统计 |

**注意事项**：
- localStorage 有 5MB 左右限制，对单词应用足够
- 所有读取都有 try-catch，防止数据损坏导致应用崩溃
- Set 类型存储前转为 Array

---

## 九、性能优化

1. **单文件**：减少 HTTP 请求
2. **内联资源**：无外部依赖
3. **避免重复 DOM 查询**：初始化时缓存 DOM 元素
4. **语音预加载**：页面加载时预加载 voices 列表
5. **动画优化**：使用 CSS transform 和 opacity，避免重排
6. **事件委托**：选项卡片使用事件监听但数量固定为 4 个，无需委托

---

## 十、测试清单

### 10.1 功能测试

- [ ] 6 种模式都能正常开始和结束
- [ ] 写单词模式大小写不敏感
- [ ] 卡片模式选项不重复
- [ ] 听音模式能自动播放
- [ ] 看图片模式 emoji 显示正常
- [ ] 错题自动记录和移除
- [ ] 自定义词包导入和持久化
- [ ] 设置（音效、语速、题量）保存生效
- [ ] 打卡功能正常工作
- [ ] 学习统计准确更新

### 10.2 兼容性测试

- [ ] Chrome/Edge/Safari 最新版
- [ ] 移动端浏览器
- [ ] 无网络环境
- [ ] 禁用 localStorage 的降级（应用仍可用，但不保存数据）

### 10.3 边界测试

- [ ] 词包只有 1 个单词时提示
- [ ] 卡片模式少于 4 个单词时提示
- [ ] 导入空文本/格式错误文本
- [ ] 连续快速点击提交
- [ ] 答题中途返回首页

---

## 十一、版本管理

### 11.1 版本号规则

采用语义化版本：
- 主版本号：重大架构变化
- 次版本号：新增功能
- 修订号：Bug 修复

### 11.2 Git 工作流

1. 在 `/Users/yinyin/word-challenge` 目录工作
2. 每次修改后执行：
   ```bash
   git add .
   git commit -m "类型: 简短描述"
   git push
   ```
3. 在 `CHANGELOG.md` 记录重要变更

### 11.3 历史版本归档

重大版本可在 `versions/` 目录保留副本，或在 Git tag 中标记：

```bash
git tag -a v1.1.0 -m "单词大闯关 v1.1.0"
git push origin --tags
```

---

## 十二、发布流程

1. 本地测试所有模式
2. 更新版本号和 CHANGELOG
3. commit + push
4. 在 GitHub 创建 Release（可选）
5. 用户通过 `git clone` 或直接下载 `index.html` 使用

---

## 十三、常见问题

**Q1: 语音朗读没声音？**
- 检查设备音量
- 检查浏览器是否允许自动播放
- 部分浏览器需要用户先点击页面才能播放音频

**Q2: 自定义词包刷新后消失？**
- v1.1.0 已修复，确保使用最新版
- 检查浏览器是否禁用 localStorage

**Q3: 看图片选词模式下图片不准确？**
- 自定义单词通过 emojiMap 自动匹配
- 复杂词汇可能匹配不到精确 emoji，会显示默认 📖

**Q4: 如何添加新的内置词包？**
- 编辑 `index.html` 中 `builtInPacks` 对象
- 按年级/主题添加新词包
- 为每个单词提供 en、cn、icon、phonetic、phonics、accepts

---

## 十四、未来扩展方向

1. **自然拼读独立应用**：从单词闯关中拆分出来，单独成页
2. **多用户支持**：为不同孩子建立独立学习档案
3. **复习算法**：根据艾宾浩斯遗忘曲线安排复习
4. **成就系统**：更多徽章和奖励
5. **打印功能**：生成纸质单词表/错题本
6. **PWA 化**：支持添加到主屏幕、离线缓存

---

## 十五、开发经验总结

1. **单文件应用适合快速交付**：无需构建工具，用户零门槛使用
2. **内联样式和脚本增加文件体积，但减少依赖**：对于离线场景值得
3. **Web Speech API 跨浏览器差异大**：需要准备降级方案
4. **localStorage 是简单持久化方案**：但要注意数据结构和容量
5. **游戏化设计提升儿童参与度**：即时反馈、连击、星星都很重要
6. **版本管理和文档不能少**：随着功能增加，CHANGELOG 和开发文档帮助后续维护
