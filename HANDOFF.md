# Handoff: 单词大闯关 + 自然拼读应用

**日期**: 2026-06-29
**交接人**: Claude Code (当前会话)
**接棒人**: 下一个 Claude 实例

---

## 1. 项目仓库

- **GitHub**: https://github.com/yinxiaowei10/word-challenge
- **本地路径**: `/Users/yinyin/word-challenge/`
- **当前最新提交**: `0a79511` (已 push)

---

## 2. 单词大闯关 (index.html)

### 最新状态

- v1.1.0 已发布并 push
- 修复了**答案泄露问题**：答题时不显示音标/自然拼读/例句，答对后才展示学习卡
- 修复了**配音问题**：优选 Google US English / Samantha / Microsoft David 等更自然语音，音高 1.0
- 延长了**停留时间**：答对 3.5-4 秒，答错 5.5-6 秒
- 页面加载时预加载 speech voices

### 核心文件

- `index.html` — 主应用
- `DEVELOPMENT.md` — 完整开发流程文档
- `CHANGELOG.md` — 版本日志
- `README.md` — 用户说明

### 待优化项

- 可继续扩展真实教材词库（当前为精简示例词库）
- 可加入复习算法（艾宾浩斯）
- 可考虑 PWA 化

---

## 3. 自然拼读应用 (phonics.html)

### 最新状态

- 框架已搭建，文件 `phonics.html`
- 当前使用**示例数据**，需要替换为真实牛津自然拼读课程数据
- 已实现 5 个级别 + 4 种学习模式的页面结构

### 架构

```
phonics.html
├── 首页：选择 Level 1-5
├── 单元页：选择 Unit
├── 模式页：学习/听音/拼读/写词
├── 学习模式：展示字母/组合 + 例词
├── 听音辨音：播放发音，选择字母/组合
├── 拼读练习：拆分单词拼读
└── 听音写词：听发音写单词
```

### 课程数据结构

```javascript
const phonicsData = {
    levels: [
        {
            id: 1,
            title: "The Alphabet",
            desc: "26 个字母发音",
            units: [
                {
                    id: "l1u1",
                    title: "A, B, C",
                    phonemes: [
                        {
                            symbol: "a",
                            sound: "/æ/",
                            keyword: "apple",
                            icon: "🍎",
                            words: ["apple", "ant", "ax"]
                        }
                    ]
                }
            ]
        }
    ]
};
```

### 已获取资料

百度网盘 skill 已安装并登录，资料已迁移到 `/apps/bdpan/phonics-materials/`：

```
/apps/bdpan/phonics-materials/
├── 043【Sam老师】超级英语全集/
│   ├── 1 Sam老师【超级拼读】
│   ├── 2 Sam老师【超级语法】
│   ├── 3.Sam老师【超级阅读】课程
│   ├── 4.Sam老师【超级单词】课程
│   ├── 5单词拆分课【单词速记】
│   ├── 6.发音训练  44节
│   ├── 7.每天7点晨读20节 （文本和音频）
│   └── 8 Sam「拼读+语法+音标3合1」
│
├── 2、学生用书PDF + 练习册PDF + 音频MP3 + 视频MP4（1-5级）/
│   ├── 1/Oxford_Phonics_World_1_SB.pdf (已下载到 /tmp/)
│   ├── 2/
│   ├── 3/
│   ├── 4/
│   └── 5/
│
├── 5、闪卡1-3级 + 学生卡1-5级 + 单词卡1-5级/
└── 6、教师用书 1-5级/
```

### 已读取内容

- 已下载 `Oxford_Phonics_World_1_SB.pdf` (Level 1)
- PDF 是扫描版图片，无法直接提取文字
- 已用 `pdftoppm` 转成 PNG 并读取了目录页和 Unit 1 Aa/Bb 的前几页
- Level 1 结构确认：
  - Unit 1: Aa Bb Cc
  - Unit 2: Dd Ee Ff
  - Unit 3: Gg Hh Ii
  - Unit 4: Jj Kk Ll
  - Unit 5: Mm Nn Oo
  - Unit 6: Pp Qq Rr
  - Unit 7: Ss Tt Uu Vv
  - Unit 8: Ww Xx Yy Zz
  - Review 1-4
  - The Alphabet Song, Student Cards, Certificate

---

## 4. 下一步工作

### 优先级 1：填充自然拼读课程数据

1. **继续读取牛津 PDF**
   - 已下载 Level 1，需要提取全部 8 个 Unit 的单词表
   - 需要下载 Level 2-5 的学生用书 PDF
   - 每个 Level 转成图片 → 用视觉模型/OCR 提取单词和发音

2. **整理数据结构**
   - 把提取的内容整理成 `phonicsData` 格式
   - 为每个 phoneme 准备：symbol, sound, keyword, icon, words
   - 参考牛津自然拼读级别：
     - Level 1: The Alphabet
     - Level 2: Short Vowels
     - Level 3: Long Vowels
     - Level 4: Consonant Blends
     - Level 5: Letter Combinations

3. **参考 Sam 老师教学法**
   - 查看 `043【Sam老师】超级英语全集/1 Sam老师【超级拼读】` 内容
   - 提取 Sam 的拼读技巧、口诀、教学顺序
   - 融入到应用的教学提示和反馈文案中

### 优先级 2：完善自然拼读应用

1. 替换示例数据为真实数据
2. 增加每个 phoneme 的例句/chant
3. 增加游戏化元素：星星、奖杯、进度保存
4. 增加 localStorage 学习进度
5. 增加发音口型图或动画提示
6. 移动端适配测试

### 优先级 3：单词闯关继续优化

- 根据用户反馈继续迭代
- 考虑加入更丰富的内置词库
- 考虑增加家长报告功能

---

## 5. 关键技术信息

### 百度网盘 Skill

- 已安装：`~/memory-work/.agents/skills/baidu-drive`
- 已登录：用户名 `殷小韦`
- 使用限制：只能访问 `/apps/bdpan/` 目录
- 资料已迁移到 `/apps/bdpan/phonics-materials/`

### PDF 处理方式

```bash
# 下载
bdpan download "phonics-materials/.../Oxford_Phonics_World_1_SB.pdf" "/tmp/OPW1.pdf"

# 转图片
mkdir -p /tmp/opw1_pages
pdftoppm -png -f 1 -l 8 "/tmp/OPW1.pdf" "/tmp/opw1_pages/page"

# 读取图片用 Read 工具（支持视觉）
```

### 常用 Git 命令

```bash
cd /Users/yinyin/word-challenge
git add .
git commit -m "说明"
git push
```

---

## 6. 注意事项

- 当前会话已经读取了几页牛津 PDF 图片，但尚未大规模提取数据
- 数据提取工作量较大，建议分 Level 分 Unit 逐步完成
- PDF 是扫描版，每个单词/图片都需要人工核对整理
- Sam 课程资料可能包含视频，重点看 `1 Sam老师【超级拼读】` 和 `5单词拆分课【单词速记】`

---

## 7. 推荐接棒路径

1. 先 `git pull` 拉取最新代码
2. 打开 `/Users/yinyin/word-challenge/phonics.html` 看框架
3. 继续下载 Level 2-5 PDF 并提取数据
4. 用脚本或手动把数据整理成 `phonicsData` 格式
5. 替换 `phonics.html` 中的示例数据
6. 测试所有模式
7. commit + push

---

**当前会话暂停于此，等待下次继续。**
