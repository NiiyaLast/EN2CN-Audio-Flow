# EN2CN 音频流转换系统

> 英文音频到中文音频的完整转换流水线

## 🎯 项目概述

本项目实现了一个完整的音频语言转换系统，将英文音频转换为中文音频，保持原始音色和背景音乐。系统采用高内聚低耦合的模块化架构，所有组件均可本地运行，确保数据隐私和系统独立性。

## 🏗️ 系统架构

```
输入音频 → 音频分离 → 语音转录 → 文本翻译 → 语音合成 → 音频合成 → 输出音频
    ↓         ↓         ↓         ↓         ↓         ↓
   WAV/MP3   人声+背景   英文文本   中文文本   中文语音   最终音频
```

## 🔧 核心技术栈

- **音频分离**: Demucs (Meta AI)
- **语音转录**: OpenAI Whisper
- **文本翻译**: Ollama + Qwen2.5
- **语音合成**: OpenVoice V2 / F5-TTS
- **音频处理**: FFmpeg + PyDub

## 📋 功能特性

- ✅ 高质量音频分离（人声/背景音）
- ✅ 准确的语音转录（英文转文本）
- ✅ 智能文本翻译（英文转中文）
- ✅ 零样本语音克隆
- ✅ 自然语音合成
- ✅ 完整音频重建

## 🚀 快速开始

### 环境要求
- Python 3.8+
- 8GB+ RAM
- 4GB+ 显存（推荐）
- Ollama 服务

### 安装步骤
```bash
# 克隆项目
git clone https://github.com/NiiyaLast/EN2CN-Audio-Flow.git
cd EN2CN-Audio-Flow

# 创建虚拟环境
python -m venv venv
venv\Scripts\activate  # Windows
# source venv/bin/activate  # Linux/Mac

# 安装依赖
pip install -r requirements.txt

# 安装并启动Ollama
# 下载: https://ollama.com/download
ollama pull qwen2.5:7b
```

### 基本使用
```bash
# 快速测试
python quick_start.py

# 处理音频文件
python quick_start.py --input "path/to/your/audio.wav"
```

## 📚 文档导航

- [📖 完整设计文档](EN2CN-Audio-Flow-Design-Document.md) - 基于第一性原理的详细系统设计
- [🚀 快速上手指南](Quick-Start-Guide.md) - 安装和使用的详细步骤
- [📁 项目结构总览](Project-Structure-Overview.md) - 项目架构和文件说明

## 🎨 设计理念

### 第一性原理
将复杂的音频转换问题分解为最基本的物理和数学问题：
- 音频 = 声波 = 数字信号
- 语音 = 语言信息 + 音色特征
- 翻译 = 语义映射 + 文化转换

### 高内聚低耦合
- 每个模块职责单一，功能完整
- 模块间通过标准接口通信
- 支持热插拔和独立测试

## 🛠️ 技术优势

- **本地运行**: 所有处理均在本地完成，保护数据隐私
- **模块化设计**: 易于维护、扩展和定制
- **高质量输出**: 保持原始音色和背景音乐
- **多模型支持**: 支持不同规模和类型的模型选择

## 🔄 工作流程

1. **音频分离**: 使用Demucs分离人声和背景音
2. **语音转录**: 使用Whisper将英文语音转为文本
3. **文本翻译**: 使用Ollama+Qwen2.5翻译为中文
4. **语音合成**: 使用OpenVoice V2克隆音色并合成中文语音
5. **音频合成**: 将中文语音与原背景音混合

## 🎯 应用场景

- 🎬 视频内容本地化
- 🎙️ 播客/音频内容翻译
- 🎵 音乐/有声书语言转换
- 🎓 教育内容多语言制作

## 🤝 贡献指南

欢迎贡献代码、报告问题或提出功能建议！

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 发起 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 🙏 致谢

感谢以下开源项目的支持：
- [OpenAI Whisper](https://github.com/openai/whisper)
- [Meta Demucs](https://github.com/facebookresearch/demucs)
- [MyShell OpenVoice](https://github.com/myshell-ai/OpenVoice)
- [Ollama](https://github.com/ollama/ollama)

## 📞 联系我们

- 项目主页: https://github.com/NiiyaLast/EN2CN-Audio-Flow
- 问题反馈: https://github.com/NiiyaLast/EN2CN-Audio-Flow/issues
- 功能请求: https://github.com/NiiyaLast/EN2CN-Audio-Flow/discussions

---

⭐ 如果这个项目对您有帮助，请给我们一个星标！

*最后更新: 2025年7月5日*
