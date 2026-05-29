<template>
  <div class="model-config-page">
    <!-- 页面标题 -->
    <div class="model-config-page__header">
      <h1 class="model-config-page__title">模型设置</h1>
      <p class="model-config-page__desc">配置 AI 模型，用于剧本生成和图片生成</p>
    </div>

    <!-- ============ 上半区：场景配置 ============ -->
    <section class="model-config-section">
      <div class="model-config-section__header">
        <h2 class="model-config-section__title">
          <Icon name="lucide:zap" size="20" />
          场景配置
        </h2>
        <p class="model-config-section__desc">为不同的创作场景绑定 AI 模型</p>
      </div>

      <div class="scene-cards">
        <!-- 剧本生成 -->
        <div class="scene-card">
          <div class="scene-card__icon scene-card__icon--text">
            <Icon name="lucide:file-text" size="28" />
          </div>
          <div class="scene-card__info">
            <h3 class="scene-card__title">剧本生成</h3>
            <p class="scene-card__desc">AI 对话、台本生成、分镜生成等文字类任务</p>
          </div>
          <div class="scene-card__action">
            <template v-if="scenes.script_gen">
              <div class="scene-card__bound">
                <span class="scene-card__bound-name">{{ scenes.script_gen.modelName }}</span>
                <span class="scene-card__bound-provider">{{ scenes.script_gen.modelProvider }}</span>
              </div>
              <div class="scene-card__btns">
                <button class="scene-card__change-btn" @click="openSceneBind('script_gen')">更换</button>
                <button class="scene-card__unbind-btn" @click="unbindScene('script_gen')">解绑</button>
              </div>
            </template>
            <template v-else>
              <span class="scene-card__empty">未绑定模型</span>
              <button class="scene-card__bind-btn" @click="openSceneBind('script_gen')">
                <Icon name="lucide:link" size="14" />
                绑定模型
              </button>
            </template>
          </div>
        </div>

        <!-- 图片生成 -->
        <div class="scene-card">
          <div class="scene-card__icon scene-card__icon--image">
            <Icon name="lucide:image" size="28" />
          </div>
          <div class="scene-card__info">
            <h3 class="scene-card__title">图片生成</h3>
            <p class="scene-card__desc">封面生成、角色头像、场景图等图片类任务</p>
          </div>
          <div class="scene-card__action">
            <template v-if="scenes.image_gen">
              <div class="scene-card__bound">
                <span class="scene-card__bound-name">{{ scenes.image_gen.modelName }}</span>
                <span class="scene-card__bound-provider">{{ scenes.image_gen.modelProvider }}</span>
              </div>
              <div class="scene-card__btns">
                <button class="scene-card__change-btn" @click="openSceneBind('image_gen')">更换</button>
                <button class="scene-card__unbind-btn" @click="unbindScene('image_gen')">解绑</button>
              </div>
            </template>
            <template v-else>
              <span class="scene-card__empty">未绑定模型</span>
              <button class="scene-card__bind-btn" @click="openSceneBind('image_gen')">
                <Icon name="lucide:link" size="14" />
                绑定模型
              </button>
            </template>
          </div>
        </div>
      </div>
    </section>

    <!-- ============ 下半区：模型管理 ============ -->
    <section class="model-config-section">
      <div class="model-config-section__header">
        <h2 class="model-config-section__title">
          <Icon name="lucide:brain" size="20" />
          模型管理
        </h2>
        <p class="model-config-section__desc">添加和管理你的 AI 模型</p>
      </div>

      <!-- Tab 切换 -->
      <div class="model-tabs">
        <button
          class="model-tabs__item"
          :class="{ 'model-tabs__item--active': activeTab === 'text' }"
          @click="activeTab = 'text'"
        >
          <Icon name="lucide:message-square" size="16" />
          文字模型
          <span v-if="textModels.length" class="model-tabs__count">{{ textModels.length }}</span>
        </button>
        <button
          class="model-tabs__item"
          :class="{ 'model-tabs__item--active': activeTab === 'image' }"
          @click="activeTab = 'image'"
        >
          <Icon name="lucide:image" size="16" />
          图片模型
          <span v-if="imageModels.length" class="model-tabs__count">{{ imageModels.length }}</span>
        </button>
      </div>

      <!-- 模型列表 -->
      <div class="model-grid">
        <div
          v-for="model in currentTabModels"
          :key="model.id"
          class="model-card"
          :class="{ 'model-card--inactive': !model.isActive }"
        >
          <div class="model-card__header">
            <div class="model-card__provider-badge" :class="`model-card__provider-badge--${model.provider}`">
              {{ providerLabel(model.provider) }}
            </div>
            <div class="model-card__actions">
              <button class="model-card__action-btn" title="编辑" @click="editModel(model)">
                <Icon name="lucide:pencil" size="14" />
              </button>
              <button class="model-card__action-btn model-card__action-btn--danger" title="删除" @click="confirmDeleteModel(model)">
                <Icon name="lucide:trash-2" size="14" />
              </button>
            </div>
          </div>
          <h4 class="model-card__name">{{ model.modelId }}</h4>
          <div class="model-card__meta">
            <span class="model-card__meta-item">
              <Icon name="lucide:globe" size="12" />
              {{ model.baseUrl }}
            </span>
            <span class="model-card__meta-item">
              <Icon name="lucide:key" size="12" />
              {{ model.apiKeyMask }}
            </span>
          </div>
          <div class="model-card__footer">
            <span class="model-card__status" :class="model.isActive ? 'model-card__status--active' : 'model-card__status--inactive'">
              {{ model.isActive ? '已启用' : '已停用' }}
            </span>
          </div>
        </div>

        <!-- 添加模型卡片 -->
        <div class="model-card model-card--add" @click="openAddModel">
          <Icon name="lucide:plus" size="32" class="model-card--add__icon" />
          <span class="model-card--add__text">添加模型</span>
          <span class="model-card--add__hint">支持 OpenAI 兼容协议</span>
        </div>
      </div>
    </section>

    <!-- ============ 添加/编辑模型弹窗 ============ -->
    <Teleport to="body">
      <Transition name="modal-fade">
        <div v-if="showModelDialog" class="modal-overlay" @click.self="closeModelDialog">
          <div class="modal-dialog">
            <div class="modal-dialog__header">
              <h3>{{ editingModel ? '编辑模型' : '添加模型' }}</h3>
              <button class="modal-dialog__close" @click="closeModelDialog">
                <Icon name="lucide:x" size="20" />
              </button>
            </div>

            <div class="modal-dialog__body">
              <!-- 提供商选择（按钮组） -->
              <div class="form-group">
                <label class="form-label">提供商</label>
                <div class="provider-select">
                  <button
                    v-for="p in providerOptions"
                    :key="p.value"
                    class="provider-select__btn"
                    :class="[
                      `provider-select__btn--${p.value}`,
                      { 'provider-select__btn--active': form.provider === p.value }
                    ]"
                    @click="selectProvider(p)"
                  >
                    {{ p.label }}
                  </button>
                </div>
              </div>

              <!-- 模型标识（Combobox：可输入 + 下拉选择） -->
              <div class="form-group">
                <label class="form-label">模型标识 <span class="form-required">*</span></label>
                <div class="model-combobox" ref="comboboxRef">
                  <div class="form-input-wrap">
                    <input
                      v-model="form.modelId"
                      class="form-input"
                      placeholder="输入或选择模型标识"
                      autocomplete="off"
                      @focus="showModelDropdown = true"
                      @input="showModelDropdown = true"
                    />
                    <button
                      class="form-input-wrap__toggle"
                      type="button"
                      @mousedown.prevent="showModelDropdown = !showModelDropdown"
                    >
                      <Icon :name="showModelDropdown ? 'lucide:chevron-up' : 'lucide:chevron-down'" size="16" />
                    </button>
                  </div>
                  <Transition name="dropdown-fade">
                    <div v-if="showModelDropdown && filteredModelOptions.length" class="model-dropdown">
                      <div
                        v-for="opt in filteredModelOptions"
                        :key="opt"
                        class="model-dropdown__item"
                        :class="{ 'model-dropdown__item--active': form.modelId === opt }"
                        @mousedown.prevent="pickModel(opt)"
                      >
                        {{ opt }}
                      </div>
                    </div>
                  </Transition>
                </div>
              </div>

              <div class="form-group">
                <label class="form-label">Base URL <span class="form-required">*</span></label>
                <input
                  v-model="form.baseUrl"
                  class="form-input"
                  placeholder="如：https://api.deepseek.com"
                  autocomplete="off"
                  data-1p-ignore
                  data-lpignore="true"
                  data-form-type="other"
                />
              </div>

              <div class="form-group">
                <label class="form-label">API Key <span class="form-required">*</span></label>
                <div class="form-input-wrap">
                  <input
                    v-model="form.apiKey"
                    class="form-input"
                    :class="{ 'form-input--masked': !showApiKey && form.apiKey }"
                    type="text"
                    placeholder="请输入 API Key"
                    autocomplete="off"
                    data-1p-ignore
                    data-lpignore="true"
                    data-form-type="other"
                  />
                  <button class="form-input-wrap__toggle" @click="showApiKey = !showApiKey" type="button">
                    <Icon :name="showApiKey ? 'lucide:eye-off' : 'lucide:eye'" size="16" />
                  </button>
                </div>
              </div>

              <!-- 模型类型切换 -->
              <div class="form-group">
                <label class="form-label">模型类型</label>
                <div class="type-toggle">
                  <button
                    class="type-toggle__btn"
                    :class="{ 'type-toggle__btn--active': form.type === 'text' }"
                    :disabled="!!editingModel"
                    @click="switchType('text')"
                  >
                    <Icon name="lucide:message-square" size="14" />
                    文字模型
                  </button>
                  <button
                    class="type-toggle__btn"
                    :class="{ 'type-toggle__btn--active': form.type === 'image' }"
                    :disabled="!!editingModel"
                    @click="switchType('image')"
                  >
                    <Icon name="lucide:image" size="14" />
                    图片模型
                  </button>
                </div>
              </div>
            </div>

            <div class="modal-dialog__footer">
              <button
                class="modal-btn modal-btn--outline"
                :class="{ 'modal-btn--success': tested }"
                :disabled="testLoading"
                @click="testModelConnection"
              >
                <Icon v-if="testLoading" name="lucide:loader-2" size="14" class="spin" />
                <Icon v-else-if="tested" name="lucide:check-circle" size="14" />
                <Icon v-else name="lucide:wifi" size="14" />
                {{ testLoading ? '测试中...' : tested ? '已通过' : '测试连接' }}
              </button>
              <div class="modal-dialog__footer-right">
                <button class="modal-btn modal-btn--ghost" @click="closeModelDialog">取消</button>
                <button class="modal-btn modal-btn--primary" :disabled="saving" @click="saveModel">
                  {{ saving ? '保存中...' : '保存' }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>

    <!-- ============ 场景绑定选择弹窗 ============ -->
    <Teleport to="body">
      <Transition name="modal-fade">
        <div v-if="showBindDialog" class="modal-overlay" @click.self="closeBindDialog">
          <div class="modal-dialog modal-dialog--sm">
            <div class="modal-dialog__header">
              <h3>选择模型</h3>
              <button class="modal-dialog__close" @click="closeBindDialog">
                <Icon name="lucide:x" size="20" />
              </button>
            </div>
            <div class="modal-dialog__body">
              <div v-if="bindableModels.length === 0" class="bind-empty">
                <Icon name="lucide:inbox" size="40" class="bind-empty__icon" />
                <p>暂无可用的{{ bindSceneCode === 'image_gen' ? '图片' : '文字' }}模型</p>
                <button class="modal-btn modal-btn--primary" @click="closeBindDialog(); openAddModel()">
                  去添加模型
                </button>
              </div>
              <div v-else class="bind-list">
                <div
                  v-for="model in bindableModels"
                  :key="model.id"
                  class="bind-list__item"
                  @click="bindSceneToModel(model.id)"
                >
                  <div class="bind-list__info">
                    <span class="bind-list__name">{{ model.modelId }}</span>
                    <span class="bind-list__meta">{{ providerLabel(model.provider) }}</span>
                  </div>
                  <Icon name="lucide:chevron-right" size="16" class="bind-list__arrow" />
                </div>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>

    <!-- ============ 删除确认弹窗 ============ -->
    <Teleport to="body">
      <Transition name="modal-fade">
        <div v-if="showDeleteConfirm" class="modal-overlay" @click.self="showDeleteConfirm = false">
          <div class="modal-dialog modal-dialog--sm">
            <div class="modal-dialog__header">
              <h3>确认删除</h3>
              <button class="modal-dialog__close" @click="showDeleteConfirm = false">
                <Icon name="lucide:x" size="20" />
              </button>
            </div>
            <div class="modal-dialog__body">
              <p>确定要删除模型「{{ deletingModel?.modelId }}」吗？绑定该模型的场景将自动解绑。</p>
            </div>
            <div class="modal-dialog__footer">
              <div class="modal-dialog__footer-right">
                <button class="modal-btn modal-btn--ghost" @click="showDeleteConfirm = false">取消</button>
                <button class="modal-btn modal-btn--danger" @click="doDeleteModel">确认删除</button>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup>
/**
 * 模型设置页面
 * 路由：/user/model-config
 * 渲染模式：CSR（需登录态）
 */
import { useUserStore } from '~/stores/user'

definePageMeta({ middleware: 'auth' })
useSeo()

const userStore = useUserStore()
const { get, post, put, del } = useApi()
const { showToast } = useToast()

/* ========== 数据状态 ========== */
const activeTab = ref('text')
const allModels = ref([])
const scenes = ref({})
const loading = ref(true)

const textModels = computed(() => allModels.value.filter(m => m.type === 'text'))
const imageModels = computed(() => allModels.value.filter(m => m.type === 'image'))
const currentTabModels = computed(() => activeTab.value === 'text' ? textModels.value : imageModels.value)

/* ========== 提供商 & 模型标识数据 ========== */
/* 预设 baseUrl / 模型 ID 以各平台文档为准；非 DeepSeek 时后端走 OpenAI 兼容通道（Azure 需替换 YOUR_RESOURCE） */
const providerOptions = [
  { value: 'deepseek', label: 'DeepSeek', baseUrl: 'https://api.deepseek.com' },
  { value: 'doubao', label: '豆包（火山方舟）', baseUrl: 'https://ark.cn-beijing.volces.com/api/v3' },
  { value: 'qwen', label: '通义千问（DashScope）', baseUrl: 'https://dashscope.aliyuncs.com/compatible-mode/v1' },
  { value: 'zhipu', label: '智谱 GLM', baseUrl: 'https://open.bigmodel.cn/api/paas/v4' },
  { value: 'moonshot', label: '月之暗面 Kimi', baseUrl: 'https://api.moonshot.cn/v1' },
  { value: 'minimax', label: 'MiniMax', baseUrl: 'https://api.minimax.chat/v1' },
  { value: 'siliconflow', label: '硅基流动 SiliconFlow', baseUrl: 'https://api.siliconflow.cn/v1' },
  { value: 'qianfan', label: '百度千帆（文心）', baseUrl: 'https://qianfan.baidubce.com/v2' },
  { value: 'openai', label: 'OpenAI', baseUrl: 'https://api.openai.com/v1' },
  { value: 'azure', label: 'Azure OpenAI', baseUrl: 'https://YOUR_RESOURCE.openai.azure.com/openai/v1' },
  { value: 'gemini', label: 'Google Gemini（兼容）', baseUrl: 'https://generativelanguage.googleapis.com/v1beta/openai' },
  { value: 'groq', label: 'Groq', baseUrl: 'https://api.groq.com/openai/v1' },
  { value: 'mistral', label: 'Mistral AI', baseUrl: 'https://api.mistral.ai/v1' },
  { value: 'xai', label: 'xAI Grok', baseUrl: 'https://api.x.ai/v1' },
  { value: 'together', label: 'Together AI', baseUrl: 'https://api.together.xyz/v1' },
  { value: 'fireworks', label: 'Fireworks AI', baseUrl: 'https://api.fireworks.ai/inference/v1' },
  { value: 'hunyuan', label: '腾讯混元', baseUrl: 'https://api.hunyuan.cloud.tencent.com/v1' },
  { value: 'stepfun', label: '阶跃星辰 StepFun', baseUrl: 'https://api.stepfun.com/v1' },
  { value: 'baichuan', label: '百川智能', baseUrl: 'https://api.baichuan-ai.com/v1' },
  { value: 'novita', label: 'Novita AI', baseUrl: 'https://api.novita.ai/v3/openai' },
  { value: 'perplexity', label: 'Perplexity', baseUrl: 'https://api.perplexity.ai' },
  { value: 'nebius', label: 'Nebius AI Studio', baseUrl: 'https://api.studio.nebius.ai/v1' },
  { value: 'openrouter', label: 'OpenRouter（聚合）', baseUrl: 'https://openrouter.ai/api/v1' },
  { value: 'anthropic', label: 'Claude（OpenRouter）', baseUrl: 'https://openrouter.ai/api/v1' },
  { value: 'custom', label: '自定义', baseUrl: '' },
]

/**
 * 各厂商主力模型快捷选项（文本 / 图像分开）
 * 豆包 / 方舟等为 Endpoint ID，以火山控制台为准；Azure 须将 baseUrl 中 YOUR_RESOURCE 换成实际资源名。
 * Anthropic 官方非 OpenAI 协议，此处与 OpenRouter 同域，模型 ID 使用路由前缀形式。
 * 另含：腾讯混元、阶跃星辰、百川、Novita、Perplexity、Nebius 等 OpenAI 兼容端点，具体模型名以各厂商控制台为准。
 */
const providerModels = {
  deepseek: {
    text: [
      'deepseek-chat',
      'deepseek-reasoner',
    ],
    image: [],
  },
  doubao: {
    text: [
      'doubao-seed-1-8-251228',
      'doubao-seed-1-6-251015',
      'doubao-seed-1-6-lite-251015',
      'doubao-seed-1-6-flash-250828',
    ],
    image: [
      'doubao-seedream-4-5-251128',
      'doubao-seedream-4-0-250828',
    ],
  },
  qwen: {
    text: [
      'qwen3.6-plus',
      'qwen3-max',
      'qwen3-coder-plus',
      'qwen-plus',
      'qwen-plus-latest',
      'qwen-turbo',
      'qwen-long',
      'qwen-max',
      'qwen-vl-max',
      'qwen-vl-plus',
    ],
    image: [
      'wanx2.1-t2i-turbo',
      'wanx-v1',
    ],
  },
  zhipu: {
    text: [
      'glm-4.7',
      'glm-4.7-flashx',
      'glm-4.7-flash',
      'glm-4.6',
      'glm-4.5-air',
      'glm-4.5-airx',
      'glm-4.5-flash',
      'glm-4-long',
      'glm-4-flashx-250414',
      'glm-4-flash-250414',
    ],
    image: [
      'cogview-4-plus',
      'cogview-4-250304',
      'cogview-4-flash',
    ],
  },
  moonshot: {
    text: [
      'kimi-k2-turbo-preview',
      'kimi-k2-0711-preview',
      'moonshot-v1-128k',
      'moonshot-v1-32k',
      'moonshot-v1-8k',
    ],
    image: [],
  },
  minimax: {
    text: [
      'MiniMax-M2',
      'MiniMax-Text-01',
      'abab6.5s-chat',
      'abab6.5g-chat',
      'abab6.5t-chat',
    ],
    image: [],
  },
  siliconflow: {
    text: [
      'deepseek-ai/DeepSeek-V3.2',
      'deepseek-ai/DeepSeek-R1',
      'Qwen/Qwen3-235B-A22B-Instruct-2507',
      'Qwen/Qwen3-30B-A3B-Instruct-2507',
      'meta-llama/Llama-3.3-70B-Instruct',
      'THUDM/GLM-Z1-32B-0414',
    ],
    image: [
      'black-forest-labs/FLUX.1-schnell',
      'Qwen/Qwen-Image',
    ],
  },
  qianfan: {
    text: [
      'ernie-4.5-turbo-128k',
      'ernie-4.5-8k-preview',
      'ernie-x1-turbo-32k',
      'ernie-x1-8k-preview',
      'ernie-4.0-turbo-8k',
      'ernie-speed-128k',
    ],
    image: [
      'irag-1.0.0',
    ],
  },
  openai: {
    text: [
      'gpt-5.1',
      'gpt-5.1-chat-latest',
      'gpt-5',
      'gpt-5-mini',
      'gpt-5-nano',
      'gpt-4.1',
      'gpt-4.1-mini',
      'gpt-4.1-nano',
      'gpt-4o',
      'gpt-4o-mini',
      'o3',
      'o4-mini',
    ],
    image: [
      'gpt-image-2',
      'gpt-image-2-2026-04-21',
      'gpt-image-1.5',
      'gpt-image-1',
      'gpt-image-1-mini',
      'dall-e-3',
    ],
  },
  azure: {
    text: [
      'gpt-4o',
      'gpt-4o-mini',
      'gpt-4.1',
      'gpt-4.1-mini',
      'gpt-35-turbo',
    ],
    image: [
      'gpt-image-2',
      'gpt-image-1.5',
      'gpt-image-1',
      'dall-e-3',
    ],
  },
  gemini: {
    text: [
      'gemini-3.5-pro',
      'gemini-3.5-flash',
      'gemini-3-pro-preview',
      'gemini-2.5-pro',
      'gemini-2.5-flash',
      'gemini-2.5-flash-lite',
      'gemini-2.0-flash',
      'gemini-2.0-flash-lite',
    ],
    image: [
      'gemini-3-pro-image-preview',
      'gemini-2.5-flash-image',
    ],
  },
  groq: {
    text: [
      'openai/gpt-oss-120b',
      'openai/gpt-oss-20b',
      'llama-3.3-70b-versatile',
      'llama-3.1-70b-versatile',
      'llama-3.1-8b-instant',
      'mixtral-8x7b-32768',
      'gemma2-9b-it',
    ],
    image: [],
  },
  mistral: {
    text: [
      'mistral-large-latest',
      'mistral-medium-latest',
      'mistral-small-latest',
      'ministral-8b-latest',
      'codestral-latest',
      'pixtral-large-latest',
    ],
    image: [],
  },
  xai: {
    text: [
      'grok-3',
      'grok-3-mini',
      'grok-2-1212',
      'grok-2-vision-1212',
    ],
    image: [],
  },
  together: {
    text: [
      'meta-llama/Llama-3.3-70B-Instruct-Turbo',
      'meta-llama/Llama-3.1-70B-Instruct-Turbo',
      'Qwen/Qwen2.5-72B-Instruct-Turbo',
      'deepseek-ai/DeepSeek-R1-Distill-Llama-70B',
    ],
    image: [],
  },
  fireworks: {
    text: [
      'accounts/fireworks/models/llama-v3p3-70b-instruct',
      'accounts/fireworks/models/deepseek-v3',
      'accounts/fireworks/models/qwen2p5-72b-instruct',
    ],
    image: [
      'accounts/fireworks/models/flux-1-dev-fp8',
    ],
  },
  hunyuan: {
    text: [
      'hunyuan-turbos-latest',
      'hunyuan-turbo',
      'hunyuan-t1-latest',
      'hunyuan-large',
      'hunyuan-standard-256K',
      'hunyuan-lite',
    ],
    image: [],
  },
  stepfun: {
    text: [
      'step-2-16k-exp',
      'step-2-mini',
      'step-1-8k',
      'step-1-32k',
      'step-1-flash',
    ],
    image: [],
  },
  baichuan: {
    text: [
      'Baichuan4-Turbo',
      'Baichuan4-Air',
      'Baichuan3-Turbo-128k',
      'Baichuan2-Turbo',
    ],
    image: [],
  },
  novita: {
    text: [
      'deepseek/deepseek_v3',
      'deepseek/deepseek-r1',
      'meta-llama/llama-3.3-70b-instruct',
      'Qwen/Qwen2.5-72B-Instruct',
      'mistralai/Mixtral-8x7B-Instruct-v0.1',
    ],
    image: [
      'black-forest-labs/FLUX.1-schnell',
    ],
  },
  perplexity: {
    text: [
      'sonar',
      'sonar-pro',
      'sonar-reasoning',
      'sonar-reasoning-pro',
    ],
    image: [],
  },
  nebius: {
    text: [
      'openai/gpt-oss-120b',
      'openai/gpt-oss-20b',
      'meta-llama/Llama-3.3-70B-Instruct',
      'Qwen/Qwen2.5-72B-Instruct',
      'deepseek-ai/DeepSeek-V3',
    ],
    image: [],
  },
  openrouter: {
    text: [
      'anthropic/claude-sonnet-4',
      'anthropic/claude-opus-4.1',
      'openai/gpt-4o',
      'openai/o3',
      'google/gemini-2.5-pro',
      'google/gemini-2.5-flash',
      'deepseek/deepseek-chat',
      'deepseek/deepseek-r1-0528',
      'meta-llama/llama-3.3-70b-instruct',
      'mistralai/mistral-large-2411',
      'x-ai/grok-3',
    ],
    image: [
      'google/gemini-2.5-flash-image-preview',
      'black-forest-labs/flux-1.1-pro',
    ],
  },
  anthropic: {
    text: [
      'anthropic/claude-opus-4.1',
      'anthropic/claude-sonnet-4',
      'anthropic/claude-3.7-sonnet',
      'anthropic/claude-3.5-sonnet',
      'anthropic/claude-3.5-haiku',
    ],
    image: [],
  },
  custom: { text: [], image: [] },
}

const providerLabel = (p) => {
  const map = {
    deepseek: 'DeepSeek',
    doubao: '豆包',
    qwen: '千问',
    zhipu: '智谱',
    moonshot: 'Kimi',
    minimax: 'MiniMax',
    siliconflow: '硅基流动',
    qianfan: '千帆',
    openai: 'OpenAI',
    azure: 'Azure',
    gemini: 'Gemini',
    groq: 'Groq',
    mistral: 'Mistral',
    xai: 'xAI',
    together: 'Together',
    fireworks: 'Fireworks',
    openrouter: 'OpenRouter',
    anthropic: 'Anthropic',
    hunyuan: '混元',
    stepfun: '阶跃',
    baichuan: '百川',
    novita: 'Novita',
    perplexity: 'Perplexity',
    nebius: 'Nebius',
    custom: '自定义',
  }
  return map[p] || p
}

const getProviderBaseUrl = (provider) => providerOptions.find(item => item.value === provider)?.baseUrl || ''

/* ========== 初始化加载 ========== */
onMounted(async () => {
  await Promise.all([fetchModels(), fetchScenes()])
  loading.value = false
})

const fetchModels = async () => {
  try {
    const res = await get('/api/model-config/models')
    if (res.code === 200) allModels.value = res.data || []
  } catch {}
}

const fetchScenes = async () => {
  try {
    const res = await get('/api/model-config/scenes')
    if (res.code === 200) scenes.value = res.data || {}
  } catch {}
}

/* ========== 添加/编辑模型弹窗 ========== */
const showModelDialog = ref(false)
const editingModel = ref(null)
const saving = ref(false)
const testLoading = ref(false)
const showApiKey = ref(false)
const tested = ref(false)

const defaultForm = () => ({
  type: activeTab.value,
  provider: 'deepseek',
  apiKey: '',
  baseUrl: getProviderBaseUrl('deepseek'),
  modelId: '',
})

const form = ref(defaultForm())

/** 关键字段变化时重置测试状态（弹窗未打开时不触发，避免初始化时误重置） */
const formWatchEnabled = ref(false)
watch(
  () => [form.value.apiKey, form.value.baseUrl, form.value.modelId],
  () => { if (formWatchEnabled.value) tested.value = false },
)

const openAddModel = () => {
  formWatchEnabled.value = false
  editingModel.value = null
  form.value = defaultForm()
  showApiKey.value = false
  tested.value = false
  showModelDropdown.value = false
  showModelDialog.value = true
  nextTick(() => { formWatchEnabled.value = true })
}

const editModel = (model) => {
  formWatchEnabled.value = false
  editingModel.value = model
  form.value = {
    type: model.type,
    provider: model.provider,
    apiKey: model.apiKey || '',
    baseUrl: model.baseUrl,
    modelId: model.modelId,
  }
  showApiKey.value = false
  tested.value = true
  showModelDropdown.value = false
  showModelDialog.value = true
  nextTick(() => { formWatchEnabled.value = true })
}

const closeModelDialog = () => {
  formWatchEnabled.value = false
  showModelDialog.value = false
  editingModel.value = null
  showModelDropdown.value = false
}

/** 选择提供商：自动填 baseUrl、清空 modelId */
const selectProvider = (p) => {
  form.value.provider = p.value
  form.value.baseUrl = getProviderBaseUrl(p.value)
  form.value.modelId = ''
}

/** 切换模型类型时清空 modelId（因为不同类型模型列表不同） */
const switchType = (type) => {
  if (editingModel.value) return
  form.value.type = type
  form.value.modelId = ''
}

/* ========== 模型标识 Combobox ========== */
const showModelDropdown = ref(false)
const comboboxRef = ref(null)

/** 点击 combobox 外部时关闭下拉 */
const handleDocClick = (e) => {
  if (comboboxRef.value && !comboboxRef.value.contains(e.target)) {
    showModelDropdown.value = false
  }
}
onMounted(() => document.addEventListener('mousedown', handleDocClick))
onUnmounted(() => document.removeEventListener('mousedown', handleDocClick))

/** 当前类型下可选的模型标识列表，按输入值模糊过滤 */
const filteredModelOptions = computed(() => {
  const type = form.value.type
  const provider = form.value.provider
  const keyword = (form.value.modelId || '').toLowerCase()

  let list = []
  if (provider === 'custom') {
    Object.keys(providerModels).forEach(k => {
      list.push(...(providerModels[k][type] || []))
    })
  } else {
    list = providerModels[provider]?.[type] || []
  }

  if (!keyword) return list
  return list.filter(m => m.toLowerCase().includes(keyword))
})

const pickModel = (modelId) => {
  form.value.modelId = modelId
  showModelDropdown.value = false
}

/* ========== 保存 & 测试 ========== */
const saveModel = async () => {
  if (!form.value.baseUrl || !form.value.modelId) {
    showToast('请填写模型标识和 Base URL', 'error')
    return
  }
  if (!form.value.apiKey) {
    showToast('请输入 API Key', 'error')
    return
  }
  if (!tested.value) {
    showToast('请先测试连接，确认模型可用后再保存', 'error')
    return
  }

  saving.value = true
  try {
    const payload = {
      type: form.value.type,
      provider: form.value.provider,
      baseUrl: form.value.baseUrl,
      modelId: form.value.modelId,
      apiKey: form.value.apiKey,
    }

    let res
    if (editingModel.value) {
      res = await put(`/api/model-config/models/${editingModel.value.id}`, payload)
    } else {
      res = await post('/api/model-config/models', payload)
    }

    if (res.code === 200) {
      showToast(editingModel.value ? '模型更新成功' : '模型添加成功', 'success')
      closeModelDialog()
      await fetchModels()
    } else {
      showToast(res.message || '操作失败', 'error')
    }
  } catch (err) {
    showToast(err.message || '操作失败', 'error')
  } finally {
    saving.value = false
  }
}

const testModelConnection = async () => {
  if (!form.value.baseUrl) {
    showToast('请先填写 Base URL', 'error')
    return
  }
  if (!form.value.modelId) {
    showToast('请填写模型标识', 'error')
    return
  }
  if (!form.value.apiKey) {
    showToast('请先填写 API Key', 'error')
    return
  }
  testLoading.value = true
  try {
    const res = await post('/api/model-config/test', {
      type: form.value.type,
      apiKey: form.value.apiKey,
      baseUrl: form.value.baseUrl,
      modelId: form.value.modelId,
    })
    if (res.code === 200 && res.data?.success) {
      tested.value = true
      showToast('连接成功！模型可用', 'success')
    } else {
      tested.value = false
      showToast(res.data?.message || '连接失败', 'error')
    }
  } catch (err) {
    tested.value = false
    showToast(err.message || '测试失败', 'error')
  } finally {
    testLoading.value = false
  }
}

/* ========== 删除模型 ========== */
const showDeleteConfirm = ref(false)
const deletingModel = ref(null)

const confirmDeleteModel = (model) => {
  deletingModel.value = model
  showDeleteConfirm.value = true
}

const doDeleteModel = async () => {
  if (!deletingModel.value) return
  try {
    const res = await del(`/api/model-config/models/${deletingModel.value.id}`)
    if (res.code === 200) {
      showToast('模型已删除', 'success')
      await Promise.all([fetchModels(), fetchScenes()])
    } else {
      showToast(res.message || '删除失败', 'error')
    }
  } catch (err) {
    showToast(err.message || '删除失败', 'error')
  } finally {
    showDeleteConfirm.value = false
    deletingModel.value = null
  }
}

/* ========== 场景绑定 ========== */
const showBindDialog = ref(false)
const bindSceneCode = ref('')

const bindableModels = computed(() => {
  const expectedType = bindSceneCode.value === 'image_gen' ? 'image' : 'text'
  return allModels.value.filter(m => m.type === expectedType && m.isActive)
})

const openSceneBind = (sceneCode) => {
  bindSceneCode.value = sceneCode
  showBindDialog.value = true
}

const closeBindDialog = () => {
  showBindDialog.value = false
  bindSceneCode.value = ''
}

const bindSceneToModel = async (modelId) => {
  try {
    const res = await post('/api/model-config/scenes/bind', {
      sceneCode: bindSceneCode.value,
      modelId,
    })
    if (res.code === 200) {
      showToast('绑定成功', 'success')
      closeBindDialog()
      await fetchScenes()
    } else {
      showToast(res.message || '绑定失败', 'error')
    }
  } catch (err) {
    showToast(err.message || '绑定失败', 'error')
  }
}

const unbindScene = async (sceneCode) => {
  try {
    const res = await post('/api/model-config/scenes/unbind', { sceneCode })
    if (res.code === 200) {
      showToast('已解绑', 'success')
      await fetchScenes()
    }
  } catch (err) {
    showToast(err.message || '操作失败', 'error')
  }
}
</script>

<style scoped>
/* ========== 页面容器 ========== */
.model-config-page {
  max-width: 960px;
  margin: 0 auto;
  padding: 8px 0 40px;
}

.model-config-page__header {
  margin-bottom: 28px;
}

.model-config-page__title {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--color-text);
  margin: 0 0 4px;
}

