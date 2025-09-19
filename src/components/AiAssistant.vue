<script setup lang="ts">
import { ref } from 'vue'
import MarkdownIt from 'markdown-it'

// 初始化 MarkdownIt
const md = new MarkdownIt({
  html: true,
  linkify: true,
  typographer: true
})

// 定义消息类型
interface Message {
  id: number
  content: string
  role: 'user' | 'assistant'
  timestamp: Date
  type: 'text' | 'image'
}

// 定义内置指令类型
interface Command {
  id: string
  name: string
  userPrompt: string
  systemPrompt: string
}

// 定义API请求类型
interface ApiMessage {
  role: 'user' | 'assistant' | 'system'
  content: string | Array<{type: string, text?: string, image_url?: {url: string}}>
}

// 响应式数据
const messages = ref<Message[]>([
  {
    id: 1,
    content: '你好！我是你的AI学习助手。我可以帮助你解答问题、评分、翻译、查错和提取难点。',
    role: 'assistant',
    timestamp: new Date(),
    type: 'text'
  }
])

const userInput = ref('')
const selectedImage = ref<File | null>(null)
const imagePreview = ref<string | null>(null)
const isLoading = ref(false)

// API配置
const API_URL = 'https://api.siliconflow.cn/v1/chat/completions'
const API_KEY = import.meta.env.VITE_API_KEY || 'your-api-key-here'

// 内置指令 - 每个指令都有详细的system prompt
const commands: Command[] = [
  {
    id: 'explain',
    name: '解答',
    userPrompt: '',
    systemPrompt: '你是一位专业的教育专家和问题解答师。你的任务是为学生提供详细、准确、易懂的答案和解释。请遵循以下指导原则：\n\n1. 仔细分析问题，确保完全理解用户的需求\n2. 提供结构化的回答，使用清晰的逻辑顺序\n3. 用简单明了的语言解释复杂概念\n4. 适当使用例子和类比来帮助理解\n5. 如果问题涉及多个方面，请分点详细说明\n6. 在必要时提供相关的背景知识\n7. 鼓励学生进一步思考和探索\n\n请以耐心、专业和友好的态度提供帮助。'
  },
  {
    id: 'grade',
    name: '评分',
    userPrompt: '',
    systemPrompt: '你是一位经验丰富的教育评估专家。你的任务是对学生的作品进行专业评分和提供建设性反馈。请遵循以下评估标准：\n\n评分标准（满分100分）：\n- 内容准确性（30分）：信息的正确性和完整性\n- 逻辑结构（20分）：条理清晰，论证合理\n- 创新思维（15分）：独特见解和创造性思考\n- 表达质量（20分）：语言流畅，表达清晰\n- 实用价值（15分）：对学习和实践的指导意义\n\n反馈要求：\n1. 指出具体优点和不足\n2. 提供改进建议和学习方向\n3. 用鼓励性语言激发学习动力\n4. 避免过于严厉的批评\n\n请以专业、公正、鼓励的态度进行评估。'
  },
  {
    id: 'translate',
    name: '翻译',
    userPrompt: '',
    systemPrompt: '你是一位专业的多语言翻译专家，精通中英文互译。你的任务是提供准确、自然、符合语言习惯的翻译服务。请遵循以下翻译原则：\n\n翻译标准：\n1. 忠实原文：准确传达原意，不遗漏不添加\n2. 语言自然：符合目标语言表达习惯\n3. 文化适应：考虑文化差异，适当调整表达\n4. 专业准确：专业术语使用准确\n5. 风格保持：保持原文的语调和风格\n\n翻译要求：\n1. 对于复杂句子，可适当调整语序以符合目标语言习惯\n2. 对于文化特定概念，可添加简要说明\n3. 保持术语一致性\n4. 注意语境，选择最合适的词汇\n\n请提供高质量、地道的翻译结果。'
  },
  {
    id: 'debug',
    name: '查错',
    userPrompt: '',
    systemPrompt: '你是一位专业的错误检测和纠正专家。你的任务是仔细检查内容中的各种错误并提供纠正建议。请按照以下分类进行检查：\n\n检查范围：\n1. 语法错误：句子结构、词法错误\n2. 拼写错误：错别字、拼写问题\n3. 标点符号：使用不当或缺失\n4. 逻辑错误：推理不严密、前后矛盾\n5. 事实错误：不准确的信息或数据\n6. 表达问题：语言不流畅、表意不清\n\n纠错要求：\n1. 明确指出错误位置和类型\n2. 提供正确的修改建议\n3. 解释错误原因\n4. 给出避免类似错误的建议\n\n请以细致、专业、耐心的态度进行纠错。'
  },
  {
    id: 'extract',
    name: '难点提取',
    userPrompt: '',
    systemPrompt: '你是一位专业的学习指导专家，擅长提取和总结学习内容的重点和难点。你的任务是帮助学生高效掌握学习材料。请按照以下方法进行分析：\n\n分析框架：\n1. 核心概念识别：找出关键术语和重要概念\n2. 知识结构梳理：理清内容的逻辑关系\n3. 重点内容标记：标出必须掌握的部分\n4. 难点分析：指出理解困难的部分并分析原因\n5. 学习建议：提供针对性的学习策略\n\n输出格式：\n- 核心概念：[列出3-5个关键概念]\n- 知识结构：[简述内容的逻辑关系]\n- 学习重点：[列出2-3个必须掌握的要点]\n- 主要难点：[列出1-2个难点及解决建议]\n- 学习建议：[提供具体的学习方法和步骤]\n\n请以清晰、结构化的方式提供分析结果。'
  }
]

