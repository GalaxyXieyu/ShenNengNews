# 智能新闻分析日报系统

## 项目概述

本系统是一个基于AI的新闻数据采集、分析和可视化报告生成平台。通过多源新闻数据采集，结合大语言模型进行智能分析，自动生成包含风险预警和发展机遇评估的图文并茂日报。

### 核心功能
- 🔍 **多源新闻采集**: 支持Bing、Google、博查、Tavily等多个搜索引擎
- 🤖 **AI智能分析**: 基于DeepSeek模型进行情感分析、关键词提取、因子评估
- 📊 **多维可视化**: 6种图表类型全面展示数据洞察
- 📄 **自动报告生成**: HTML/PDF格式日报，支持邮件推送
- ⚠️ **风险预警系统**: 五维度风险评估和实时预警
- 📈 **发展机遇识别**: 智能识别业务发展机会

---

## 系统架构

### 整体架构设计
```
┌─────────────────────────────────────────┐
│           Web管理界面 (可选)              │
├─────────────────────────────────────────┤
│           API接口层                      │
│    /api/v1/news /api/v1/analysis        │
├─────────────────────────────────────────┤
│           业务服务层                      │
│  ┌─────────┬─────────┬─────────┬───────┐  │
│  │搜索服务  │分析服务  │统计服务  │报告服务│  │
│  └─────────┴─────────┴─────────┴───────┘  │
├─────────────────────────────────────────┤
│           数据访问层                      │
│      Repository + Cache + Queue         │
├─────────────────────────────────────────┤
│      数据存储层 (MySQL + Redis)           │
└─────────────────────────────────────────┘
```

### 技术栈
- **后端框架**: FastAPI + SQLAlchemy + Celery
- **数据库**: MySQL + Redis
- **AI模型**: DeepSeek API
- **图表库**: ECharts.js + Matplotlib
- **报告生成**: Jinja2 + WeasyPrint
- **任务调度**: APScheduler + Celery
- **搜索引擎**: Bing API + Google API + 博查API + Tavily API

---

## 功能模块设计

### MVP vs 完整版功能规划

| 功能模块 | MVP版本 | 完整版本 | 扩展版本 |
|---------|---------|----------|----------|
| 新闻搜索 | 单一搜索源 | 多搜索源聚合 | 智能搜索策略 |
| AI分析 | 基础情感分析 | 完整因子分析 | 自定义分析模型 |
| 图表生成 | 简单统计图 | 多维度图表 | 交互式仪表板 |
| 报告输出 | HTML报告 | PDF+邮件推送 | 多格式+定制模板 |
| 任务调度 | 手动触发 | 定时自动化 | 智能调度策略 |

### 1. 数据采集模块 (collector)

#### MVP版本
```
app/services/collector/
├── base.py        # 采集器基类
├── tavily.py      # Tavily搜索采集器
└── manual.py      # 手动录入采集器
```

#### 完整版本
```
app/services/collector/
├── base.py        # 采集器基类
├── tavily.py      # Tavily搜索采集器
├── bing.py        # Bing搜索采集器
├── google.py      # Google搜索采集器
├── bocha.py       # 博查搜索采集器
├── rss.py         # RSS订阅采集器
├── factory.py     # 采集器工厂
├── aggregator.py  # 结果聚合器
└── dedup.py       # 去重处理器
```

#### 核心功能
- **统一接口设计**: 所有采集器继承BaseCollector基类
- **配置驱动**: 通过配置文件管理不同数据源
- **结果标准化**: 统一数据格式和字段映射
- **去重处理**: 基于URL和内容相似度的智能去重
- **异常处理**: 完善的错误处理和重试机制

### 2. AI分析模块 (analyzer)

#### MVP版本
```
app/services/analyzer/
├── base.py        # 分析器基类
├── content.py     # 内容分析器(摘要+情感)
└── keyword.py     # 关键词提取器
```

#### 完整版本
```
app/services/analyzer/
├── base.py        # 分析器基类
├── content.py     # 内容分析器
├── sentiment.py   # 情感分析器
├── keyword.py     # 关键词提取器
├── factor.py      # 因子分析器
├── importance.py  # 重要性评分器
├── risk.py        # 风险评估器
├── category.py    # 分类标注器
└── batch.py       # 批处理器
```

