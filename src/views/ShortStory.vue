<template>
  <div class="short-story">
    
    <!-- 左右分栏布局 -->
    <div class="story-workspace">
      <!-- 左侧配置面板 -->
      <div class="config-panel">
                <div class="panel-header">
          <div class="header-title-row">
            <h3>创作配置</h3>
          </div>
          <div class="header-actions-row">
            <div class="secondary-actions">
              <el-button 
                size="small"
                text
                type="warning"
                @click="showConfigManager = true"
                title="管理配置选项"
              >
                <el-icon><Setting /></el-icon>
                配置管理
              </el-button>
              <el-button 
                size="small"
                text
                type="info"
                @click="resetConfig"
                title="重置所有配置"
              >
                <el-icon><Refresh /></el-icon>
                重置
              </el-button>
            </div>
            <div class="primary-action">
              <el-button 
                type="primary" 
                size="small" 
                @click="generateStory" 
                :loading="generating"
                :disabled="!isConfigValid"
                style="width: 100%;"
              >
                <el-icon><MagicStick /></el-icon>
                {{ generating ? '生成中...' : '生成小说' }}
              </el-button>
            </div>
          </div>
          <!-- 必填项提示 -->
          <div v-if="!isConfigValid" class="required-items-tip">
            <span class="tip-label">缺少必填项：</span>
            <span class="missing-items">
              <span v-if="!storyData.title" class="missing-item">小说标题</span>
              <span v-if="!storyData.protagonist.name" class="missing-item">主角姓名</span>
              <span v-if="!unifiedPrompt.trim()" class="missing-item">创作提示词</span>
            </span>
          </div>
        </div>
        
        <div class="config-scroll-container">
          <el-scrollbar height="100%">
            <div class="config-content">
              <!-- 快速配置区 -->
              <div class="quick-config">
                <!-- 基础选择 -->
                <div class="basic-selects">
                  <div class="select-row">
                    <div class="select-item">
                      <label>题材</label>
                      <el-select v-model="storyData.genre" placeholder="选择题材" size="small">
                        <el-option v-for="genre in customGenres" :key="genre.value" :label="genre.label" :value="genre.value" />
                      </el-select>
                    </div>
                    <div class="select-item">
                      <label>情节</label>
                      <el-select v-model="storyData.plotType" placeholder="选择情节类型" size="small">
                        <el-option v-for="plot in customPlotTypes" :key="plot.value" :label="plot.label" :value="plot.value" />
                      </el-select>
                    </div>
                  </div>
                  
                  <div class="select-row">
                    <div class="select-item">
                      <label>氛围</label>
                      <el-select v-model="storyData.emotion" placeholder="选择情绪氛围" size="small">
                        <el-option v-for="emotion in customEmotions" :key="emotion.value" :label="emotion.label" :value="emotion.value" />
                      </el-select>
                    </div>
                    <div class="select-item">
                      <label>时代</label>
                      <el-select v-model="storyData.timeFrame" placeholder="选择时间背景" size="small">
                        <el-option v-for="timeFrame in customTimeFrames" :key="timeFrame.value" :label="timeFrame.label" :value="timeFrame.value" />
                      </el-select>
                    </div>
                  </div>
                  
                  <div class="select-row">
                    <div class="select-item">
                      <label>字数</label>
                      <el-input-number 
                        v-model="storyData.wordCount" 
                        :min="500" 
                        :max="10000" 
                        :step="100"
                        size="small"
                        placeholder="目标字数"
                        style="width: 100%"
                      />
                    </div>
                    <div class="select-item">
                      <!-- 占位，保持布局平衡 -->
                    </div>
                  </div>
                </div>

                <!-- 快速输入区 -->
                <div class="quick-inputs">
                  <div class="input-row">
                    <el-input v-model="storyData.title" placeholder="小说标题" size="small" />
                    <el-input v-model="storyData.protagonist.name" placeholder="主角姓名" size="small" />
                  </div>
                </div>

                <!-- 提示词选择和编辑区 -->
                <div class="prompt-area">
                  <div class="prompt-header">
                    <span>创作提示词</span>
                    <div class="prompt-actions">
                      <el-button 
                        size="small" 
                        text 
                        type="info"
                        @click="showPromptSelector = true"
                      >
                        <el-icon><List /></el-icon>
                        选择模板
                      </el-button>
                      <el-button 
                        size="small" 
                        text 
                        type="primary" 
                        @click="showAdvancedConfig = !showAdvancedConfig"
                      >
                        {{ showAdvancedConfig ? '收起高级设置' : '展开高级设置' }}
                      </el-button>
                    </div>
                  </div>
                  
                  <!-- 当前选择的提示词模板 -->
                  <div v-if="selectedPromptTemplate" class="selected-template">
                    <div class="template-info">
                      <el-tag type="info" size="small">已选择模板</el-tag>
                      <span class="template-title">{{ selectedPromptTemplate.title }}</span>
                      <el-button 
                        size="small" 
                        type="text" 
                        @click="clearSelectedTemplate"
                      >
                        清除
                      </el-button>
                    </div>
                    <div class="template-description">{{ selectedPromptTemplate.description }}</div>
                  </div>
                  
                  <!-- 提示词编辑区 -->
                  <el-input
                    v-model="unifiedPrompt"
                    type="textarea"
                    :rows="6"
                    :placeholder="promptPlaceholder"
                    class="unified-prompt-input"
                  />
                </div>
              </div>

              <!-- 高级配置区（可折叠） -->
              <div v-if="showAdvancedConfig" class="advanced-config">
                <el-divider content-position="left">高级配置</el-divider>
                
                <el-form :model="storyData" label-width="80px" size="small">
                  <el-form-item label="主角性别">
                    <el-radio-group v-model="storyData.protagonist.gender" size="small">
                      <el-radio label="male">男</el-radio>
                      <el-radio label="female">女</el-radio>
                    </el-radio-group>
                  </el-form-item>
                  
                  <el-form-item label="主角年龄">
                    <el-input-number 
                      v-model="storyData.protagonist.age" 
                      :min="1" 
                      :max="100" 
                      size="small"
                      style="width: 120px"
                    />
                  </el-form-item>
                  
                  <el-form-item label="故事地点">
                    <el-input v-model="storyData.location" placeholder="描述故事发生的地点" />
                  </el-form-item>
                  
                  <el-form-item label="参考文本">
                    <el-input 
                      v-model="storyData.referenceText" 
                      type="textarea" 
                      :rows="3"
                      placeholder="可以粘贴一些参考文本或风格样例（可选）"
                    />
                  </el-form-item>
                </el-form>
              </div>
           </div>
         </el-scrollbar>
        </div>
      </div>
      
      <!-- 右侧内容区域 -->
      <div class="content-panel">
        <div class="panel-header">
          <h3>{{ storyData.title || '生成内容' }}</h3>
          <div class="content-actions" v-if="generatedStory">
            <el-button size="small" @click="regenerateStory">
              <el-icon><Refresh /></el-icon>
              重新生成
            </el-button>
            <el-button size="small" @click="continueStory" :loading="continuingStory">
              <el-icon><Plus /></el-icon>
              {{ continuingStory ? '续写中...' : '续写' }}
            </el-button>
            <el-button size="small" @click="showOptimizeDialog">
              <el-icon><EditPen /></el-icon>
              选段优化
            </el-button>
            <el-button size="small" @click="showAiAssistant = true">
              <el-icon><ChatDotRound /></el-icon>
              AI助手
            </el-button>
          </div>
        </div>
        
        <div class="content-body">
          <!-- 富文本编辑器 - 始终显示 -->
          <div class="story-result">
            <div class="story-editor">
              <div class="editor-wrapper">
                <Toolbar
                  :editor="editorRef"
                  :defaultConfig="toolbarConfig"
                  mode="default"
                  style="border-bottom: 1px solid #e4e7ed;"
                />
                <Editor
                  v-model="generatedStory"
                  :defaultConfig="editorConfig"
                  mode="default"
                  @onCreated="handleEditorCreated"
                  @onChange="onEditorChange"
                  style="height: 620px; overflow-y: hidden;"
                />
              </div>
            </div>
          </div>
        </div>
        
        <!-- 底部操作栏 -->
        <div class="content-footer">
          <div class="word-count">
            <el-tag>字数：{{ getTextWordCount(generatedStory) }}</el-tag>
          </div>
          <div class="footer-actions">

            <el-button @click="exportStory" :disabled="!generatedStory">
              <el-icon><Download /></el-icon>
              导出文档
            </el-button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- AI助手对话框 -->
    <el-dialog v-model="showAiAssistant" title="AI写作助手" width="600px">
      <div class="ai-assistant">
        <div class="chat-history">
          <div 
            v-for="(message, index) in chatHistory" 
            :key="index"
            class="chat-message"
            :class="{ 'user': message.type === 'user', 'ai': message.type === 'ai' }"
          >
            <div class="message-content">{{ message.content }}</div>
          </div>
        </div>
        <div class="chat-input">
          <el-input 
            v-model="assistantInput"
            placeholder="请输入您的问题或需求..."
            @keyup.enter="sendToAssistant"
          />
          <el-button type="primary" @click="sendToAssistant" :loading="assistantLoading">
            发送
          </el-button>
        </div>
      </div>
    </el-dialog>

    <!-- 选段优化弹窗 -->
    <el-dialog v-model="showOptimizeModal" title="选段优化" width="700px" :close-on-click-modal="false">
      <div class="optimize-dialog">
        <div class="selected-content">
          <h4>选中的内容：</h4>
          <div class="selected-text">{{ selectedTextForOptimize }}</div>
        </div>
        
        <div class="optimize-direction">
          <h4>优化方向：</h4>
          <el-input 
            v-model="optimizeDirection"
            type="textarea"
            :rows="3"
            placeholder="请描述您希望如何优化这段文字，例如：让语言更生动、增加细节描写、调整语气等..."
          />
        </div>
        
        <div class="optimize-actions">
          <el-button type="primary" @click="performOptimize" :loading="optimizing">
            <el-icon><MagicStick /></el-icon>
            {{ optimizing ? '优化中...' : '开始优化' }}
          </el-button>
        </div>
        
        <div v-if="optimizedResult || optimizing" class="optimize-result">
          <h4>{{ optimizing ? '优化中...' : '优化结果：' }}</h4>
          <div class="optimized-text" ref="optimizedTextRef">
            <div v-if="optimizing && !optimizedResult" class="optimizing-placeholder">
              <el-icon class="is-loading"><Loading /></el-icon>
              正在生成优化内容...
            </div>
            <div v-else class="optimized-content">{{ optimizedResult }}</div>
          </div>
          <div v-if="!optimizing && optimizedResult" class="result-actions">
            <el-button type="success" @click="copyOptimizedText">
              <el-icon><DocumentCopy /></el-icon>
              复制内容
            </el-button>
            <el-button @click="replaceOriginalText">
              <el-icon><Switch /></el-icon>
              替换原文
            </el-button>
          </div>
        </div>
      </div>
    </el-dialog>

    <!-- 配置管理弹窗 -->
    <el-dialog 
      v-model="showConfigManager" 
      title="创作配置管理" 
      width="800px"
      :close-on-click-modal="false"
    >
      <el-tabs v-model="activeConfigTab" type="border-card">
        <!-- 题材管理 -->
        <el-tab-pane label="题材" name="genres">
          <div class="config-section">
            <div class="config-header">
              <h4>题材配置</h4>
              <el-button size="small" type="primary" @click="addConfigItem('genres')">
                <el-icon><Plus /></el-icon>
                添加题材
              </el-button>
            </div>
            <div class="config-list">
              <div 
                v-for="(item, index) in configData.genres" 
                :key="index"
                class="config-item"
              >
                <el-input v-model="item.label" placeholder="题材名称" size="small" />
                <el-input v-model="item.value" placeholder="题材值" size="small" />
                <el-input v-model="item.description" placeholder="题材描述（可选）" size="small" />
                <el-button 
                  size="small" 
                  type="danger" 
                  text 
                  @click="removeConfigItem('genres', index)"
                >
                  删除
                </el-button>
              </div>
            </div>
          </div>
        </el-tab-pane>

        <!-- 情节管理 -->
        <el-tab-pane label="情节" name="plotTypes">
          <div class="config-section">
            <div class="config-header">
              <h4>情节配置</h4>
              <el-button size="small" type="primary" @click="addConfigItem('plotTypes')">
                <el-icon><Plus /></el-icon>
                添加情节
              </el-button>
            </div>
            <div class="config-list">
              <div 
                v-for="(item, index) in configData.plotTypes" 
                :key="index"
                class="config-item"
              >
                <el-input v-model="item.label" placeholder="情节名称" size="small" />
                <el-input v-model="item.value" placeholder="情节值" size="small" />
                <el-input v-model="item.description" placeholder="情节描述（可选）" size="small" />
                <el-button 
                  size="small" 
                  type="danger" 
                  text 
                  @click="removeConfigItem('plotTypes', index)"
                >
                  删除
                </el-button>
              </div>
            </div>
          </div>
        </el-tab-pane>

        <!-- 氛围管理 -->
        <el-tab-pane label="氛围" name="emotions">
          <div class="config-section">
            <div class="config-header">
              <h4>氛围配置</h4>
              <el-button size="small" type="primary" @click="addConfigItem('emotions')">
                <el-icon><Plus /></el-icon>
                添加氛围
              </el-button>
            </div>
            <div class="config-list">
              <div 
                v-for="(item, index) in configData.emotions" 
                :key="index"
                class="config-item"
              >
                <el-input v-model="item.label" placeholder="氛围名称（支持表情符号）" size="small" />
                <el-input v-model="item.value" placeholder="氛围值" size="small" />
                <el-input v-model="item.description" placeholder="氛围描述（可选）" size="small" />
                <el-button 
                  size="small" 
                  type="danger" 
                  text 
                  @click="removeConfigItem('emotions', index)"
                >
                  删除
                </el-button>
              </div>
            </div>
          </div>
        </el-tab-pane>

        <!-- 时代管理 -->
        <el-tab-pane label="时代" name="timeFrames">
          <div class="config-section">
            <div class="config-header">
              <h4>时代配置</h4>
              <el-button size="small" type="primary" @click="addConfigItem('timeFrames')">
                <el-icon><Plus /></el-icon>
                添加时代
              </el-button>
            </div>
            <div class="config-list">
              <div 
                v-for="(item, index) in configData.timeFrames" 
                :key="index"
                class="config-item"
              >
                <el-input v-model="item.label" placeholder="时代名称" size="small" />
                <el-input v-model="item.value" placeholder="时代值" size="small" />
                <el-input v-model="item.description" placeholder="时代描述（可选）" size="small" />
                <el-button 
                  size="small" 
                  type="danger" 
                  text 
                  @click="removeConfigItem('timeFrames', index)"
                >
                  删除
                </el-button>
              </div>
            </div>
          </div>
        </el-tab-pane>
      </el-tabs>

      <template #footer>
        <div class="dialog-footer">
          <el-button @click="showConfigManager = false">取消</el-button>
          <el-button @click="resetToDefault">恢复默认</el-button>
          <el-button type="primary" @click="saveConfigData">保存配置</el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 续写弹窗 -->
    <el-dialog 
      v-model="showContinueDialog" 
      title="智能续写" 
      width="900px"
      :close-on-click-modal="false"
      class="continue-dialog"
    >
      <div class="continue-container">
        <!-- 左侧：续写配置 -->
        <div class="continue-config">
          <div class="config-section">
            <h4>续写方向</h4>
            <el-input
              v-model="continueDirection"
              type="textarea"
              :rows="4"
              placeholder="请描述您希望故事如何发展，例如：希望主角遇到新的挑战、希望故事走向高潮、希望增加新的角色等..."
            />
          </div>
          
          <div class="config-section">
            <h4>续写字数</h4>
            <el-input-number
              v-model="continueWordCount"
              :min="100"
              :max="10000"
              :step="100"
              placeholder="续写字数"
              style="width: 100%"
            />
            <div class="word-count-tips">
              <span>建议字数：100-10000字</span>
            </div>
          </div>
          
          <div class="config-section">
            <h4>续写建议：</h4>
            <ul class="tips-list">
              <li>描述您希望故事发展的方向</li>
              <li>可以指定新的情节转折点</li>
              <li>可以要求增加新的角色或场景</li>
              <li>可以指定希望达到的情感效果</li>
            </ul>
          </div>
          
          <div class="config-actions">
            <el-button @click="showContinueDialog = false">取消</el-button>
            <el-button type="primary" @click="performContinue" :loading="continuingStory">
              {{ continuingStory ? '续写中...' : '开始续写' }}
            </el-button>
          </div>
        </div>
        
        <!-- 右侧：续写结果 -->
        <div class="continue-result">
          <div class="result-header">
            <h4>续写结果</h4>
          </div>
          
          <div class="result-content">
            <div v-if="continueResult" class="continued-content">
              <div class="continued-text" ref="continueTextRef">
                {{ continueResult }}
              </div>
              <div v-if="continuingStory" class="continuing-indicator">
                <el-icon class="loading-icon"><Loading /></el-icon>
                <span>续写中...</span>
              </div>
            </div>
            
            <div v-else-if="continuingStory" class="continuing-placeholder">
              <el-icon class="loading-icon"><Loading /></el-icon>
              <span>续写中...</span>
            </div>
            
            <div v-else class="empty-placeholder">
              <el-empty description="点击开始续写按钮生成续写内容" />
            </div>
          </div>
          
          <div v-if="continueResult && !continuingStory" class="result-actions">
            <el-button type="primary" @click="copyContinueText">
              <el-icon><DocumentCopy /></el-icon>
              复制内容
            </el-button>
          </div>
        </div>
      </div>
    </el-dialog>

    <!-- 提示词选择器对话框 -->
    <el-dialog 
      v-model="showPromptSelector" 
      title="选择短篇小说提示词模板" 
      width="900px"
      @close="resetPromptSelector"
    >
      <div class="prompt-selector">
        <!-- 提示词列表 -->
        <div class="prompt-list">
          <div class="prompt-grid">
            <div 
              v-for="prompt in shortStoryPrompts"
              :key="prompt.id"
              class="prompt-card"
              :class="{ active: selectedPromptId === prompt.id }"
              @click="selectPrompt(prompt)"
            >
              <div class="prompt-card-header">
                <h5>{{ prompt.title }}</h5>
                <el-icon v-if="selectedPromptId === prompt.id" class="selected-icon"><Check /></el-icon>
              </div>
              <div class="prompt-card-description">
                <p>{{ prompt.description }}</p>
              </div>
              <div class="prompt-card-tags">
                <el-tag v-for="tag in prompt.tags" :key="tag" size="small" type="info">{{ tag }}</el-tag>
              </div>
            </div>
          </div>
          
          <div v-if="shortStoryPrompts.length === 0" class="empty-prompts">
            <el-empty description="暂无短篇小说提示词模板">
              <el-button type="primary" @click="goToPromptLibrary">去提示词库添加</el-button>
            </el-empty>
          </div>
        </div>

        <!-- 预览区域 -->
        <div v-if="previewPrompt" class="prompt-preview">
          <h4>模板预览</h4>
          <div class="preview-content">
            <el-input
              v-model="editablePromptContent"
              type="textarea"
              :rows="12"
              placeholder="提示词内容"
              class="prompt-content-editor"
            />
          </div>
        </div>
      </div>
      
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="showPromptSelector = false">取消</el-button>
          <el-button @click="useOriginalPrompt" :disabled="!previewPrompt">使用原版</el-button>
          <el-button type="primary" @click="useEditedPrompt" :disabled="!previewPrompt">使用编辑版</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, shallowRef, onMounted, nextTick } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { MagicStick, Refresh, EditPen, ChatDotRound, Download, Check, Loading, Plus, Setting, List, DocumentCopy, Switch } from '@element-plus/icons-vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'