// 添加渲染 Markdown 的函数
const renderMarkdown = (content: string) => {
  return md.render(content)
}

// 调用AI API
const callAIApi = async (messagesToSend: ApiMessage[]) => {
  try {
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: 'THUDM/GLM-4.1V-9B-Thinking',
        messages: messagesToSend,
        stream: false
      })
    })
    
    if (!response.ok) {
      throw new Error(`API request failed with status ${response.status}`)
    }
    
    const data = await response.json()
    return data.choices[0].message.content
  } catch (error) {
    console.error('API调用失败:', error)
    throw new Error('AI助手暂时无法响应，请稍后再试。')
  }
}

// 发送文本消息
const sendTextMessage = async () => {
  if (userInput.value.trim() === '' || isLoading.value) return
  
  // 添加用户消息
  const userMessage: Message = {
    id: Date.now(),
    content: userInput.value,
    role: 'user',
    timestamp: new Date(),
    type: 'text'
  }
  
  messages.value.push(userMessage)
  
  // 设置加载状态
  isLoading.value = true
  
  // 清空输入框
  userInput.value = ''
  
  try {
    // 准备发送给API的消息（普通对话不需要特殊的system prompt）
    const apiMessages: ApiMessage[] = [
      {
        role: 'system',
        content: '你是一位专业的AI学习助手，能够帮助学生解答问题、提供学习建议、进行内容分析等。请以耐心、专业、友好的态度提供帮助。'
      },
      ...messages.value.map(msg => ({
        role: msg.role as 'user' | 'assistant',
        content: msg.content
      }))
    ]
    
    // 调用AI API
    const aiResponse = await callAIApi(apiMessages)
    
    // 添加AI回复
    const aiMessage: Message = {
      id: Date.now() + 1,
      content: aiResponse,
      role: 'assistant',
      timestamp: new Date(),
      type: 'text'
    }
    
    messages.value.push(aiMessage)
  } catch (error: any) {
    // 错误处理
    const errorMessage: Message = {
      id: Date.now() + 1,
      content: error.message || '发生未知错误',
      role: 'assistant',
      timestamp: new Date(),
      type: 'text'
    }
    
    messages.value.push(errorMessage)
  } finally {
    // 取消加载状态
    isLoading.value = false
    // 滚动到最新消息
    scrollToBottom()
  }
}

// 发送图片消息
const sendImageMessage = async () => {
  if (!selectedImage.value || isLoading.value) return
  
  // 添加用户图片消息
  const userMessage: Message = {
    id: Date.now(),
    content: imagePreview.value || '',
    role: 'user',
    timestamp: new Date(),
    type: 'image'
  }
  
  messages.value.push(userMessage)
  
  // 设置加载状态
  isLoading.value = true
  
  // 清空图片选择
  selectedImage.value = null
  imagePreview.value = null
  
  try {
    // 准备发送给API的消息（包含图片）
    const apiMessages: ApiMessage[] = [
      {
        role: 'system',
        content: '你是一位专业的AI学习助手，能够分析图片内容并提供相关帮助。'
      },
      ...messages.value.slice(0, -1).map(msg => ({
        role: msg.role as 'user' | 'assistant',
        content: msg.content
      })),
      {
        role: 'user',
        content: [
          {
            type: 'text',
            text: '请分析这张图片的内容：'
          },
          {
            type: 'image_url',
            image_url: {
              url: userMessage.content
            }
          }
        ]
      }
    ]
    
    // 调用AI API
    const aiResponse = await callAIApi(apiMessages)
    
    // 添加AI回复
    const aiMessage: Message = {
      id: Date.now() + 1,
      content: aiResponse,
      role: 'assistant',
      timestamp: new Date(),
      type: 'text'
    }
    
    messages.value.push(aiMessage)
  } catch (error: any) {
    // 错误处理
    const errorMessage: Message = {
      id: Date.now() + 1,
      content: error.message || '发生未知错误',
      role: 'assistant',
      timestamp: new Date(),
      type: 'text'
    }
    
    messages.value.push(errorMessage)
  } finally {
    // 取消加载状态
    isLoading.value = false
    // 滚动到最新消息
    scrollToBottom()
  }
}