.model-config-page__desc {
  font-size: 0.85rem;
  color: var(--color-text-secondary);
  margin: 0;
}

/* ========== Section ========== */
.model-config-section {
  margin-bottom: 36px;
}

.model-config-section__header {
  margin-bottom: 16px;
}

.model-config-section__title {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 4px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.model-config-section__desc {
  font-size: 0.8rem;
  color: var(--color-text-secondary);
  margin: 0;
}

/* ========== 场景卡片 ========== */
.scene-cards {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.scene-card {
  background: var(--color-bg-white);
  border: 1px solid var(--color-border-light);
  border-radius: var(--radius);
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 14px;
  box-shadow: var(--shadow-sm);
  transition: box-shadow 0.2s;
}

.scene-card:hover {
  box-shadow: var(--shadow-lg);
}

.scene-card__icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.scene-card__icon--text {
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
}

.scene-card__icon--image {
  background: linear-gradient(135deg, #ec4899, #f43f5e);
}

.scene-card__title {
  font-size: 1rem;
  font-weight: 600;
  margin: 0;
  color: var(--color-text);
}

.scene-card__desc {
  font-size: 0.78rem;
  color: var(--color-text-secondary);
  margin: 0;
}

.scene-card__action {
  margin-top: auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.scene-card__bound {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.scene-card__bound-name {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--color-text);
}

.scene-card__bound-provider {
  font-size: 0.72rem;
  color: var(--color-text-light);
}

.scene-card__btns {
  display: flex;
  gap: 6px;
}

.scene-card__change-btn,
.scene-card__unbind-btn {
  padding: 4px 12px;
  font-size: 0.78rem;
  border-radius: 6px;
  font-weight: 500;
  transition: all 0.2s;
}

.scene-card__change-btn {
  color: var(--color-primary);
  background: var(--color-primary-bg);
}

.scene-card__change-btn:hover {
  background: rgba(255, 107, 53, 0.15);
}

.scene-card__unbind-btn {
  color: var(--color-text-secondary);
  background: var(--color-bg);
}

.scene-card__unbind-btn:hover {
  color: var(--color-danger);
  background: rgba(239, 68, 68, 0.08);
}

.scene-card__empty {
  font-size: 0.82rem;
  color: var(--color-text-light);
}

.scene-card__bind-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 14px;
  font-size: 0.8rem;
  color: #fff;
  background: var(--color-primary-gradient);
  border-radius: 8px;
  font-weight: 500;
  transition: opacity 0.2s;
}

.scene-card__bind-btn:hover {
  opacity: 0.9;
}

/* ========== Tab 切换 ========== */
.model-tabs {
  display: flex;
  gap: 4px;
  margin-bottom: 16px;
  background: var(--color-bg);
  border-radius: 10px;
  padding: 4px;
  width: fit-content;
}

.model-tabs__item {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 18px;
  font-size: 0.85rem;
  color: var(--color-text-secondary);
  background: transparent;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.2s;
}

.model-tabs__item:hover {
  color: var(--color-text);
}

.model-tabs__item--active {
  color: var(--color-text);
  background: var(--color-bg-white);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.model-tabs__count {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 18px;
  height: 18px;
  font-size: 0.7rem;
  background: var(--color-primary-bg);
  color: var(--color-primary);
  border-radius: 9px;
  padding: 0 5px;
  font-weight: 600;
}

/* ========== 模型卡片网格 ========== */
.model-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 14px;
}

.model-card {
  background: var(--color-bg-white);
  border: 1px solid var(--color-border-light);
  border-radius: var(--radius);
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  transition: all 0.2s;
}

.model-card:hover {
  box-shadow: var(--shadow-lg);
}

.model-card--inactive {
  opacity: 0.55;
}

.model-card__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.model-card__provider-badge {
  font-size: 0.7rem;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 6px;
  letter-spacing: 0.02em;
}

.model-card__provider-badge--deepseek { background: #e0f2fe; color: #0369a1; }
.model-card__provider-badge--doubao { background: #fef3c7; color: #92400e; }
.model-card__provider-badge--openai { background: #d1fae5; color: #065f46; }
.model-card__provider-badge--anthropic { background: #fce7f3; color: #9d174d; }
.model-card__provider-badge--zhipu { background: #ede9fe; color: #5b21b6; }
.model-card__provider-badge--qwen { background: #dbeafe; color: #1d4ed8; }
.model-card__provider-badge--gemini { background: #dcfce7; color: #166534; }
.model-card__provider-badge--moonshot { background: #fce7f3; color: #831843; }
.model-card__provider-badge--minimax { background: #e0e7ff; color: #3730a3; }
.model-card__provider-badge--siliconflow { background: #cffafe; color: #0e7490; }
.model-card__provider-badge--qianfan { background: #dbeafe; color: #1e40af; }
.model-card__provider-badge--azure { background: #e8f5ff; color: #0369a1; }
.model-card__provider-badge--groq { background: #fef9c3; color: #854d0e; }
.model-card__provider-badge--mistral { background: #ffedd5; color: #9a3412; }
.model-card__provider-badge--xai { background: #f3f4f6; color: #111827; }
.model-card__provider-badge--together { background: #ede9fe; color: #5b21b6; }
.model-card__provider-badge--fireworks { background: #fed7aa; color: #9a3412; }
.model-card__provider-badge--openrouter { background: #fce7f3; color: #9d174d; }
.model-card__provider-badge--hunyuan { background: #dbeafe; color: #1d4ed8; }
.model-card__provider-badge--stepfun { background: #d1fae5; color: #047857; }
.model-card__provider-badge--baichuan { background: #fef3c7; color: #b45309; }
.model-card__provider-badge--novita { background: #e0f2fe; color: #0369a1; }
.model-card__provider-badge--perplexity { background: #e0e7ff; color: #4338ca; }
.model-card__provider-badge--nebius { background: #ccfbf1; color: #0f766e; }
.model-card__provider-badge--custom { background: var(--color-bg); color: var(--color-text-secondary); }

.model-card__actions {
  display: flex;
  gap: 4px;
}

.model-card__action-btn {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  color: var(--color-text-secondary);
  background: transparent;
  transition: all 0.2s;
}

.model-card__action-btn:hover {
  background: var(--color-bg);
  color: var(--color-primary);
}

.model-card__action-btn--danger:hover {
  color: var(--color-danger);
  background: rgba(239, 68, 68, 0.08);
}

.model-card__name {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0;
}

.model-card__meta {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.model-card__meta-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.75rem;
  color: var(--color-text-light);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.model-card__footer {
  margin-top: auto;
}

.model-card__status {
  font-size: 0.72rem;
  font-weight: 500;
  padding: 2px 8px;
  border-radius: 4px;
}

.model-card__status--active {
  color: var(--color-success);
  background: rgba(34, 197, 94, 0.1);
}

.model-card__status--inactive {
  color: var(--color-text-light);
  background: var(--color-bg);
}

/* 添加模型卡片 */
.model-card--add {
  border: 2px dashed var(--color-border);
  align-items: center;
  justify-content: center;
  cursor: pointer;
  min-height: 160px;
  gap: 6px;
  transition: all 0.2s;
}

.model-card--add:hover {
  border-color: var(--color-primary);
  background: var(--color-primary-bg);
}

.model-card--add__icon {
  color: var(--color-text-light);
}

.model-card--add:hover .model-card--add__icon {
  color: var(--color-primary);
}

.model-card--add__text {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--color-text-secondary);
}

.model-card--add:hover .model-card--add__text {
  color: var(--color-primary);
}

.model-card--add__hint {
  font-size: 0.72rem;
  color: var(--color-text-light);
}

/* ========== 弹窗 ========== */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 20px;
}

.modal-dialog {
  background: var(--color-bg-white);
  border-radius: 16px;
  width: 100%;
  max-width: 500px;
  max-height: 85vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
}

.modal-dialog--sm {
  max-width: 420px;
}

.modal-dialog__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px 16px;
  border-bottom: 1px solid var(--color-border-light);
}

.modal-dialog__header h3 {
  font-size: 1.1rem;
  font-weight: 600;
  margin: 0;
}

.modal-dialog__close {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  color: var(--color-text-secondary);
  background: transparent;
  transition: all 0.2s;
}

.modal-dialog__close:hover {
  background: var(--color-bg);
}

.modal-dialog__body {
  padding: 20px 24px;
  overflow-y: auto;
  flex: 1;
}

.modal-dialog__footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
  border-top: 1px solid var(--color-border-light);
}

.modal-dialog__footer-right {
  display: flex;
  gap: 8px;
}

/* ========== 提供商选择按钮组 ========== */
.provider-select {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.provider-select__btn {
  padding: 5px 14px;
  font-size: 0.78rem;
  font-weight: 600;
  border-radius: 20px;
  border: 1.5px solid var(--color-border);
  background: var(--color-bg-white);
  color: var(--color-text-secondary);
  cursor: pointer;
  transition: all 0.2s;
}

.provider-select__btn:hover {
  border-color: var(--color-text-light);
  color: var(--color-text);
}

.provider-select__btn--active.provider-select__btn--deepseek { background: #e0f2fe; color: #0369a1; border-color: #7dd3fc; }
.provider-select__btn--active.provider-select__btn--doubao { background: #fef3c7; color: #92400e; border-color: #fcd34d; }
.provider-select__btn--active.provider-select__btn--openai { background: #d1fae5; color: #065f46; border-color: #6ee7b7; }
.provider-select__btn--active.provider-select__btn--qwen { background: #dbeafe; color: #1d4ed8; border-color: #93c5fd; }
.provider-select__btn--active.provider-select__btn--zhipu { background: #ede9fe; color: #5b21b6; border-color: #c4b5fd; }
.provider-select__btn--active.provider-select__btn--gemini { background: #dcfce7; color: #166534; border-color: #86efac; }
.provider-select__btn--active.provider-select__btn--anthropic { background: #fce7f3; color: #9d174d; border-color: #f9a8d4; }
.provider-select__btn--active.provider-select__btn--moonshot { background: #fce7f3; color: #831843; border-color: #f9a8d4; }
.provider-select__btn--active.provider-select__btn--minimax { background: #e0e7ff; color: #3730a3; border-color: #a5b4fc; }
.provider-select__btn--active.provider-select__btn--siliconflow { background: #cffafe; color: #0e7490; border-color: #22d3ee; }
.provider-select__btn--active.provider-select__btn--qianfan { background: #dbeafe; color: #1e40af; border-color: #60a5fa; }
.provider-select__btn--active.provider-select__btn--azure { background: #e8f5ff; color: #0369a1; border-color: #38bdf8; }
.provider-select__btn--active.provider-select__btn--groq { background: #fef9c3; color: #854d0e; border-color: #facc15; }
.provider-select__btn--active.provider-select__btn--mistral { background: #ffedd5; color: #9a3412; border-color: #fb923c; }
.provider-select__btn--active.provider-select__btn--xai { background: #f3f4f6; color: #111827; border-color: #9ca3af; }
.provider-select__btn--active.provider-select__btn--together { background: #ede9fe; color: #5b21b6; border-color: #c4b5fd; }
.provider-select__btn--active.provider-select__btn--fireworks { background: #fed7aa; color: #9a3412; border-color: #fdba74; }
.provider-select__btn--active.provider-select__btn--openrouter { background: #fce7f3; color: #9d174d; border-color: #f9a8d4; }
.provider-select__btn--active.provider-select__btn--hunyuan { background: #dbeafe; color: #1d4ed8; border-color: #60a5fa; }
.provider-select__btn--active.provider-select__btn--stepfun { background: #d1fae5; color: #047857; border-color: #34d399; }
.provider-select__btn--active.provider-select__btn--baichuan { background: #fef3c7; color: #b45309; border-color: #fcd34d; }
.provider-select__btn--active.provider-select__btn--novita { background: #e0f2fe; color: #0369a1; border-color: #7dd3fc; }
.provider-select__btn--active.provider-select__btn--perplexity { background: #e0e7ff; color: #4338ca; border-color: #a5b4fc; }
.provider-select__btn--active.provider-select__btn--nebius { background: #ccfbf1; color: #0f766e; border-color: #2dd4bf; }
.provider-select__btn--active.provider-select__btn--custom { background: var(--color-bg); color: var(--color-text); border-color: var(--color-text-secondary); }

/* ========== 模型标识 Combobox ========== */
.model-combobox {
  position: relative;
}

.model-dropdown {
  position: absolute;
  top: calc(100% + 4px);
  left: 0;
  right: 0;
  max-height: 220px;
  overflow-y: auto;
  background: var(--color-bg-white);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  z-index: 10;
  padding: 4px;
}

.model-dropdown__item {
  padding: 8px 12px;
  font-size: 0.84rem;
  color: var(--color-text);
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.15s;
  font-family: 'SF Mono', 'Cascadia Code', 'Menlo', monospace;
  letter-spacing: 0.01em;
}

.model-dropdown__item:hover {
  background: var(--color-bg-hover, var(--color-bg));
}

.model-dropdown__item--active {
  background: var(--color-primary-bg);
  color: var(--color-primary);
  font-weight: 600;
}

/* 下拉动画 */
.dropdown-fade-enter-active { transition: all 0.15s ease-out; }
.dropdown-fade-leave-active { transition: all 0.1s ease-in; }
.dropdown-fade-enter-from { opacity: 0; transform: translateY(-4px); }
.dropdown-fade-leave-to { opacity: 0; transform: translateY(-4px); }

/* ========== 模型类型切换 ========== */
.type-toggle {
  display: flex;
  gap: 0;
  background: var(--color-bg);
  border-radius: 8px;
  padding: 3px;
  width: fit-content;
}

.type-toggle__btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 16px;
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--color-text-secondary);
  background: transparent;
  border-radius: 6px;
  transition: all 0.2s;
}

.type-toggle__btn--active {
  color: var(--color-text);
  background: var(--color-bg-white);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.type-toggle__btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* ========== 表单 ========== */
.form-group {
  margin-bottom: 16px;
}

.form-label {
  display: block;
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--color-text);
  margin-bottom: 6px;
}

.form-required {
  color: var(--color-danger);
}

.form-input {
  width: 100%;
  padding: 9px 12px;
  font-size: 0.85rem;
  border: 1px solid var(--color-border);
  border-radius: 8px;
  outline: none;
  color: var(--color-text);
  background: var(--color-bg-white);
  transition: border-color 0.2s;
  font-family: inherit;
}

.form-input:focus {
  border-color: var(--color-primary);
}

/* API Key 遮罩：用 CSS 圆点替代 type=password，避免触发浏览器密码管理器 */
.form-input--masked {
  -webkit-text-security: disc;
}

.form-input-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.form-input-wrap .form-input {
  padding-right: 40px;
}

.form-input-wrap__toggle {
  position: absolute;
  right: 8px;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  color: var(--color-text-light);
  background: transparent;
  cursor: pointer;
  transition: color 0.2s;
}

.form-input-wrap__toggle:hover {
  color: var(--color-text);
}

.form-hint {
  font-size: 0.72rem;
  color: var(--color-text-light);
  margin-top: 4px;
  display: block;
}

/* ========== 按钮 ========== */
.modal-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 18px;
  font-size: 0.85rem;
  font-weight: 500;
  border-radius: 8px;
  transition: all 0.2s;
  font-family: inherit;
  white-space: nowrap;
}

.modal-btn--primary {
  color: #fff;
  background: var(--color-primary-gradient);
}

.modal-btn--primary:hover { opacity: 0.9; }
.modal-btn--primary:disabled { opacity: 0.5; cursor: not-allowed; }

.modal-btn--ghost {
  color: var(--color-text-secondary);
  background: var(--color-bg);
  border: 1px solid var(--color-border);
}

.modal-btn--ghost:hover { color: var(--color-text); border-color: var(--color-text-secondary); }

.modal-btn--outline {
  color: var(--color-primary);
  background: transparent;
  border: 1px solid var(--color-primary);
}

.modal-btn--outline:hover { background: var(--color-primary-bg); }
.modal-btn--outline:disabled { opacity: 0.5; cursor: not-allowed; }

.modal-btn--success {
  color: var(--color-success, #22c55e);
  border-color: var(--color-success, #22c55e);
}

.modal-btn--success:hover {
  background: rgba(34, 197, 94, 0.08);
}

.modal-btn--danger {
  color: #fff;
  background: var(--color-danger);
}

.modal-btn--danger:hover { opacity: 0.9; }

/* ========== 绑定弹窗 ========== */
.bind-empty {
  text-align: center;
  padding: 24px 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.bind-empty__icon {
  color: var(--color-text-light);
}

.bind-empty p {
  font-size: 0.85rem;
  color: var(--color-text-secondary);
  margin: 0;
}

.bind-list__item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 8px;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
}

.bind-list__item:hover {
  background: var(--color-bg-hover);
}

.bind-list__info {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.bind-list__name {
  font-size: 0.9rem;
  font-weight: 500;
  color: var(--color-text);
}

.bind-list__meta {
  font-size: 0.75rem;
  color: var(--color-text-light);
}

.bind-list__arrow {
  color: var(--color-text-light);
}

/* ========== 动画 ========== */
.modal-fade-enter-active { transition: all 0.25s ease-out; }
.modal-fade-leave-active { transition: all 0.2s ease-in; }
.modal-fade-enter-from { opacity: 0; }
.modal-fade-enter-from .modal-dialog { transform: scale(0.95) translateY(10px); }
.modal-fade-leave-to { opacity: 0; }
.modal-fade-leave-to .modal-dialog { transform: scale(0.98) translateY(5px); }

.spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

@media (max-width: 640px) {
  .scene-cards {
    grid-template-columns: 1fr;
  }

  .model-grid {
    grid-template-columns: 1fr;
  }
}
</style>