#### 分析维度
```python
# 分析结果数据结构
AnalysisResult = {
    "summary": str,                    # 300字摘要
    "one_line_summary": str,          # 一句话总结
    "keywords": List[str],            # 关键词(5-10个)
    "sentiment": str,                 # positive/negative/neutral
    "sentiment_score": float,         # -1到1的情感分数
    "importance_level": int,          # 1-5星重要程度
    "industry_tags": List[str],       # 行业标签
    "region_tags": List[str],         # 地域标签
    "business_relevance": str,        # high/medium/low/none
    "risk_factors": List[dict],       # 风险因子分析
    "development_factors": List[dict], # 发展因子分析
    "overall_impact": float,          # 综合影响分数
    "risk_level": str                 # A/B/C风险等级
}
```

### 3. 数据管理模块 (data)

#### 数据模型设计 (大宽表)
```python
class NewsWideTable(Base):
    # === 原始数据字段 ===
    id: int
    title: str                    # 标题
    content: Text                 # 正文内容
    source: str                   # 来源媒体
    author: str                   # 作者
    publish_time: datetime        # 发布时间
    url: str                      # 原文链接
    images: JSON                  # 配图URL数组
    
    # === AI分析字段 ===
    summary: Text                 # 300字摘要
    one_line_summary: str         # 一句话总结
    keywords: JSON                # 关键词数组
    sentiment: str                # positive/negative/neutral
    sentiment_score: float        # -1到1的情感分数
    importance_level: int         # 1-5星重要程度
    
    # === 分类标签字段 ===
    industry_tags: JSON           # 行业标签数组
    region_tags: JSON             # 地域标签数组
    business_relevance: str       # high/medium/low/none
    
    # === 风险和发展因子字段 ===
    risk_level: str              # A/B/C风险等级
    risk_factors: JSON           # 风险因子数组
    development_factors: JSON    # 发展因子数组
    impact_score: float          # 业务影响分数
    
    # === 统计辅助字段 ===
    create_time: datetime
    update_time: datetime
    processed: Boolean           # 是否已分析处理
```

#### 仓储模式
```
app/repositories/
├── base.py        # 仓储基类
├── news.py        # 新闻数据访问
├── analysis.py    # 分析数据访问
├── factor.py      # 因子配置访问
├── cache.py       # 缓存数据访问
└── stats.py       # 统计数据访问
```

### 4. 因子分析系统

#### 发展因子配置
```json
{
  "policy_support": {
    "name": "政策支持",
    "weight": 0.25,
    "keywords": ["新能源政策", "补贴", "税收优惠", "碳中和", "双碳目标"],
    "description": "政府政策对业务的支持程度"
  },
  "market_opportunity": {
    "name": "市场机遇", 
    "weight": 0.20,
    "keywords": ["市场扩大", "需求增长", "新兴市场", "合作机会"],
    "description": "市场发展机遇和增长潜力"
  },
  "technology_innovation": {
    "name": "技术创新",
    "weight": 0.20,
    "keywords": ["技术突破", "创新成果", "专利", "研发投入"],
    "description": "技术创新对业务的推动作用"
  },
  "industry_upgrade": {
    "name": "产业升级",
    "weight": 0.15,
    "keywords": ["智能化", "数字化", "绿色转型", "产业链"],
    "description": "产业结构升级和转型机遇"
  },
  "capital_support": {
    "name": "资本支持",
    "weight": 0.20,
    "keywords": ["融资", "投资", "上市", "并购", "资金"],
    "description": "资本市场对业务的支持力度"
  }
}
```

#### 风险预警因子
```json
{
  "policy_risk": {
    "name": "政策风险",
    "weight": 0.25,
    "keywords": ["政策收紧", "监管加强", "合规要求", "处罚"],
    "alert_threshold": 0.6,
    "description": "政策变化对业务的不利影响"
  },
  "market_risk": {
    "name": "市场风险",
    "weight": 0.20,
    "keywords": ["价格下跌", "需求萎缩", "竞争加剧", "市场饱和"],
    "alert_threshold": 0.7,
    "description": "市场环境变化带来的风险"
  },
  "operational_risk": {
    "name": "运营风险",
    "weight": 0.20,
    "keywords": ["安全事故", "环保问题", "质量问题", "供应链"],
    "alert_threshold": 0.8,
    "description": "运营过程中的风险因素"
  },
  "financial_risk": {
    "name": "财务风险",
    "weight": 0.15,
    "keywords": ["债务", "资金链", "成本上升", "盈利下滑"],
    "alert_threshold": 0.7,
    "description": "财务状况恶化风险"
  },
  "reputation_risk": {
    "name": "声誉风险",
    "weight": 0.20,
    "keywords": ["负面报道", "舆情危机", "品牌形象", "客户投诉"],
    "alert_threshold": 0.5,
    "description": "品牌声誉受损风险"
  }
}
```