// 滚动到底部
const scrollToBottom = () => {
  const messagesContainer = document.querySelector('.messages-fluent')
  if (messagesContainer) {
    messagesContainer.scrollTop = messagesContainer.scrollHeight
  }
}

// 处理图片选择
const handleImageSelect = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  
  if (file) {
    selectedImage.value = file
    const reader = new FileReader()
    reader.onload = (e) => {
      imagePreview.value = e.target?.result as string
    }
    reader.readAsDataURL(file)
  }
}

// 使用内置指令 - 修复逻辑：用户输入内容后点击按钮应触发携带对应prompt的对话
const useCommand = async (command: Command) => {
  // 检查用户是否输入了内容
  if (userInput.value.trim() === '') {
    // 如果没有输入内容，提示用户输入
    alert('请先输入内容，然后点击指令按钮')
    return
  }
  
  // 构造用户消息
  const userMessageContent = userInput.value
  
  // 添加用户消息
  const userMessage: Message = {
    id: Date.now(),
    content: userMessageContent,
    role: 'user',
    timestamp: new Date(),
    type: 'text'
  }
  
  messages.value.push(userMessage)
  
  // 设置加载状态
  isLoading.value = true
  
  // 清空输入框
  userInput.value = ''
  
  try {
    // 准备发送给API的消息（包含特定的system prompt）
    const apiMessages: ApiMessage[] = [
      {
        role: 'system',
        content: command.systemPrompt
      },
      ...messages.value.map(msg => ({
        role: msg.role as 'user' | 'assistant',
        content: msg.content
      }))
    ]
    
    // 调用AI API
    const aiResponse = await callAIApi(apiMessages)
    
    // 添加AI回复
    const aiMessage: Message = {
      id: Date.now() + 1,
      content: aiResponse,
      role: 'assistant',
      timestamp: new Date(),
      type: 'text'
    }
    
    messages.value.push(aiMessage)
  } catch (error: any) {
    // 错误处理
    const errorMessage: Message = {
      id: Date.now() + 1,
      content: error.message || '发生未知错误',
      role: 'assistant',
      timestamp: new Date(),
      type: 'text'
    }
    
    messages.value.push(errorMessage)
  } finally {
    // 取消加载状态
    isLoading.value = false
    // 滚动到最新消息
    scrollToBottom()
  }
}

// 组件挂载后滚动到底部
import { onMounted } from 'vue'
onMounted(() => {
  scrollToBottom()
})
</script>