import '@wangeditor/editor/dist/css/style.css'
import { useNovelStore } from '@/stores/novel'
import { useRouter } from 'vue-router'

const novelStore = useNovelStore()
const router = useRouter()

// 响应式数据
const generating = ref(false)
const continuingStory = ref(false)
const generatedStory = ref('')
const hasSelection = ref(false)
const selectedText = ref('')
const showAiAssistant = ref(false)
const assistantInput = ref('')
const assistantLoading = ref(false)
const chatHistory = ref([])
const showAdvancedConfig = ref(false)
const unifiedPrompt = ref('')

// 续写相关
const showContinueDialog = ref(false)
const continueDirection = ref('')
const continueWordCount = ref(2000)
const continueResult = ref('')
const continueTextRef = ref(null)

// 配置管理相关
const showConfigManager = ref(false)
const activeConfigTab = ref('genres')

// 提示词选择相关
const showPromptSelector = ref(false)
const selectedPromptId = ref(null)
const selectedPromptTemplate = ref(null)
const previewPrompt = ref(null)
const editablePromptContent = ref('')
const availablePrompts = ref([])

// 选段优化相关
const showOptimizeModal = ref(false)
const selectedTextForOptimize = ref('')
const optimizeDirection = ref('')
const optimizing = ref(false)
const optimizedResult = ref('')
const optimizedTextRef = ref(null)

