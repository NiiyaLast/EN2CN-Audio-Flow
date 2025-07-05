# EN2CN 音频转换系统项目结构总览

## 📁 项目目录结构

```
EN2CN-Audio-Flow/
├── 📄 README.md                           # 项目说明（原始文档）
├── 📄 EN2CN-Audio-Flow-Design-Document.md # 完整设计文档
├── 📄 Quick-Start-Guide.md                # 快速上手指南
├── 📄 requirements.txt                    # Python依赖
├── 📄 setup.py                           # 项目安装脚本
├── 📄 install.ps1                        # Windows安装脚本
├── 📄 quick_start.py                     # 快速启动脚本
│
├── 📁 src/                               # 源代码
│   ├── 📁 core/                          # 核心模块
│   │   ├── 📄 __init__.py
│   │   ├── 📄 audio_separator.py         # 音频分离模块
│   │   ├── 📄 speech_to_text.py          # 语音转录模块
│   │   ├── 📄 text_translator.py         # 文本翻译模块
│   │   ├── 📄 text_to_speech.py          # 语音合成模块
│   │   ├── 📄 audio_mixer.py             # 音频合成模块
│   │   └── 📄 flow_controller.py         # 流程控制模块
│   │
│   ├── 📁 utils/                         # 工具模块
│   │   ├── 📄 __init__.py
│   │   ├── 📄 config_manager.py          # 配置管理
│   │   ├── 📄 file_handler.py            # 文件处理
│   │   ├── 📄 logger.py                  # 日志管理
│   │   └── 📄 quality_assessor.py        # 质量评估
│   │
│   ├── 📁 models/                        # 数据模型
│   │   ├── 📄 __init__.py
│   │   └── 📄 data_models.py             # 数据结构定义
│   │
│   └── 📁 cli/                           # 命令行界面
│       ├── 📄 __init__.py
│       └── 📄 main.py                    # 主命令行程序
│
├── 📁 config/                            # 配置文件
│   ├── 📄 default.yaml                   # 默认配置
│   ├── 📄 simple.yaml                    # 简化配置
│   └── 📁 prompts/                       # 提示词文件
│       ├── 📄 translation_prompt.txt     # 翻译提示词
│       └── 📄 system_prompt.txt          # 系统提示词
│
├── 📁 data/                              # 数据目录
│   ├── 📁 input/                         # 输入音频文件
│   ├── 📁 output/                        # 输出结果文件
│   ├── 📁 temp/                          # 临时文件
│   └── 📁 logs/                          # 日志文件
│
├── 📁 models/                            # 模型文件
│   ├── 📁 whisper/                       # Whisper模型
│   ├── 📁 demucs/                        # Demucs模型
│   └── 📁 openvoice/                     # OpenVoice模型
│
├── 📁 tests/                             # 测试文件
│   ├── 📄 test_audio_separator.py
│   ├── 📄 test_speech_to_text.py
│   ├── 📄 test_text_translator.py
│   └── 📄 test_integration.py
│
├── 📁 docs/                              # 文档目录
│   ├── 📄 API.md                         # API文档
│   ├── 📄 DEPLOYMENT.md                  # 部署指南
│   └── 📄 TROUBLESHOOTING.md             # 故障排除
│
└── 📁 examples/                          # 示例文件
    ├── 📄 basic_usage.py                 # 基础使用示例
    ├── 📄 batch_processing.py            # 批量处理示例
    └── 📄 advanced_usage.py              # 高级用法示例
```

## 📋 文件说明

### 📚 文档文件
- **README.md**: 原始需求文档，包含工具推荐和基本流程
- **EN2CN-Audio-Flow-Design-Document.md**: 完整的系统设计文档，基于第一性原理和高内聚低耦合设计
- **Quick-Start-Guide.md**: 快速上手指南，包含安装和使用说明

### 🔧 安装和启动
- **install.ps1**: Windows PowerShell安装脚本
- **quick_start.py**: 快速启动脚本，用于演示和测试
- **requirements.txt**: Python依赖包列表
- **setup.py**: 项目安装配置