### 5. 图表生成模块 (chart)

#### 六大核心图表

##### 1. 概览仪表板
```
┌─────────────────────────────────────────┐
│  今日新闻概览 (2024-01-15)                │
│  ├─ 总数：156条                           │
│  ├─ 重要新闻：23条                         │
│  ├─ 高风险：3条 中风险：12条 低风险：141条    │
│  └─ 发展机遇：8条                          │
└─────────────────────────────────────────┘
```

##### 2. 情感趋势图 (折线图)
- **X轴**: 时间（过去7天）
- **Y轴**: 新闻数量
- **数据线**: 积极、消极、中性新闻数量变化

##### 3. 风险预警雷达图
- **5个维度**: 政策风险、市场风险、运营风险、财务风险、声誉风险
- **评分范围**: 0-100分，超过阈值显示预警
- **颜色编码**: 绿色(安全) → 黄色(注意) → 红色(预警)

##### 4. 发展机遇矩阵图 (气泡图)
- **X轴**: 机遇程度（0-100）
- **Y轴**: 实现难度（0-100）  
- **气泡大小**: 影响程度
- **目标区域**: 右下角（高机遇+低难度）

##### 5. 关键词热度词云
- **词汇大小**: 按出现频次
- **颜色深浅**: 按重要程度
- **交互功能**: 点击查看相关新闻

##### 6. 地域分布热力图
- **地图类型**: 中国地图
- **着色规则**: 按新闻关注度
- **交互功能**: 悬停显示详细数据

#### 图表实现技术
```
app/services/chart/
├── base.py        # 图表基类
├── dashboard.py   # 仪表板
├── trend.py       # 趋势图
├── radar.py       # 雷达图
├── matrix.py      # 矩阵图
├── wordcloud.py   # 词云图
├── heatmap.py     # 热力图
└── config.py      # 图表配置
```

### 6. 报告生成模块 (report)

#### 日报结构设计
```html
<!DOCTYPE html>
<html>
<head>
    <title>新闻分析日报 - {{date}}</title>
    <style>/* 响应式样式 */</style>
</head>
<body>
    <!-- 1. 头部概览 -->
    <section class="overview">
        <h1>{{date}} 新闻分析日报</h1>
        <div class="stats-grid">
            <!-- 统计卡片 -->
        </div>
    </section>
    
    <!-- 2. 重点新闻 -->
    <section class="important-news">
        <h2>重点新闻</h2>
        <div class="news-cards">
            <!-- 新闻卡片 -->
        </div>
    </section>
    
    <!-- 3. 数据可视化 -->
    <section class="charts">
        <div class="chart-grid">
            <!-- 6个图表 -->
        </div>
    </section>
    
    <!-- 4. 风险预警 -->
    <section class="risk-alert">
        <h2>风险预警</h2>
        <!-- 高风险新闻列表 -->
    </section>
    
    <!-- 5. 发展机遇 -->
    <section class="opportunities">
        <h2>发展机遇</h2>
        <!-- 机遇新闻列表 -->
    </section>
</body>
</html>
```

#### 报告生成流程
```
app/services/report/
├── base.py        # 报告基类
├── daily.py       # 日报生成器
├── weekly.py      # 周报生成器
├── pdf.py         # PDF导出器
├── email.py       # 邮件发送器
└── template.py    # 模板引擎
```

### 7. 任务调度模块 (scheduler)

#### 调度任务类型
```python
# 调度任务配置
SCHEDULED_TASKS = {
    "daily_collection": {
        "schedule": "0 6 * * *",     # 每天6点执行
        "function": "collect_daily_news",
        "description": "每日新闻采集"
    },
    "hourly_analysis": {
        "schedule": "0 * * * *",      # 每小时执行
        "function": "analyze_pending_news",
        "description": "新闻分析处理"
    },
    "daily_report": {
        "schedule": "0 8 * * *",      # 每天8点执行
        "function": "generate_daily_report",
        "description": "生成日报"
    },
    "risk_monitoring": {
        "schedule": "*/15 * * * *",   # 每15分钟执行
        "function": "monitor_high_risk_news",
        "description": "高风险新闻监控"
    }
}
```

---

## API接口设计

### RESTful API 端点

#### 新闻管理
```
GET    /api/v1/news                    # 获取新闻列表
POST   /api/v1/news/collect            # 手动触发采集
GET    /api/v1/news/{id}               # 获取新闻详情
PUT    /api/v1/news/{id}/analyze       # 手动触发分析
DELETE /api/v1/news/{id}               # 删除新闻
```