// 计算属性 - 短篇小说提示词
const shortStoryPrompts = computed(() => {
  return availablePrompts.value.filter(prompt => prompt.category === 'short-story')
})

// 计算属性 - 提示词占位符
const promptPlaceholder = computed(() => {
  if (selectedPromptTemplate.value) {
    return '请编辑上方选择的提示词模板，可以根据需要修改内容'
  }
  return `请详细描述您想要创作的短篇小说，包括：
• 主角的性格特点和背景
• 故事情节和冲突
• 场景和环境描述
• 您希望的故事风格和结局

例如：创作一篇都市爱情小说，主角是25岁的软件工程师李明，性格内向但善良。故事讲述他在咖啡馆遇到了画家女孩小雅，两人从陌生到相知相爱的过程。希望故事温馨感人，有一些生活的小细节，结局美满。`
})

// WangEditor相关
const editorRef = shallowRef()
const toolbarConfig = {}
const editorConfig = {
  placeholder: '生成的小说内容将显示在这里...',
  MENU_CONF: {
    uploadImage: {
      server: '/api/upload-image',
      fieldName: 'file',
      maxFileSize: 5 * 1024 * 1024,
      allowedFileTypes: ['image/*']
    }
  }
}

// 故事数据
const storyData = reactive({
  genre: '',
  protagonist: {
    name: '',
    gender: 'male',
    age: 25
  },
  plotType: '',
  emotion: '',
  timeFrame: '',
  location: '',
  referenceText: '',
  title: '',
  wordCount: 3000
})

