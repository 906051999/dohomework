# DoHomework - AI 学习助手

基础的学习助手

## 功能特性

1. 支持文本对话
2. 支持图片对话
3. 支持5个内置指令（解答、评分、翻译、查错、难点提取），每个内置指令预设对应详细的system prompt，用户输入内容后点击对应指令按钮，即可携带对应的prompt进行对话。

## 设计特点

- 采用Fluent UI设计语言，界面优雅简洁大方
- 响应式设计，适配各种设备屏幕
- 流畅的动画效果和用户体验

## 技术栈

- Vue 3 + TypeScript + Vite
- Vercel 部署

## 使用的AI模型

- THUDM/GLM-4.1V-9B-Thinking
- deepseek-ai/DeepSeek-R1-0528-Qwen3-8B
- Qwen/Qwen3-8B

## API 配置

使用 OpenAI 格式 API:
```
https://api.siliconflow.cn/v1/chat/completions
```

## 环境变量配置

1. 复制 `.env.example` 文件并重命名为 `.env`
2. 在 `.env` 文件中填写你的API密钥：
```
VITE_API_KEY=your_actual_api_key_here
VITE_GEMINI_API_KEY=your_gemini_api_key_here
```

### API密钥获取

- `VITE_API_KEY`：从 [SiliconFlow](https://siliconflow.cn) 获取，用于GLM模型
- `VITE_GEMINI_API_KEY`：从 [Google AI Studio](https://aistudio.google.com/) 获取，用于Gemini模型

## 使用说明

1. 在输入框中输入您的问题或内容
2. 点击5个内置指令按钮之一来触发特定类型的AI分析：
   - 解答：详细解答问题（使用专业的教育专家system prompt）
   - 评分：对内容进行评分和建议（使用教育评估专家system prompt）
   - 翻译：将内容翻译成中文（使用专业翻译专家system prompt）
   - 查错：检查内容中的错误（使用错误检测专家system prompt）
   - 难点提取：提取重点和难点（使用学习指导专家system prompt）
3. 也可以直接点击"发送"按钮进行普通对话
4. 支持图片上传分析功能

## 本地开发

```bash
npm install
npm run dev
```

## 部署

使用 Vercel 进行部署。