### 💻 核心代码
- **src/core/**: 核心业务逻辑模块
  - 音频分离、语音转录、文本翻译、语音合成、音频混合
- **src/utils/**: 通用工具模块
  - 配置管理、文件处理、日志、质量评估
- **src/models/**: 数据模型定义
- **src/cli/**: 命令行界面

### ⚙️ 配置文件
- **config/default.yaml**: 完整的默认配置
- **config/simple.yaml**: 简化配置，适合快速测试
- **config/prompts/**: 翻译提示词文件

### 📊 数据目录
- **data/input/**: 输入音频文件存放目录
- **data/output/**: 处理结果输出目录
- **data/temp/**: 临时文件目录
- **data/logs/**: 日志文件目录

## 🚀 快速开始

### 1. 下载项目
```bash
git clone https://github.com/your-repo/EN2CN-Audio-Flow.git
cd EN2CN-Audio-Flow
```

### 2. 安装依赖
```powershell
# Windows
powershell -ExecutionPolicy Bypass -File install.ps1

# Linux/Mac
chmod +x install.sh
./install.sh
```

### 3. 启动Ollama
```bash
# 安装Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 下载模型
ollama pull qwen3:7b
```

### 4. 运行测试
```bash
python quick_start.py
```

## 📖 使用方式

### 🎯 快速测试
```bash
# 使用内置测试功能
python quick_start.py

# 处理指定文件
python quick_start.py --input path/to/audio.wav
```

### 🔧 API调用
```python
from src.core.flow_controller import FlowController
from src.utils.config_manager import ConfigManager

# 加载配置
config = ConfigManager("config/default.yaml")

# 创建流程控制器
controller = FlowController(config)

# 处理音频
result = controller.process_audio("input.wav", "output.wav")
```

### 📱 命令行使用
```bash
# 基础使用
python -m src.cli.main -i input.wav -o output.wav

# 批量处理
python -m src.cli.main -i input_dir/ -o output_dir/ --batch

# 自定义配置
python -m src.cli.main -i input.wav -o output.wav -c config/custom.yaml
```

## 🛠️ 开发指南

### 🔍 代码结构
- **高内聚**: 每个模块职责单一，内部功能紧密相关
- **低耦合**: 模块间通过标准接口通信，减少依赖
- **可扩展**: 支持新模型和算法的插件式扩展

### 🧪 测试
```bash
# 运行单元测试
python -m pytest tests/

# 运行集成测试
python -m pytest tests/test_integration.py

# 运行性能测试
python -m pytest tests/test_performance.py
```

### 📝 贡献
1. Fork项目
2. 创建特性分支
3. 提交更改
4. 发起Pull Request

## 🎯 主要特性

### ✅ 已实现功能
- 🎵 音频分离（人声/背景音）
- 🗣️ 语音转录（英文转文本）
- 🔄 文本翻译（英文转中文）
- 🎙️ 语音合成（中文文本转语音）
- 🎚️ 音频混合（合成最终音频）

### 🔧 技术栈
- **音频处理**: Demucs, FFmpeg, PyDub
- **语音识别**: OpenAI Whisper
- **文本翻译**: Ollama + Qwen3
- **语音合成**: OpenVoice V2 / F5-TTS
- **开发语言**: Python 3.8+

### 🌟 设计优势
- **本地运行**: 所有模型本地部署，保护隐私
- **模块化**: 高内聚低耦合，易于维护和扩展
- **可配置**: 丰富的配置选项，适应不同场景
- **易用性**: 提供GUI、CLI、API多种使用方式

## 📞 支持和反馈

### 📧 联系方式
- **项目主页**: https://github.com/your-repo/EN2CN-Audio-Flow
- **问题反馈**: https://github.com/your-repo/EN2CN-Audio-Flow/issues
- **文档更新**: https://github.com/your-repo/EN2CN-Audio-Flow/wiki

### 🤝 社区
- **讨论区**: https://github.com/your-repo/EN2CN-Audio-Flow/discussions
- **更新日志**: https://github.com/your-repo/EN2CN-Audio-Flow/releases

---

*项目结构总览 v1.0*  
*更新时间: 2025年7月5日*