// 默认配置数据
const defaultConfigData = {
  genres: [
    { value: 'urban', label: '都市生活', description: '现代都市背景，贴近生活' },
    { value: 'urban_evil', label: '都市恶灵', description: '都市背景的恐怖灵异故事' },
    { value: 'fantasy', label: '奇幻冒险', description: '魔法世界，英雄历险' },
    { value: 'romance', label: '浪漫爱情', description: '感人爱情故事' },
    { value: 'mystery', label: '悬疑推理', description: '谜题解密，逻辑推理' },
    { value: 'scifi', label: '科幻未来', description: '未来科技，星际探索' },
    { value: 'horror', label: '惊悚恐怖', description: '恐怖氛围，惊心动魄' }
  ],
  plotTypes: [
    { value: 'growth', label: '成长蜕变', description: '主角经历挫折后成长' },
    { value: 'adventure', label: '冒险探索', description: '探索未知，寻找宝藏' },
    { value: 'conflict', label: '冲突解决', description: '面对冲突，寻求解决' },
    { value: 'redemption', label: '救赎重生', description: '犯错后的救赎之路' },
    { value: 'discovery', label: '发现真相', description: '揭露隐藏的秘密' }
  ],
  emotions: [
    { value: 'happy', label: '😊 欢乐', description: '轻松愉快的氛围' },
    { value: 'sad', label: '😢 悲伤', description: '感人催泪的情感' },
    { value: 'tense', label: '😰 紧张', description: '紧张刺激的氛围' },
    { value: 'romantic', label: '💕 浪漫', description: '温馨浪漫的情调' },
    { value: 'mysterious', label: '🔮 神秘', description: '神秘未知的氛围' }
  ],
  timeFrames: [
    { value: 'ancient', label: '古代', description: '古代背景设定' },
    { value: 'modern', label: '近代', description: '近代历史背景' },
    { value: 'contemporary', label: '当代', description: '现代社会背景' },
    { value: 'future', label: '未来', description: '未来科幻背景' }
  ]
}

// 配置数据
const configData = reactive({
  genres: [],
  plotTypes: [],
  emotions: [],
  timeFrames: []
})

// 计算属性
const isConfigValid = computed(() => {
  return storyData.title && 
         storyData.protagonist.name && 
         unifiedPrompt.value.trim().length > 0
})

// 获取当前配置选项
const customGenres = computed(() => configData.genres)
const customPlotTypes = computed(() => configData.plotTypes)
const customEmotions = computed(() => configData.emotions)
const customTimeFrames = computed(() => configData.timeFrames)

// 方法