#### 分析结果
```
GET    /api/v1/analysis/sentiment      # 情感分析统计
GET    /api/v1/analysis/keywords       # 关键词统计
GET    /api/v1/analysis/factors        # 因子分析结果
GET    /api/v1/analysis/risks          # 风险评估结果
```

#### 报告生成
```
GET    /api/v1/reports/daily/{date}    # 获取日报
POST   /api/v1/reports/generate        # 生成报告
GET    /api/v1/reports/download/{id}   # 下载报告
POST   /api/v1/reports/email           # 邮件发送报告
```

#### 配置管理
```
GET    /api/v1/config/factors          # 获取因子配置
PUT    /api/v1/config/factors          # 更新因子配置
GET    /api/v1/config/sources          # 获取数据源配置
PUT    /api/v1/config/sources          # 更新数据源配置
```

### 请求/响应示例

#### 获取新闻列表
```bash
curl -X GET "/api/v1/news?date=2024-01-15&sentiment=positive&limit=10"
```

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 156,
    "page": 1,
    "limit": 10,
    "items": [
      {
        "id": 1,
        "title": "深圳新能源政策出台",
        "source": "深圳特区报",
        "publish_time": "2024-01-15T08:30:00",
        "sentiment": "positive",
        "importance_level": 4,
        "risk_level": "C",
        "summary": "深圳市政府发布新能源汽车产业扶持政策..."
      }
    ]
  }
}
```

---

## 快速构建策略

### Phase 1: MVP版本 (2-3天)
**目标**: 验证核心功能，快速可用

#### Day 1: 基础框架
- [x] 项目结构搭建
- [x] 数据库设计和迁移
- [x] 基础配置和环境设置
- [ ] 单一数据源采集器 (Tavily)
- [ ] 基础数据模型

#### Day 2: 核心功能
- [ ] AI分析服务 (DeepSeek集成)
- [ ] 基础情感分析和关键词提取
- [ ] 简单统计功能
- [ ] HTML报告模板

#### Day 3: 集成测试
- [ ] 端到端流程测试
- [ ] 基础API接口
- [ ] 简单前端界面
- [ ] 部署和上线

#### MVP功能范围
- ✅ 单一搜索源 (Tavily)
- ✅ 基础AI分析 (摘要、情感、关键词)
- ✅ 3个基础图表 (柱状图、饼图、折线图)
- ✅ HTML日报生成
- ✅ 手动触发流程

### Phase 2: 完整版本 (1-2周)
**目标**: 功能完善，企业可用

#### Week 1: 功能扩展
- [ ] 多数据源集成和聚合
- [ ] 完整因子分析系统
- [ ] 6种图表的完整实现
- [ ] PDF导出和邮件推送
- [ ] 定时任务调度

#### Week 2: 质量提升
- [ ] Web管理界面
- [ ] 完整API文档
- [ ] 单元测试覆盖
- [ ] 性能优化
- [ ] 错误处理和日志

### Phase 3: 高端报告优化 (1-2周)
**目标**: 打造出彩的客户报告，体现专业价值

#### 报告质量提升
- [ ] 震撼开场设计：One-Page Executive Summary
- [ ] 商业洞察深度挖掘：从数据到策略建议
- [ ] 竞争情报专业分析：对手动向与应对策略
- [ ] 商业价值量化：ROI计算与财务影响评估
- [ ] 可执行行动建议：48小时/本周/持续关注清单
- [ ] 个性化定制：基于客户行业和关注点
- [ ] 视觉冲击力设计：信息图表与故事化叙述

---

## 💎 客户报告设计：从普通到出彩的蜕变

### 🎯 报告价值定位转变

#### ❌ 传统报告问题
- 信息堆砌，缺乏洞察深度
- 通用模板，千人一面
- 数据展示，缺少商业价值
- 被动描述，缺乏行动指导

#### ✅ 出彩报告标准
- **专业咨询级洞察**：不只是数据，更是战略建议
- **个性化定制内容**：针对客户行业和痛点
- **量化商业价值**：每个建议都有ROI计算  
- **可执行行动方案**：具体到联系人和截止时间

### 📊 报告结构重构：钻石模型

#### 1. 💎 One-Page Executive Summary（震撼开场）
```
📈 30秒核心发现
├─ 今日最重要的3件事（高影响+紧急程度）
├─ 财务影响预估（能省多少钱/能赚多少钱）
└─ 48小时行动清单（具体可执行）

