# 简易Cloudflare大模型API反代工具

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/dogchild/api-cf)


## 项目介绍
本项目提供了一个在Cloudflare Workers或Pages上部署的简易解决方案，用于反代中国大陆无法直接访问的大模型API服务。通过本工具，您可以方便地搭建自己的API网关，实现对多个主流大模型服务的访问，同时支持**可选的**API密钥轮询和请求日志记录等增强功能。

## 项目特点
- **基础功能**：
  - **完全透传**：不修改原始API请求和响应，确保与官方API完全兼容
  - **多服务支持**：支持Gemini、OpenAI、Claude、Groq、Cerebras等主流大模型服务
- **可选功能**：
  - **API密钥轮询**：**可选**的轮询功能，自动轮询多个API密钥，有效避免单个密钥的请求限制问题
  - **请求日志记录**：**可选**的日志记录功能，提供请求可观测性和性能分析
- **简单部署**：基于Cloudflare Workers平台，部署简单，无需维护服务器

## 支持的服务提供商
暂时支持gemini, openai, claude, groq, cerebras大模型服务。

## 如何使用
**基本使用方法：**
- **GEMINI:**   `https://<your_url>/gemini/...`
- **OPENAI:**   `https://<your_url>/openai/...`
- **CLAUDE:**   `https://<your_url>/claude/...`
- **GROQ:**     `https://<your_url>/groq/...`
- **CEREBRAS:**  `https://<your_url>/cerebras/...`

## 可选功能

### 1. API密钥轮询功能
该功能可实现API密钥的自动轮询，有效避免单个密钥的请求限制问题。

**配置方法：**
- 参考详细配置指南：[轮询配置指南](ROTATION_SETUP.md)
- 在Cloudflare仪表板创建D1数据库
- 在仪表板或wrangler.toml中配置DB绑定
- 创建rotation_state表用于存储轮询状态
- 设置MASTER_KEY环境变量和各服务商的密钥列表

### 2. Analytics Engine 日志功能
该功能可记录所有API请求的关键指标，提供强大的可观测性。

**配置方法：**
- 参考详细配置指南：[Analytics Engine 日志配置指南](ANALYTICS_ENGINE_SETUP.md)
- 在Cloudflare仪表板创建Analytics Engine数据集
- 在仪表板或wrangler.toml中配置LOGS绑定
- 日志会自动记录服务商、模型、状态码、响应时间等信息

## 注意事项
- **基础反代功能**：无需额外配置即可使用，直接部署即可使用。
- **轮询功能**：需要配置D1数据库、MASTER_KEY和各服务商的密钥列表。
- **日志功能**：需要配置Analytics Engine数据集和LOGS绑定。
- **所有功能均为可选**：您可以根据需要选择性配置。