<template>
  <div class="ai-assistant-fluent">
    <div class="header-fluent">
      <h1 class="header-title">DoHomework - AI 学习助手</h1>
      <p class="header-subtitle">您的智能学习伙伴</p>
    </div>
    
    <div class="chat-container-fluent">
      <div class="messages-fluent">
        <div 
          v-for="message in messages" 
          :key="message.id"
          :class="['message-fluent', message.role]"
        >
          <div class="message-content-fluent">
            <template v-if="message.type === 'text'">
              <div v-html="renderMarkdown(message.content)"></div>
            </template>
            <template v-else>
              <img :src="message.content" alt="用户上传的图片" class="message-image-fluent" />
            </template>
          </div>
          <div class="timestamp-fluent">
            {{ message.timestamp.toLocaleTimeString() }}
          </div>
        </div>
        
        <!-- 加载指示器 -->
        <div v-if="isLoading" class="message-fluent assistant">
          <div class="message-content-fluent">
            <div class="loading-indicator-fluent">
              <div class="loading-spinner-fluent"></div>
              <p>AI助手正在思考中...</p>
            </div>
          </div>
        </div>
      </div>
      
      <div class="commands-fluent">
        <button 
          v-for="command in commands" 
          :key="command.id"
          @click="useCommand(command)"
          class="command-button-fluent"
          :disabled="isLoading"
        >
          {{ command.name }}
        </button>
      </div>
      
      <div class="input-area-fluent">
        <div class="text-input-fluent">
          <textarea 
            v-model="userInput" 
            placeholder="输入你的问题... (按Enter发送，Shift+Enter换行)"
            @keydown.enter.exact.prevent="sendTextMessage"
            @keydown.enter.shift.exact.prevent="userInput += '\n'"
            class="input-field-fluent"
          ></textarea>
          <button 
            @click="sendTextMessage" 
            class="send-button-fluent"
            :disabled="isLoading || userInput.trim() === ''"
          >
            发送
          </button>
        </div>
        
        <div class="image-input-fluent">
          <input 
            type="file" 
            accept="image/*" 
            @change="handleImageSelect" 
            id="image-upload-fluent"
            hidden
          />
          <label for="image-upload-fluent" class="image-upload-label-fluent">
            选择图片
          </label>
          <button 
            @click="sendImageMessage" 
            :disabled="!selectedImage || isLoading"
            class="send-image-button-fluent"
          >
            发送图片
          </button>
          <div v-if="imagePreview" class="image-preview-fluent">
            <img :src="imagePreview" alt="预览图片" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ai-assistant-fluent {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
}

.header-fluent {
  padding: 24px 0;
  text-align: center;
  margin-bottom: 24px;
  border-bottom: 1px solid #f3f2f1;
  background-color: #ffffff;
}

.header-title {
  margin: 0 0 8px 0;
  color: #201f1e;
  font-size: 28px;
  font-weight: 600;
  line-height: 36px;
}

.header-subtitle {
  margin: 0;
  color: #605e5c;
  font-size: 16px;
  font-weight: 400;
  line-height: 22px;
}

.chat-container-fluent {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: #ffffff;
  border-radius: 0;
  box-shadow: none;
  border: none;
}