🎯 个性化价值主张
├─ 致[客户名]CEO：专属定制开场
├─ 紧急程度标识：X条高影响发现需要关注
└─ 预期收益：本报告预计为您创造XX万价值
```

#### 2. 🔍 深度洞察章节（专业分析）
```
📖 故事化数据叙述
├─ 趋势背后的逻辑：不只是"发生了什么"，更是"为什么发生"
├─ 商业影响解读：对您的业务意味着什么
└─ 时间窗口分析：机会持续多久，什么时候行动

💡 洞察置信度体系
├─ 数据支撑程度：基于X条相关报道
├─ 专家判断权重：行业资深分析师观点
└─ 风险提示：不确定性因素说明
```

#### 3. 🎭 竞争情报中心（战略视角）
```
👁️ 对手动向深度解读
├─ 竞争对手在做什么：最新动作梳理
├─ 战略意图分析：为什么这样做
├─ 影响程度评估：对您的威胁等级
└─ 应对策略建议：您应该如何响应

⚡ 差异化机会识别
├─ 市场空白点：对手忽略的机会
├─ 时间窗口：抢先布局的可能性
└─ 资源要求：需要投入的人力物力
```

#### 4. 💰 商业价值量化（ROI导向）
```
📈 投资回报分析
├─ 机遇价值：+XX万元（政策补贴、市场机会等）
├─ 风险成本：-XX万元（合规成本、竞争压力等）
└─ 净影响评估：综合收益XX万元

🎯 场景化ROI计算
├─ 保守场景：最低收益预期
├─ 理想场景：最高收益可能
└─ 推荐场景：平衡风险收益的最佳方案
```

#### 5. 🚀 行动建议引擎（执行导向）
```
⚡ 紧急行动（48小时内）
├─ 具体行动：联系深圳发改委李处长（138-xxxx-1234）
├─ 准备材料：营业执照、可研报告、环评文件
├─ 申请截止：2024年2月15日
└─ 预期收益：补贴资金200-500万元

📅 战略规划（本周内）
├─ 中期布局：技术研发投入规划
├─ 团队建设：关键岗位招聘需求
└─ 合作伙伴：战略联盟机会评估

👁️ 持续监控（长期关注）
├─ 政策跟踪：相关政策发布时间节点
├─ 竞争监控：对手动向预警机制
└─ 市场趋势：行业发展阶段性特征
```

### ✨ 差异化设计元素

#### 1. 情感化语言风格
```
❌ 官方表述："根据数据分析显示..."
✅ 咨询语言："基于我们的深度分析，发现了一个重要机遇..."

❌ 被动描述："政策已发布"
✅ 主动提醒："政府刚刚发布了一项对您极为有利的政策"
```

#### 2. 视觉冲击力设计
```
🎨 信息图表设计
├─ 颜色编码：红色(风险) 绿色(机遇) 橙色(关注)
├─ 图标语言：💰(财务) 📈(趋势) ⚠️(风险) 🚀(机遇)
├─ 进度条：政策申请进度、市场渗透率等
└─ 趋势箭头：↗️上升 ↘️下降 ↔️横盘

📊 数据可视化升级
├─ 不只是数字：用故事串联数据
├─ 对比突出：同比、环比、行业平均
└─ 预测延伸：基于趋势的未来3个月预测
```

#### 3. 互动元素嵌入
```
🔗 延伸阅读链接
├─ 点击查看完整政策文件
├─ 扫码加入专家解读群
├─ 在线预约一对一咨询
└─ 实时数据仪表板访问

📱 移动端优化
├─ 关键信息手机阅读友好
├─ 一键分享到企业微信群
└─ 语音播报重点内容
```

### 🎯 个性化定制策略

#### 1. 行业深度定制
```
🏭 制造业客户
├─ 关注点：产能规划、成本控制、供应链
├─ 语言风格：精准数据、效率导向
└─ 决策周期：季度规划、年度预算

💡 科技企业客户  
├─ 关注点：技术趋势、人才竞争、资本市场
├─ 语言风格：前瞻视角、创新机遇
└─ 决策周期：快速响应、敏捷调整

🏢 服务业客户
├─ 关注点：政策环境、客户需求、品牌形象
├─ 语言风格：市场导向、客户价值
└─ 决策周期：灵活应变、机会导向
```

#### 2. 决策层级适配
```
👨‍💼 CEO关注
├─ 战略影响：对公司整体发展的影响
├─ 财务回报：投资收益和风险评估
└─ 竞争优势：如何保持或获得竞争优势

👨‍💻 CTO关注
├─ 技术趋势：新技术对业务的影响
├─ 研发方向：技术投入的优先级
└─ 人才需求：关键技术人才的获取

