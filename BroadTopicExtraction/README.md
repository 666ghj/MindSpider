# BroadTopicExtraction 模块

## 概述

BroadTopicExtraction是MindSpider AI爬虫项目的第一个核心模块，负责从多平台热点新闻中提取话题关键词并生成分析总结。

## 设计理念

采用**简化设计**理念，避免过度复杂的架构：
- 不使用复杂的聚类算法
- 不需要embedding向量计算  
- 直接使用DeepSeek对话模型进行话题分析
- 数据库结构简单明了

## 核心功能

### 1. 新闻收集
- 支持多个热门平台：微博、知乎、B站、头条、抖音等
- 异步并发获取，提高效率
- 自动去重和数据清理
- 存储到MySQL数据库

### 2. 话题提取  
- 基于DeepSeek AI模型
- 提取热点关键词（适合社交媒体搜索）
- 生成新闻分析总结
- 支持自定义关键词数量

### 3. 数据存储
- 简化的MySQL表结构
- 只保留必要字段：日期、关键词、总结
- 支持历史数据查询和统计

## 模块结构

```
BroadTopicExtraction/
├── main.py                         # 主程序和工作流程
├── get_today_news.py              # 新闻获取和收集
├── topic_extractor.py             # 话题提取器  
├── database_manager.py            # 数据库管理器
└── README.md                       # 模块文档
```

## 数据库表结构

### daily_news 表
| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 自增ID |
| news_id | varchar(128) | 新闻唯一ID |
| source_platform | varchar(32) | 新闻源平台 |
| title | varchar(500) | 新闻标题 |
| url | varchar(512) | 新闻链接 |
| crawl_date | date | 爬取日期 |
| rank_position | int | 排名位置 |
| add_ts | bigint | 添加时间戳 |

### daily_topics 表
| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 自增ID |
| extract_date | date | 提取日期 |
| keywords | text | 关键词列表(JSON) |
| summary | text | 新闻分析总结 |
| add_ts | bigint | 添加时间戳 |

## 使用方法

### 基本使用

```python
from BroadTopicExtraction.main import BroadTopicExtraction

async def main():
    async with BroadTopicExtraction() as extractor:
        # 运行每日话题提取（默认使用所有平台）
        result = await extractor.run_daily_extraction()
        
        # 打印结果
        extractor.print_extraction_results(result)
        
        # 获取用于爬取的关键词
        keywords = extractor.get_keywords_for_crawling()
        print(f"爬取关键词: {keywords}")
```

### 命令行使用

```bash
# 使用所有平台，提取100个关键词
python BroadTopicExtraction/main.py

# 指定平台
python BroadTopicExtraction/main.py --sources weibo zhihu

# 指定关键词数量
python BroadTopicExtraction/main.py --keywords 150

# 简化输出
python BroadTopicExtraction/main.py --quiet

# 查看支持的平台
python BroadTopicExtraction/main.py --list-sources
```

### 高级使用

```python
# 1. 只收集新闻
from BroadTopicExtraction.get_today_news import NewsCollector

async with NewsCollector() as collector:
    result = await collector.collect_and_save_news()  # 默认所有平台

# 2. 只提取话题
from BroadTopicExtraction.topic_extractor import TopicExtractor

extractor = TopicExtractor()
keywords, summary = extractor.extract_keywords_and_summary(news_list)

# 3. 数据库操作
from BroadTopicExtraction.database_manager import DatabaseManager

with DatabaseManager() as db:
    # 保存话题分析
    db.save_daily_topics(keywords, summary)
    
    # 获取今日分析
    analysis = db.get_daily_topics()
```

## 工作流程

```
1. 新闻收集
   ↓
2. DeepSeek话题分析
   ↓  
3. 提取关键词+生成总结
   ↓
4. 保存到数据库
   ↓
5. 为DeepSentimentCrawling提供关键词
```

## 配置要求

### 环境依赖
- Python 3.11+ (conda环境: pytorch_python11)
- MySQL数据库
- DeepSeek API密钥

### 必要配置 (config.py)
```python
# MySQL数据库配置
DB_HOST = "your_host"
DB_PORT = 3306
DB_USER = "your_user"  
DB_PASSWORD = "your_password"
DB_NAME = "mindspider"
DB_CHARSET = "utf8mb4"

# DeepSeek API密钥
DEEPSEEK_API_KEY = "sk-xxxxxx"
```

## 测试

运行完整测试：
```bash
python debug_tests/test_broad_topic_extraction.py
```

运行生产环境：
```bash
python BroadTopicExtraction/main.py
```

## 输出示例

### 关键词示例
```
["人工智能", "明星绯闻", "游戏发布", "社会热点", "科技创新"]
```

### 总结示例
```
今日热点呈现多元化特征，科技领域AI技术发展引发广泛关注，
娱乐圈明星动态保持较高热度，游戏产业新作发布获得用户期待，
同时社会民生话题也持续引发公众讨论，体现出舆论场的丰富性。
```

## 特点优势

✅ **简单易用**：避免复杂的机器学习算法  
✅ **高效稳定**：异步处理，错误容忍  
✅ **扩展性好**：易于添加新的新闻源  
✅ **资源友好**：只需要对话API，无需embedding  
✅ **实时性强**：快速获取和分析当日热点  

## 下一步

该模块的输出（关键词列表）将作为DeepSentimentCrawling模块的输入，用于在各社交媒体平台进行针对性的内容爬取和情感分析。