.messages-fluent {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
  margin-bottom: 24px;
  background-color: #faf9f8;
  border-radius: 0;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.message-fluent {
  max-width: 85%;
  padding: 16px;
  border-radius: 8px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  animation: fadeInFluent 0.2s ease;
  position: relative;
}

@keyframes fadeInFluent {
  from { opacity: 0; transform: translateY(4px); }
  to { opacity: 1; transform: translateY(0); }
}

.message-fluent.user {
  background-color: #0078d4;
  color: white;
  align-self: flex-end;
  border-bottom-right-radius: 2px;
}

.message-fluent.assistant {
  background-color: #ffffff;
  color: #201f1e;
  border: 1px solid #f3f2f1;
  align-self: flex-start;
  border-bottom-left-radius: 2px;
}

.message-content-fluent {
  margin-bottom: 8px;
  line-height: 1.5;
  font-size: 15px;
  font-weight: 400;
}

.message-content-fluent :deep(p) {
  margin: 0 0 10px 0;
}

.message-content-fluent :deep(h1),
.message-content-fluent :deep(h2),
.message-content-fluent :deep(h3) {
  margin: 15px 0 10px 0;
  font-weight: 600;
}

.message-content-fluent :deep(h1) {
  font-size: 24px;
}

.message-content-fluent :deep(h2) {
  font-size: 20px;
}

.message-content-fluent :deep(h3) {
  font-size: 18px;
}

.message-content-fluent :deep(ul),
.message-content-fluent :deep(ol) {
  margin: 10px 0;
  padding-left: 20px;
}

.message-content-fluent :deep(li) {
  margin: 5px 0;
}

.message-content-fluent :deep(pre) {
  background-color: #f5f5f5;
  padding: 12px;
  border-radius: 4px;
  overflow-x: auto;
  margin: 10px 0;
}

.message-content-fluent :deep(code) {
  background-color: #f5f5f5;
  padding: 2px 4px;
  border-radius: 2px;
  font-family: 'Consolas', 'Courier New', monospace;
}

.message-content-fluent :deep(blockquote) {
  border-left: 3px solid #0078d4;
  padding-left: 12px;
  margin: 10px 0;
  color: #605e5c;
}

.message-content-fluent :deep(a) {
  color: #0078d4;
  text-decoration: none;
}

.message-content-fluent :deep(a:hover) {
  text-decoration: underline;
}

.message-content-fluent :deep(table) {
  border-collapse: collapse;
  width: 100%;
  margin: 10px 0;
}

.message-content-fluent :deep(th),
.message-content-fluent :deep(td) {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.message-content-fluent :deep(th) {
  background-color: #f5f5f5;
  font-weight: 600;
}

.message-image-fluent {
  max-width: 100%;
  max-height: 300px;
  border-radius: 4px;
}

.timestamp-fluent {
  font-size: 12px;
  color: #a19f9d;
  text-align: right;
  margin-top: 8px;
}

.commands-fluent {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-bottom: 24px;
  flex-wrap: wrap;
  padding: 0 12px;
}

.command-button-fluent {
  padding: 10px 20px;
  background-color: #ffffff;
  color: #201f1e;
  border: 1px solid #8a8886;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.1s ease;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.command-button-fluent:hover:not(:disabled) {
  background-color: #f3f2f1;
  border-color: #605e5c;
}

.command-button-fluent:active:not(:disabled) {
  background-color: #edebe9;
  border-color: #323130;
}

.command-button-fluent:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.input-area-fluent {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 24px;
  background: #faf9f8;
  border-radius: 0;
  border-top: 1px solid #f3f2f1;
}

.text-input-fluent {
  display: flex;
  gap: 12px;
}

.input-field-fluent {
  flex: 1;
  padding: 12px 16px;
  border: 1px solid #8a8886;
  border-radius: 4px;
  resize: none;
  height: 100px;
  font-size: 15px;
  font-family: inherit;
  background: #ffffff;
  transition: border-color 0.1s ease;
  color: #201f1e;
}

.input-field-fluent:focus {
  outline: none;
  border-color: #0078d4;
  box-shadow: 0 0 0 2px rgba(0, 120, 212, 0.2);
}

.send-button-fluent {
  padding: 12px 24px;
  background-color: #0078d4;
  color: white;
  border: 1px solid #0078d4;
  border-radius: 4px;
  cursor: pointer;
  font-size: 15px;
  font-weight: 600;
  transition: all 0.1s ease;
  align-self: flex-start;
}

.send-button-fluent:hover:not(:disabled) {
  background-color: #106ebe;
  border-color: #106ebe;
}

.send-button-fluent:active:not(:disabled) {
  background-color: #005a9e;
  border-color: #005a9e;
}

.send-button-fluent:disabled {
  background-color: #f3f2f1;
  border-color: #f3f2f1;
  color: #a19f9d;
  cursor: not-allowed;
}

.image-input-fluent {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.image-upload-label-fluent {
  padding: 10px 20px;
  background-color: #ffffff;
  color: #201f1e;
  border: 1px solid #8a8886;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.1s ease;
}

.image-upload-label-fluent:hover {
  background-color: #f3f2f1;
  border-color: #605e5c;
}

.send-image-button-fluent {
  padding: 10px 20px;
  background-color: #0078d4;
  color: white;
  border: 1px solid #0078d4;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.1s ease;
}

.send-image-button-fluent:hover:not(:disabled) {
  background-color: #106ebe;
  border-color: #106ebe;
}

.send-image-button-fluent:disabled {
  background-color: #f3f2f1;
  border-color: #f3f2f1;
  color: #a19f9d;
  cursor: not-allowed;
}

.image-preview-fluent {
  max-width: 150px;
  margin-top: 12px;
}

.image-preview-fluent img {
  max-width: 100%;
  max-height: 150px;
  border-radius: 4px;
  border: 1px solid #f3f2f1;
}

.loading-indicator-fluent {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16px;
}

.loading-spinner-fluent {
  width: 24px;
  height: 24px;
  border: 2px solid #f3f2f1;
  border-top: 2px solid #0078d4;
  border-radius: 50%;
  animation: spinFluent 1s linear infinite;
  margin-bottom: 12px;
}

@keyframes spinFluent {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@media (max-width: 768px) {
  .ai-assistant-fluent {
    padding: 0;
  }
  
  .header-title {
    font-size: 24px;
  }
  
  .message-fluent {
    max-width: 90%;
  }
  
  .text-input-fluent {
    flex-direction: column;
  }
  
  .input-field-fluent {
    height: 80px;
  }
  
  .send-button-fluent {
    padding: 12px;
  }
  
  .commands-fluent {
    flex-direction: column;
    align-items: center;
  }
  
  .command-button-fluent {
    width: 100%;
    max-width: 300px;
  }
}
</style>