💰 CFO关注
├─ 成本影响：政策变化对成本结构的影响
├─ 资金需求：新项目的资金规划
└─ 风险控制：财务风险的预警和控制
```

---

## 使用说明

### 环境初始化

```bash
# 1. 创建虚拟环境 (使用 uv)
uv venv venv --python 3.10
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows

# 2. 安装依赖
uv pip install -r requirements.txt

# 3. 环境配置
cp .env.example .env
# 编辑 .env 文件，配置数据库和API密钥

# 4. 数据库初始化
python -m app.db.init_db

# 5. 启动服务
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 基础使用

```python
# 手动触发新闻采集
from app.services.collector import NewsCollector

collector = NewsCollector()
news_list = collector.collect_news("深圳 新能源", limit=50)

# 分析新闻
from app.services.analyzer import NewsAnalyzer

analyzer = NewsAnalyzer()
for news in news_list:
    analysis = analyzer.analyze(news)
    print(f"情感: {analysis.sentiment}, 风险等级: {analysis.risk_level}")

# 生成日报
from app.services.report import DailyReportGenerator

generator = DailyReportGenerator()
report = generator.generate("2024-01-15")
print(f"日报已生成: {report.file_path}")
```

### API调用示例

```bash
# 触发新闻采集
curl -X POST "http://localhost:8000/api/v1/news/collect" \
  -H "Content-Type: application/json" \
  -d '{"keywords": "深圳 新能源", "limit": 50}'

# 生成日报
curl -X POST "http://localhost:8000/api/v1/reports/generate" \
  -H "Content-Type: application/json" \
  -d '{"date": "2024-01-15", "format": "pdf"}'

# 获取风险预警
curl -X GET "http://localhost:8000/api/v1/analysis/risks?level=A&date=2024-01-15"
```

---

---

## 🚀 业务导向的创新功能

### 当前缺失的核心功能

#### 1. 🎯 智能执行摘要引擎
```python
# 今日最重要的3件事：
executive_summary = {
    "urgent": "深圳出台新能源补贴政策，建议立即申请 [高影响]",
    "threat": "竞争对手A获得5亿融资，需警惕价格战 [中风险]",
    "opportunity": "碳交易市场活跃，考虑提前布局 [机遇]"
}
```

#### 2. ⚡ 智能行动建议引擎
```python
# 基于今日分析，建议您：
action_recommendations = {
    "immediate": "🎯 立即行动：联系政府部门了解补贴申请流程",
    "risk_control": "⚠️ 风险防控：制定应对价格战的策略方案",
    "opportunity": "📈 机遇抓取：调研碳交易市场准入条件",
    "monitoring": "📄 持续关注：监控竞争对手后续动作"
}
```

#### 3. 🎭 竞争情报监控中心
```python
# 竞争对手动态：
competitor_intelligence = {
    "深能A": {
        "action": "新获政府订单2亿元",
        "threat_level": "高",
        "impact": "政府关系竞争加剧"
    },
    "深能B": {
        "action": "技术团队扩招50人",
        "threat_level": "中",
        "impact": "技术研发能力提升"
    },
    "countermeasures": "加强政府关系维护，提升技术研发投入"
}
```

#### 4. 📈 市场情绪指数
```python
# 今日市场情绪: 乐观 ↗️ (+15%)
market_sentiment = {
    "overall": {“值”: 0.15, “趋势”: “乐观”},
    "dimensions": {
        "政策情绪": {"score": 0.25, "status": "积极"},
        "资本情绪": {"score": -0.05, "status": "谨慎"},
        "行业情绪": {"score": 0.20, "status": "乐观"},
        "媒体情绪": {"score": 0.02, "status": "中性"}
    },
    "trend": "持续上升，建议抓住窗口期"
}
```

#### 5. 📊 个性化业务影响评估
```python
# 针对您的新能源业务：
business_impact = {
    "high_impact": {"count": 3, "description": "直接影响业务决策"},
    "medium_impact": {"count": 8, "description": "需要关注跟踪"},
    "low_impact": {"count": 45, "description": "了解即可"},
    "impact_dimensions": {
        "政策影响": {"score": 85, "reason": "新补贴政策出台"},
        "市场影响": {"score": 72, "reason": "需求增长预期"},
        "技术影响": {"score": 45, "reason": "新技术突破"},
        "资金影响": {"score": 68, "reason": "融资环境改善"}
    }
}
```

#### 6. 🚨 舆情危机预警系统
```python
# ⚠️ 舆情风险预警
crisis_alert = {
    "alert_type": "环保问题",
    "severity": "中级",
    "trend": "相关负面报道增加26%",
    "potential_impact": ["政府审批", "公众形象", "股价表现"],
    "recommendation": "准备公关应对方案，主动发声澄清"
}
```