const generateStory = async () => {
  if (generating.value) return
  
  generating.value = true
  generatedStory.value = ''
  
  try {
    const prompt = buildStoryPrompt()
    
    // 添加详细的调试信息
    console.log('=== 短篇小说生成调试信息 ===')
    console.log('prompt类型:', typeof prompt)
    console.log('prompt长度:', prompt.length)
    console.log('prompt内容:', prompt)
    
    // 检查prompt中是否包含可能导致JSON问题的字符
    const problematicChars = prompt.match(/[\u0000-\u001F\u007F-\u009F]/g)
    if (problematicChars) {
      console.warn('发现控制字符:', problematicChars)
    }
    
    // 检查是否有未转义的引号
    const unescapedQuotes = prompt.match(/(?<!\\)"/g)
    if (unescapedQuotes) {
      console.warn('发现未转义的引号数量:', unescapedQuotes.length)
    }
    
    // 尝试JSON序列化测试
    try {
      JSON.stringify({ content: prompt })
      console.log('JSON序列化测试通过')
    } catch (jsonError) {
      console.error('JSON序列化测试失败:', jsonError)
      throw new Error('提示词包含无法序列化的字符: ' + jsonError.message)
    }
    
    // 使用流式返回
    let accumulatedText = ''
    await novelStore.generateContent(prompt, (chunk) => {
      accumulatedText += chunk
      // 将纯文本转换为HTML格式
      generatedStory.value = accumulatedText.replace(/\n/g, '<br/>')
    })
    
    ElMessage.success('小说生成成功！')
  } catch (error) {
    console.error('=== 生成失败详细信息 ===')
    console.error('错误类型:', error.constructor.name)
    console.error('错误消息:', error.message)
    console.error('错误堆栈:', error.stack)
    ElMessage.error('生成失败：' + error.message)
  } finally {
    generating.value = false
  }
}

const buildStoryPrompt = () => {
  const { protagonist, genre, plotType, emotion, timeFrame, location } = storyData
  
  let prompt = `请根据以下要求创作一篇短篇小说：\n\n`
  
  // 基础信息 - 始终包含所有参数设置
  prompt += `【基础设定】\n`
  prompt += `- 小说标题：${storyData.title}\n`
  prompt += `- 主角姓名：${protagonist.name}`
  if (protagonist.gender) {
    prompt += `（${protagonist.gender === 'male' ? '男性' : '女性'}`
    if (protagonist.age) {
      prompt += `，${protagonist.age}岁`
    }
    prompt += `）`
  }
  prompt += `\n`
  
  // 所有设置参数都传递给AI
  if (genre) {
    const genreInfo = customGenres.value.find(g => g.value === genre)
    prompt += `- 题材风格：${genreInfo?.label || genre}\n`
  }
  if (plotType) {
    const plotInfo = customPlotTypes.value.find(p => p.value === plotType)
    prompt += `- 情节类型：${plotInfo?.label || plotType}\n`
  }
  if (emotion) {
    const emotionInfo = customEmotions.value.find(e => e.value === emotion)
    // 修复表情符号处理，确保JSON序列化安全
    let emotionLabel = emotion
    if (emotionInfo && emotionInfo.label) {
      // 移除所有表情符号和特殊字符，只保留文字
      emotionLabel = emotionInfo.label.replace(/[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]/gu, '').trim()
      // 如果去掉表情符号后为空，使用原始emotion值
      if (!emotionLabel) {
        emotionLabel = emotion
      }
    }
    prompt += `- 情绪氛围：${emotionLabel}\n`
  }
  if (timeFrame) {
    const timeInfo = customTimeFrames.value.find(t => t.value === timeFrame)
    prompt += `- 时间背景：${timeInfo?.label || timeFrame}\n`
  }
  if (location) {
    prompt += `- 故事地点：${location}\n`
  }
  
  // 字数要求 - 现在是数字形式
  if (storyData.wordCount) {
    prompt += `- 目标字数：${storyData.wordCount}字\n`
  }
  
  // 创作要求部分 - 包含提示词模板和自定义要求
  prompt += `\n【创作要求】\n`
  
  // 如果使用了提示词模板，将其作为创作要求的一部分
  if (selectedPromptTemplate.value && unifiedPrompt.value) {
    prompt += `${unifiedPrompt.value}\n\n`
    console.log('已将提示词模板和所有参数设置传递给AI')
  } else if (unifiedPrompt.value) {
    prompt += `${unifiedPrompt.value}\n\n`
  }
  
  if (storyData.referenceText) {
    prompt += `【参考文本】\n${storyData.referenceText}\n\n`
  }
  
  prompt += `请创作一篇完整的短篇小说，字数控制在${storyData.wordCount}字左右，要求情节完整，人物鲜明，语言生动。`
  
  // 添加调试日志
  console.log('构建的prompt长度:', prompt.length)
  console.log('prompt预览:', prompt.substring(0, 200) + '...')
  
  return prompt
}

const regenerateStory = () => {
  generatedStory.value = ''
  generateStory()
}

// 续写功能
const continueStory = async () => {
  if (continuingStory.value) return
  
  // 显示续写弹窗
  showContinueDialog.value = true
  continueDirection.value = ''
  continueResult.value = ''
}

// 执行续写
const performContinue = async () => {
  if (continuingStory.value) return
  
  // 获取当前故事内容（去除HTML标签）
  const currentText = generatedStory.value ? generatedStory.value.replace(/<[^>]*>/g, '') : ''
  if (!currentText.trim()) {
    ElMessage.warning('请先生成一些内容再进行续写')
    return
  }
  
  continuingStory.value = true
  continueResult.value = ''
  
  try {
    // 构建续写提示词
    const continuePrompt = buildContinuePrompt(currentText)
    
    console.log('=== 续写调试信息 ===')
    console.log('续写prompt长度:', continuePrompt.length)
    console.log('当前内容长度:', currentText.length)
    console.log('续写方向:', continueDirection.value)
    
    // 使用流式返回，实时更新续写结果
    await novelStore.generateContent(continuePrompt, (chunk) => {
      continueResult.value += chunk
      // 自动滚动到底部
      nextTick(() => {
        if (continueTextRef.value) {
          continueTextRef.value.scrollTop = continueTextRef.value.scrollHeight
        }
      })
    })
    
    ElMessage.success('续写完成！')
  } catch (error) {
    console.error('续写失败:', error)
    ElMessage.error('续写失败：' + error.message)
  } finally {
    continuingStory.value = false
  }
}



// 复制续写内容
const copyContinueText = async () => {
  try {
    if (!continueResult.value.trim()) {
      ElMessage.warning('没有续写内容可以复制')
      return
    }
    
    await navigator.clipboard.writeText(continueResult.value)
    ElMessage.success('续写内容已复制到剪贴板')
  } catch (error) {
    console.error('复制失败:', error)
    ElMessage.error('复制失败，请手动复制')
  }
}

// 构建续写提示词
const buildContinuePrompt = (currentText) => {
  const { protagonist, genre, plotType, emotion, timeFrame, location } = storyData
  
  let prompt = `请继续续写以下短篇小说，保持风格和情节的连贯性：\n\n`
  
  // 添加原始设置信息，保持一致性
  prompt += `【原始设定】\n`
  prompt += `- 小说标题：${storyData.title}\n`
  prompt += `- 主角姓名：${protagonist.name}`
  if (protagonist.gender) {
    prompt += `（${protagonist.gender === 'male' ? '男性' : '女性'}`
    if (protagonist.age) {
      prompt += `，${protagonist.age}岁`
    }
    prompt += `）`
  }
  prompt += `\n`
  
  if (genre) {
    const genreInfo = customGenres.value.find(g => g.value === genre)
    prompt += `- 题材风格：${genreInfo?.label || genre}\n`
  }
  if (emotion) {
    const emotionInfo = customEmotions.value.find(e => e.value === emotion)
    let emotionLabel = emotion
    if (emotionInfo && emotionInfo.label) {
      emotionLabel = emotionInfo.label.replace(/[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]/gu, '').trim()
      if (!emotionLabel) {
        emotionLabel = emotion
      }
    }
    prompt += `- 情绪氛围：${emotionLabel}\n`
  }
  
  prompt += `\n【当前内容】\n${currentText}\n\n`
  
  prompt += `【续写要求】\n`
  prompt += `请继续续写这个故事，保持以下要求：\n`
  prompt += `1. 保持与前文的风格和语调一致\n`
  prompt += `2. 情节发展自然流畅，不要突兀转折\n`
  prompt += `3. 继续深入刻画人物性格\n`
  prompt += `4. 续写长度约${continueWordCount.value}字\n`
  prompt += `5. 推进故事情节向高潮或结局发展\n`
  
  // 添加用户指定的续写方向
  if (continueDirection.value.trim()) {
    prompt += `6. 按照以下方向发展：${continueDirection.value}\n`
  }
  
  prompt += `\n请直接开始续写，不要重复前面的内容：`
  
  return prompt
}

const resetConfig = () => {
  ElMessageBox.confirm(
    '确定要重置所有配置吗？这将清空当前所有设置内容。',
    '重置确认',
    {
      confirmButtonText: '确定重置',
      cancelButtonText: '取消',
      type: 'warning',
    }
  ).then(() => {
    // 重置所有配置
    storyData.genre = ''
    storyData.title = ''
    storyData.plotType = ''
    storyData.emotion = ''
    storyData.timeFrame = ''
    storyData.location = ''
    storyData.referenceText = ''
    storyData.wordCount = 3000
    storyData.protagonist.name = ''
    storyData.protagonist.gender = 'male'
    storyData.protagonist.age = 25
    unifiedPrompt.value = ''
    showAdvancedConfig.value = false
    
    ElMessage.success('配置已重置')
  }).catch(() => {
    // 用户取消
  })
}

const handleEditorCreated = (editor) => {
  editorRef.value = editor
}

const onEditorChange = (editor) => {
  // 编辑器内容变化时的处理，v-model会自动处理
}

const handleTextSelection = (event) => {
  const selection = window.getSelection().toString()
  if (selection.length > 0) {
    selectedText.value = selection
    hasSelection.value = true
  } else {
    hasSelection.value = false
  }
}

// 显示选段优化弹窗
const showOptimizeDialog = () => {
  if (!editorRef.value) {
    ElMessage.warning('编辑器未初始化')
    return
  }
  
  const selectedText = editorRef.value.getSelectionText()
  if (!selectedText) {
    ElMessage.warning('请先选择要优化的文本')
    return
  }
  
  selectedTextForOptimize.value = selectedText
  optimizeDirection.value = ''
  optimizedResult.value = ''
  showOptimizeModal.value = true
}

// 执行优化
const performOptimize = async () => {
  if (!selectedTextForOptimize.value) {
    ElMessage.warning('没有选中的文本')
    return
  }
  
  if (!optimizeDirection.value.trim()) {
    ElMessage.warning('请填写优化方向')
    return
  }
  
  optimizing.value = true
  optimizedResult.value = ''
  
  try {
    let prompt = `请根据以下要求优化这段文字：\n\n`
    prompt += `【优化方向】\n${optimizeDirection.value}\n\n`
    prompt += `【原文】\n${selectedTextForOptimize.value}\n\n`
    prompt += `请直接输出优化后的文字，保持原文的基本意思，但要按照优化方向进行改进。`
    
    // 使用流式输出，实时显示优化过程
    await novelStore.generateContent(prompt, (chunk) => {
      optimizedResult.value += chunk
      
      // 自动滚动到底部，显示最新内容
      nextTick(() => {
        if (optimizedTextRef.value) {
          optimizedTextRef.value.scrollTop = optimizedTextRef.value.scrollHeight
        }
      })
    })
    
    ElMessage.success('优化完成！')
  } catch (error) {
    console.error('优化失败:', error)
    ElMessage.error('优化失败：' + error.message)
  } finally {
    optimizing.value = false
  }
}

// 复制优化后的文本
const copyOptimizedText = async () => {
  if (!optimizedResult.value) {
    ElMessage.warning('没有优化结果可复制')
    return
  }
  
  try {
    await navigator.clipboard.writeText(optimizedResult.value)
    ElMessage.success('已复制到剪贴板')
  } catch (error) {
    // 如果clipboard API不可用，使用传统方法
    const textArea = document.createElement('textarea')
    textArea.value = optimizedResult.value
    document.body.appendChild(textArea)
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
    ElMessage.success('已复制到剪贴板')
  }
}

// 替换原文
const replaceOriginalText = () => {
  if (!optimizedResult.value || !editorRef.value) {
    ElMessage.warning('无法替换原文')
    return
  }
  
  try {
    // 获取当前编辑器内容
    const currentContent = generatedStory.value || ''
    
    // 替换选中的文本
    const newContent = currentContent.replace(selectedTextForOptimize.value, optimizedResult.value)
    generatedStory.value = newContent
    
    // 关闭弹窗
    showOptimizeModal.value = false
    
    ElMessage.success('已替换原文')
  } catch (error) {
    console.error('替换失败:', error)
    ElMessage.error('替换失败：' + error.message)
  }
}

const optimizeSelection = async () => {
  // 保留原有方法以防兼容性问题
  showOptimizeDialog()
}

const sendToAssistant = async () => {
  if (!assistantInput.value.trim()) return
  
  const userMessage = assistantInput.value
  chatHistory.value.push({ type: 'user', content: userMessage })
  assistantInput.value = ''
  assistantLoading.value = true
  
  try {
    const currentText = generatedStory.value ? generatedStory.value.replace(/<[^>]*>/g, '') : ''
    const prompt = `作为一个专业的写作助手，请回答以下问题：\n\n${userMessage}\n\n当前小说内容：\n${currentText.slice(0, 500)}...`
    const response = await novelStore.generateContent(prompt)
    chatHistory.value.push({ type: 'ai', content: response })
  } catch (error) {
    ElMessage.error('AI助手回复失败')
  } finally {
    assistantLoading.value = false
  }
}



const exportStory = () => {
  // 实现导出功能
  const pureText = generatedStory.value ? generatedStory.value.replace(/<[^>]*>/g, '') : ''
  const content = `${storyData.title}\n\n${storyData.synopsis}\n\n${pureText}`
  const blob = new Blob([content], { type: 'text/plain;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `${storyData.title || '短篇小说'}.txt`
  link.click()
  URL.revokeObjectURL(url)
}

// 获取纯文本字数统计
const getTextWordCount = (html) => {
  if (!html) return 0
  // 移除HTML标签
  const text = html.replace(/<[^>]*>/g, '')
  return text.length
}

// 配置管理方法
const loadConfigData = () => {
  try {
    const savedConfig = localStorage.getItem('shortStoryConfig')
    if (savedConfig) {
      const config = JSON.parse(savedConfig)
      Object.keys(defaultConfigData).forEach(key => {
        configData[key] = config[key] || [...defaultConfigData[key]]
      })
    } else {
      // 首次使用，加载默认配置
      Object.keys(defaultConfigData).forEach(key => {
        configData[key] = [...defaultConfigData[key]]
      })
    }
  } catch (error) {
    console.error('加载配置失败:', error)
    // 出错时使用默认配置
    Object.keys(defaultConfigData).forEach(key => {
      configData[key] = [...defaultConfigData[key]]
    })
  }
}

const saveConfigData = () => {
  try {
    localStorage.setItem('shortStoryConfig', JSON.stringify(configData))
    ElMessage.success('配置保存成功！')
    showConfigManager.value = false
  } catch (error) {
    console.error('保存配置失败:', error)
    ElMessage.error('保存配置失败')
  }
}

const addConfigItem = (type) => {
  configData[type].push({
    label: '',
    value: '',
    description: ''
  })
}

const removeConfigItem = (type, index) => {
  configData[type].splice(index, 1)
}

const resetToDefault = () => {
  ElMessageBox.confirm(
    '确定要恢复默认配置吗？这将清除所有自定义配置。',
    '恢复默认配置',
    {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    }
  ).then(() => {
    Object.keys(defaultConfigData).forEach(key => {
      configData[key] = [...defaultConfigData[key]]
    })
    ElMessage.success('已恢复默认配置')
  }).catch(() => {
    // 用户取消
  })
}

// 提示词管理方法
const loadPrompts = () => {
  try {
    const savedPrompts = localStorage.getItem('prompts')
    if (savedPrompts) {
      availablePrompts.value = JSON.parse(savedPrompts)
    } else {
      availablePrompts.value = []
    }
    console.log('短篇小说模块加载提示词数据:', availablePrompts.value.length)
  } catch (error) {
    console.error('加载提示词失败:', error)
    availablePrompts.value = []
  }
}

const selectPrompt = (prompt) => {
  selectedPromptId.value = prompt.id
  previewPrompt.value = prompt
  editablePromptContent.value = prompt.content
}

const resetPromptSelector = () => {
  selectedPromptId.value = null
  previewPrompt.value = null
  editablePromptContent.value = ''
}

const clearSelectedTemplate = () => {
  selectedPromptTemplate.value = null
  // 清空当前提示词内容，让用户重新输入
  unifiedPrompt.value = ''
  ElMessage.success('已清除提示词模板')
}

const useOriginalPrompt = () => {
  if (!previewPrompt.value) return
  
  // 使用原版提示词并填充变量
  const filledPrompt = fillPromptVariables(previewPrompt.value.content)
  unifiedPrompt.value = filledPrompt
  selectedPromptTemplate.value = previewPrompt.value
  showPromptSelector.value = false
  ElMessage.success('已使用原版提示词模板')
}

const useEditedPrompt = () => {
  if (!previewPrompt.value || !editablePromptContent.value) return
  
  // 使用编辑后的提示词并填充变量
  const filledPrompt = fillPromptVariables(editablePromptContent.value)
  unifiedPrompt.value = filledPrompt
  selectedPromptTemplate.value = { ...previewPrompt.value, content: editablePromptContent.value }
  showPromptSelector.value = false
  ElMessage.success('已使用编辑版提示词模板')
}

const fillPromptVariables = (promptContent) => {
  let result = promptContent
  
  // 填充基础信息变量
  const variables = {
    '小说标题': storyData.title || '{小说标题}',
    '主角姓名': storyData.protagonist.name || '{主角姓名}',
    '主角性别': storyData.protagonist.gender === 'male' ? '男' : '女',
    '主角年龄': storyData.protagonist.age || '{主角年龄}',
    '故事地点': storyData.location || '{故事地点}',
    '字数要求': getWordCountText(storyData.wordCount),
    '题材类型': getGenreText(storyData.genre) || '{题材类型}',
    '情节类型': getPlotText(storyData.plotType) || '{情节类型}',
    '情绪氛围': getEmotionText(storyData.emotion) || '{情绪氛围}',
    '时间背景': getTimeFrameText(storyData.timeFrame) || '{时间背景}',
    '创作要求': '请根据上述设定创作一篇精彩的短篇小说',
    '参考文本': storyData.referenceText || '无'
  }
  
  // 替换变量
  Object.keys(variables).forEach(key => {
    const regex = new RegExp(`\\{${key}\\}`, 'g')
    result = result.replace(regex, variables[key])
  })
  
  return result
}

// 辅助方法 - 获取选项文本
const getWordCountText = (value) => {
  // 现在wordCount是数字形式
  return `${value}字`
}

const getGenreText = (value) => {
  const genre = customGenres.value.find(g => g.value === value)
  return genre?.label
}

const getPlotText = (value) => {
  const plot = customPlotTypes.value.find(p => p.value === value)
  return plot?.label
}

const getEmotionText = (value) => {
  const emotion = customEmotions.value.find(e => e.value === value)
  return emotion?.label?.replace(/[😊😢😰💕🔮]\s/, '') || emotion?.label
}

const getTimeFrameText = (value) => {
  const timeFrame = customTimeFrames.value.find(t => t.value === value)
  return timeFrame?.label
}

const goToPromptLibrary = () => {
  router.push('/prompts')
}

// 页面初始化时加载配置
onMounted(() => {
  loadConfigData()
  loadPrompts()
})
</script>

<style scoped>
.short-story {
  width: 100%;
  height: 100vh;
  padding: 20px;
  box-sizing: border-box;
  background: #f5f7fa;
}


.story-workspace {
  display: flex;
  gap: 20px;
  height: calc(100vh - 120px);
}

.config-panel {
  width: 320px;
  flex-shrink: 0;
  background: white;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.config-scroll-container {
  flex: 1;
  overflow: hidden;
}

.config-panel .panel-header {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 16px;
  flex-shrink: 0;
}

.header-title-row {
  display: flex;
  justify-content: center;
  align-items: center;
}

.header-actions-row {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.secondary-actions {
  display: flex;
  gap: 6px;
  justify-content: center;
}

.primary-action {
  display: flex;
}

.config-panel .panel-header h3 {
  margin: 0;
  color: #2c3e50;
  font-size: 16px;
}

.required-items-tip {
  padding: 4px 8px;
  background: #fef0f0;
  border: 1px solid #fbc4c4;
  border-radius: 3px;
  font-size: 11px;
  line-height: 1.3;
  width: 100%;
  align-self: stretch;
}

.tip-label {
  color: #f56c6c;
  font-weight: 500;
  margin-right: 3px;
}

.missing-items {
  display: inline;
}

.missing-item {
  color: #f56c6c;
  font-size: 11px;
}

.missing-item:not(:last-child)::after {
  content: "、";
  margin: 0 1px;
  color: #f56c6c;
}

.config-content {
  padding-bottom: 20px;
}

.quick-config {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.basic-selects {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.select-row {
  display: flex;
  gap: 12px;
}

.select-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.select-item label {
  font-size: 12px;
  color: #606266;
  font-weight: 500;
}

.quick-inputs .input-row {
  display: flex;
  gap: 12px;
}

.prompt-area {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.prompt-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
  color: #606266;
  font-weight: 500;
}

.unified-prompt-input {
  border-radius: 4px;
}

.unified-prompt-input .el-textarea__inner {
  line-height: 1.5;
  font-family: 'PingFang SC', 'Helvetica Neue', 'Microsoft YaHei', sans-serif;
}

.advanced-config {
  margin-top: 16px;
}



.generate-section {
  text-align: center;
  padding: 60px 0;
}

.content-panel {
  flex: 1;
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  flex-shrink: 0;
}

.panel-header h3 {
  margin: 0;
  color: #2c3e50;
}

.content-body {
  flex: 1;
  /* overflow: auto; */
  position: relative;
}





.editor-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  border: 1px solid #e4e7ed;
  border-radius: 6px;
  overflow: hidden;
}

.editor-wrapper .w-e-text-container {
  flex: 1;
  overflow-y: auto;
}

.editor-wrapper .w-e-text {
  font-family: 'PingFang SC', 'Helvetica Neue', 'Microsoft YaHei', sans-serif;
  font-size: 14px;
  line-height: 1.8;
  padding: 16px;
  min-height: 400px;
}



.story-result {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.story-editor {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.story-textarea {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.content-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 0;
  border-top: 1px solid #e4e7ed;
  margin-top: 16px;
  flex-shrink: 0;
}

.word-count {
  margin: 0;
}

.footer-actions {
  display: flex;
  gap: 8px;
}

.add-custom-item {
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px dashed #c0c4cc;
  border-radius: 6px;
  padding: 12px;
  cursor: pointer;
  transition: all 0.3s;
  margin-top: 8px;
  background-color: #f5f7fa;
}

.add-custom-item:hover {
  border-color: #409eff;
  background-color: #ecf5ff;
}

.add-custom-item .el-icon {
  margin-right: 8px;
  font-size: 16px;
  color: #909399;
}

.add-custom-item:hover .el-icon {
  color: #409eff;
}

.ai-assistant {
  height: 400px;
  display: flex;
  flex-direction: column;
}

.chat-history {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
  background: #f8f9fa;
  border-radius: 4px;
  margin-bottom: 16px;
}

.chat-message {
  margin-bottom: 12px;
  padding: 8px 12px;
  border-radius: 8px;
  max-width: 80%;
}

.chat-message.user {
  background: #409eff;
  color: white;
  margin-left: auto;
}

.chat-message.ai {
  background: white;
  border: 1px solid #e4e7ed;
}

.chat-input {
  display: flex;
  gap: 8px;
}

.chat-input .el-input {
  flex: 1;
}

@media (max-width: 768px) {
  .story-header {
    flex-direction: column;
    gap: 16px;
  }
  
  .story-actions {
    flex-wrap: wrap;
    justify-content: center;
  }
}

/* 配置管理弹窗样式 */
.config-section {
  padding: 16px;
}

.config-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.config-header h4 {
  margin: 0;
  color: #2c3e50;
  font-size: 16px;
}

.config-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-height: 400px;
  overflow-y: auto;
}

.config-item {
  display: flex;
  gap: 8px;
  align-items: center;
  padding: 12px;
  border: 1px solid #e4e7ed;
  border-radius: 6px;
  background: #fafbfc;
  transition: all 0.3s;
}

.config-item:hover {
  border-color: #c6e2ff;
  background: #ecf5ff;
}

.config-item .el-input {
  flex: 1;
}

/* 让第三个输入框（提示词）更宽 */
.config-item .el-input:nth-child(3) {
  flex: 2;
}

.config-item .el-button {
  flex-shrink: 0;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}

.empty-config {
  text-align: center;
  padding: 40px 20px;
  color: #909399;
}

.empty-config .el-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

/* 提示词选择器样式 */
.prompt-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.prompt-actions {
  display: flex;
  gap: 8px;
}

.selected-template {
  background: #f0f9ff;
  border: 1px solid #bfdbfe;
  border-radius: 6px;
  padding: 12px;
  margin-bottom: 12px;
}

.template-info {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 4px;
}

.template-title {
  font-weight: 500;
  color: #1e40af;
}

.template-description {
  font-size: 12px;
  color: #64748b;
  line-height: 1.4;
}

.prompt-selector {
  display: flex;
  gap: 20px;
  min-height: 500px;
}

.prompt-list {
  flex: 1;
  max-height: 500px;
  overflow-y: auto;
}

.prompt-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

.prompt-card {
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 16px;
  cursor: pointer;
  transition: all 0.2s;
  background: white;
}

.prompt-card:hover {
  border-color: #3b82f6;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.1);
}

.prompt-card.active {
  border-color: #3b82f6;
  background: #eff6ff;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.15);
}

.prompt-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.prompt-card-header h5 {
  margin: 0;
  font-size: 14px;
  font-weight: 500;
  color: #1f2937;
}

.selected-icon {
  color: #3b82f6;
  font-size: 16px;
}

.prompt-card-description {
  margin-bottom: 12px;
}

.prompt-card-description p {
  margin: 0;
  font-size: 12px;
  color: #6b7280;
  line-height: 1.4;
}

.prompt-card-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.prompt-preview {
  flex: 1;
  border-left: 1px solid #e5e7eb;
  padding-left: 20px;
}

.prompt-preview h4 {
  margin: 0 0 16px 0;
  color: #1f2937;
  font-size: 16px;
}

.preview-content {
  height: calc(100% - 40px);
}

.prompt-content-editor {
  height: 100%;
}

.prompt-content-editor .el-textarea__inner {
  height: 100% !important;
  resize: none;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
}

.empty-prompts {
  text-align: center;
  padding: 60px 20px;
  color: #6b7280;
}

.empty-prompts .el-empty {
  padding: 20px;
}

/* 续写对话框样式 */
.continue-direction {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.direction-input {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.direction-input label {
  font-weight: 500;
  color: #2c3e50;
  font-size: 14px;
}

.direction-tips {
  background: #f8f9fa;
  border: 1px solid #e9ecef;
  border-radius: 6px;
  padding: 16px;
}

.direction-tips h4 {
  margin: 0 0 12px 0;
  color: #495057;
  font-size: 14px;
  font-weight: 500;
}

.direction-tips ul {
  margin: 0;
  padding-left: 20px;
}

.direction-tips li {
  color: #6c757d;
  font-size: 13px;
  line-height: 1.5;
  margin-bottom: 4px;
}

.direction-tips li:last-child {
  margin-bottom: 0;
}

/* 选段优化弹窗样式 */
.optimize-dialog {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.selected-content h4,
.optimize-direction h4,
.optimize-result h4 {
  margin: 0 0 8px 0;
  color: #2c3e50;
  font-size: 14px;
  font-weight: 600;
}

.selected-text {
  background: #f8f9fa;
  border: 1px solid #e9ecef;
  border-radius: 6px;
  padding: 12px;
  font-size: 14px;
  line-height: 1.6;
  color: #495057;
  max-height: 120px;
  overflow-y: auto;
}

.optimize-actions {
  text-align: center;
}

.optimized-text {
  background: #f0f9ff;
  border: 1px solid #bfdbfe;
  border-radius: 6px;
  padding: 12px;
  font-size: 14px;
  line-height: 1.6;
  color: #1e40af;
  max-height: 200px;
  overflow-y: auto;
  scroll-behavior: smooth;
}

.optimizing-placeholder {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #6b7280;
  font-style: italic;
}

.optimizing-placeholder .el-icon {
  font-size: 16px;
}

.optimized-content {
  white-space: pre-wrap;
  word-wrap: break-word;
}

.result-actions {
  display: flex;
  gap: 8px;
  justify-content: center;
  margin-top: 12px;
}

/* 续写弹窗样式 */
.continue-dialog .el-dialog__body {
  padding: 20px;
}

.continue-container {
  display: flex;
  gap: 20px;
  height: 500px;
}

.continue-config {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.config-section h4 {
  margin: 0 0 8px 0;
  color: #2c3e50;
  font-size: 14px;
  font-weight: 600;
}

.tips-list {
  margin: 0;
  padding-left: 20px;
  color: #6c757d;
  font-size: 13px;
  line-height: 1.5;
}

.tips-list li {
  margin-bottom: 4px;
}

.config-actions {
  margin-top: auto;
  display: flex;
  gap: 8px;
  justify-content: flex-end;
}

.continue-result {
  flex: 1;
  display: flex;
  flex-direction: column;
  border-left: 1px solid #e5e7eb;
  padding-left: 20px;
}

.result-header h4 {
  margin: 0 0 12px 0;
  color: #2c3e50;
  font-size: 14px;
  font-weight: 600;
}

.result-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.continuing-placeholder {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #6b7280;
  font-style: italic;
  padding: 20px;
  justify-content: center;
}

.continuing-placeholder .loading-icon {
  font-size: 16px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.continued-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.continued-text {
  background: #f0f9ff;
  border: 1px solid #bfdbfe;
  border-radius: 6px;
  padding: 12px;
  font-size: 14px;
  line-height: 1.6;
  color: #1e40af;
  flex: 1;
  overflow-y: auto;
  scroll-behavior: smooth;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.continuing-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
  margin-top: 8px;
  background: #e3f2fd;
  border-radius: 4px;
  font-size: 12px;
  color: #1976d2;
}

.continuing-indicator .loading-icon {
  margin-right: 4px;
  animation: spin 1s linear infinite;
}

.word-count-tips {
  margin-top: 4px;
  font-size: 12px;
  color: #6b7280;
  text-align: center;
}

.empty-placeholder {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.empty-placeholder .el-empty {
  padding: 20px;
}
</style>