#### 7. ✨ 智能订阅推荐
```python
# 基于您的关注历史，推荐订阅：
subscription_recommendations = [
    "深圳新能源政策每日跟踪",
    "竞争对手动态实时监控",
    "行业技术突破周度摘要",
    "资本市场动向专题分析"
]
```

#### 8. 🤖 AI助手对话模式
```python
# 智能对话交互
ai_chat_examples = [
    {
        "user": "💬 今天有什么重要消息？",
        "ai": "🤖 有3条高影响新闻，最重要的是深圳新能源补贴政策，建议您优先处理申请事宜"
    },
    {
        "user": "💬 这个政策对我们影响有多大？",
        "ai": "🤖 预计可为公司节省成本约500万元，建议立即组织团队准备申请方案"
    }
]
```

#### 9. 📋 协作决策功能
```python
# 团队协作看板
team_collaboration = {
    "roles": {
        "CEO": ["政策机遇", "绞争威胁"],
        "CTO": ["技术趋势", "人才动态"],
        "CFO": ["资本市场", "成本影响"],
        "市场总监": ["客户需求", "行业动向"]
    },
    "features": [
        "内部讨论区：针对重要新闻的团队讨论",
        "决策投票：重要决策的团队投票功能"
    ]
}
```

#### 10. 📱 移动端智能推送
```python
# 移动端支持
mobile_features = {
    "wechat_miniprogram": [
        "重要新闻即时推送",
        "语音播报功能",
        "一键分享到企业群",
        "离线阅读支持"
    ],
    "smart_watch": [
        "高风险新闻震动提醒"
    ]
}
```

---

## 扩展性设计

### 插件化架构
```python
# 插件接口定义
class CollectorPlugin:
    def collect(self, keywords: str, limit: int) -> List[NewsItem]:
        raise NotImplementedError
        
class AnalyzerPlugin:
    def analyze(self, news: NewsItem) -> AnalysisResult:
        raise NotImplementedError

# 插件注册
@register_collector_plugin("custom_source")
class CustomCollector(CollectorPlugin):
    def collect(self, keywords: str, limit: int) -> List[NewsItem]:
        # 自定义采集逻辑
        pass
```

### ### 配置驱动
```yaml
# config/sources.yml
data_sources:
  - name: "tavily"
    enabled: true
    config:
      api_key: "${TAVILY_API_KEY}"
      max_results: 20
  
  - name: "custom_api"
    enabled: false
    config:
      base_url: "https://api.example.com"
      timeout: 30

# config/factors.yml  
risk_factors:
  policy_risk:
    weight: 0.25
    keywords: ["政策收紧", "监管"]
    threshold: 0.6
```

### 多语言支持
```python
# 国际化配置
LANGUAGES = {
    "zh": "中文",
    "en": "English", 
    "ja": "日本語"
}

# 多语言分析器
class MultiLanguageAnalyzer:
    def detect_language(self, text: str) -> str:
        # 语言检测逻辑
        pass
        
    def analyze_by_language(self, text: str, lang: str) -> AnalysisResult:
        # 按语言分析
        pass
```

---

## 项目结构

### 当前目录结构
```
ShenNeng/
├── app/
│   ├── __init__.py
│   ├── main.py                    # FastAPI 应用入口
│   ├── api/                       # API路由
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── news.py           # 新闻相关API
│   │       ├── analysis.py       # 分析相关API
│   │       ├── reports.py        # 报告相关API
│   │       └── config.py         # 配置相关API
│   ├── core/                      # 核心配置
│   │   ├── __init__.py
│   │   ├── config/
│   │   ├── exception/
│   │   └── logging/
│   ├── models/                    # 数据模型
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── news.py               # 新闻数据模型
│   │   ├── analysis.py           # 分析结果模型
│   │   └── report.py             # 报告数据模型
│   ├── services/                  # 业务服务
│   │   ├── __init__.py
│   │   ├── collector/            # 数据采集服务
│   │   │   ├── __init__.py
│   │   │   ├── base_collector.py
│   │   │   ├── tavily_collector.py
│   │   │   ├── bing_collector.py
│   │   │   └── aggregator.py
│   │   ├── analyzer/             # 分析服务
│   │   │   ├── __init__.py
│   │   │   ├── base_analyzer.py
│   │   │   ├── content_analyzer.py
│   │   │   ├── factor_analyzer.py
│   │   │   └── risk_assessor.py
│   │   ├── chart/                # 图表生成服务
│   │   │   ├── __init__.py
│   │   │   ├── base_chart.py
│   │   │   ├── dashboard_chart.py
│   │   │   └── trend_chart.py
│   │   ├── report/               # 报告生成服务
│   │   │   ├── __init__.py
│   │   │   ├── daily_report.py
│   │   │   ├── pdf_exporter.py
│   │   │   └── email_sender.py
│   │   └── scheduler/            # 任务调度服务
│   │       ├── __init__.py
│   │       ├── task_scheduler.py
│   │       └── job_monitor.py
│   ├── repositories/             # 数据访问层
│   │   ├── __init__.py
│   │   ├── base_repository.py
│   │   ├── news_repository.py
│   │   └── analysis_repository.py
│   ├── utils/                    # 工具函数
│   │   ├── __init__.py
│   │   ├── date_utils.py
│   │   ├── text_utils.py
│   │   └── chart_utils.py
│   └── templates/                # 报告模板
│       ├── daily_report.html
│       ├── weekly_report.html
│       └── styles/
├── config/                       # 配置文件
│   ├── factors.yml              # 因子配置
│   ├── sources.yml              # 数据源配置
│   └── charts.yml               # 图表配置
├── tests/                       # 测试文件
│   ├── __init__.py
│   ├── test_collector.py
│   ├── test_analyzer.py
│   └── test_report.py
├── scripts/                     # 脚本文件
│   ├── init_db.py              # 数据库初始化
│   ├── migrate.py              # 数据迁移
│   └── deploy.py               # 部署脚本
├── docs/                       # 文档
│   ├── api.md                  # API文档
│   ├── deployment.md           # 部署文档
│   └── examples/               # 使用示例
├── requirements.txt            # 依赖包
├── .env.example               # 环境变量示例
├── docker-compose.yml         # Docker配置
├── Dockerfile                 # Docker镜像构建
└── README.md                  # 项目说明(本文件)
```

---

## 开发进度

### 开发检查清单

#### Phase 1: MVP版本 ✅
- [x] 项目架构设计
- [x] README文档编写(包含业务功能设计)
- [ ] 环境配置和依赖安装
- [ ] 数据库模型设计
- [ ] 基础数据采集功能
- [ ] AI分析服务集成
- [ ] 简单图表生成
- [ ] HTML报告生成
- [ ] 基础API接口
- [ ] 端到端测试

#### Phase 2: 完整版本 🔄
- [ ] 多数据源集成
- [ ] 完整因子分析系统
- [ ] 6种图表完整实现
- [ ] PDF导出功能
- [ ] 邮件推送功能
- [ ] 定时任务调度
- [ ] 智能执行摘要引擎
- [ ] 行动建议系统
- [ ] 绞争情报监控
- [ ] Web管理界面
- [ ] 完整API文档
- [ ] 单元测试编写

#### Phase 3: 企业级版本 ⏳
- [ ] 用户权限管理
- [ ] 高可用部署
- [ ] 监控告警系统
- [ ] 数据备份恢复
- [ ] AI助手对话模式
- [ ] 协作决策功能
- [ ] 移动端智能推送
- [ ] 舆情危机预警
- [ ] 性能优化
- [ ] 安全加固
- [ ] 多租户支持

---

## 贡献指南

### 开发规范
1. **代码风格**: 遵循 PEP8 规范
2. **类型提示**: 使用 Python 3.10+ 类型注解
3. **文档注释**: 使用 Google 风格的 docstring
4. **测试覆盖**: 单元测试覆盖率 ≥ 90%
5. **提交规范**: 使用 Conventional Commits 格式

### 提交流程
```bash
# 1. 克隆项目
git clone <repository-url>
cd ShenNeng

# 2. 创建功能分支
git checkout -b feature/新功能名称

# 3. 开发和测试
pytest tests/ --cov=app --cov-report=html

# 4. 提交代码
git add .
git commit -m "feat: 添加新闻情感分析功能"

# 5. 推送分支
git push origin feature/新功能名称

# 6. 创建 Pull Request
```

### 代码审查标准
- ✅ 功能完整性验证
- ✅ 代码质量检查
- ✅ 测试覆盖率检查
- ✅ 文档完整性检查
- ✅ 性能影响评估

---

## 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

---

## 联系方式

- **项目负责人**: [项目负责人姓名]
- **技术支持**: [技术支持邮箱]
- **问题反馈**: [GitHub Issues](项目Issues链接)

---

**更新时间**: 2024-01-15  
**版本**: v1.0.0  
**状态**: 开发中 🚧# ShenNengNews
