# EN2CN 音频转换系统快速上手指南

## 1. 一键安装脚本

### 1.1 创建安装脚本 `install.ps1`
```powershell
# EN2CN 音频转换系统安装脚本
Write-Host "=== EN2CN 音频转换系统安装开始 ===" -ForegroundColor Green

# 检查Python
try {
    $pythonVersion = python --version
    Write-Host "检测到 Python: $pythonVersion" -ForegroundColor Yellow
} catch {
    Write-Host "错误: 请先安装 Python 3.8+" -ForegroundColor Red
    exit 1
}

# 创建虚拟环境
Write-Host "创建虚拟环境..." -ForegroundColor Yellow
python -m venv venv
if ($?) {
    Write-Host "虚拟环境创建成功" -ForegroundColor Green
} else {
    Write-Host "虚拟环境创建失败" -ForegroundColor Red
    exit 1
}

# 激活虚拟环境
Write-Host "激活虚拟环境..." -ForegroundColor Yellow
& "venv\Scripts\Activate.ps1"

# 升级pip
Write-Host "升级pip..." -ForegroundColor Yellow
python -m pip install --upgrade pip

# 安装PyTorch (CPU版本)
Write-Host "安装PyTorch..." -ForegroundColor Yellow
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cpu

# 安装基础依赖
Write-Host "安装基础依赖..." -ForegroundColor Yellow
pip install openai-whisper demucs pydub ffmpeg-python requests pyyaml click tqdm colorama

# 检查Ollama
Write-Host "检查Ollama..." -ForegroundColor Yellow
try {
    $ollamaVersion = ollama --version
    Write-Host "检测到 Ollama: $ollamaVersion" -ForegroundColor Green
} catch {
    Write-Host "警告: 未检测到Ollama，请手动安装" -ForegroundColor Yellow
    Write-Host "下载地址: https://ollama.com/download" -ForegroundColor Yellow
}

# 创建目录结构
Write-Host "创建目录结构..." -ForegroundColor Yellow
$dirs = @("data\input", "data\output", "data\temp", "data\logs", "config", "models")
foreach ($dir in $dirs) {
    New-Item -ItemType Directory -Path $dir -Force | Out-Null
}

Write-Host "=== 安装完成 ===" -ForegroundColor Green
Write-Host "接下来请:" -ForegroundColor Yellow
Write-Host "1. 安装并启动 Ollama" -ForegroundColor White
Write-Host "2. 运行: ollama pull qwen3:7b" -ForegroundColor White
Write-Host "3. 使用: python quick_start.py" -ForegroundColor White
```

### 1.2 创建快速启动脚本 `quick_start.py`
```python
#!/usr/bin/env python3
"""
EN2CN 音频转换系统快速启动脚本
"""
import os
import sys
import time
from pathlib import Path
import requests
import subprocess
import tempfile

def check_dependencies():
    """检查依赖"""
    print("🔍 检查依赖...")
    
    # 检查Python包
    required_packages = [
        'whisper', 'demucs', 'pydub', 'requests', 'yaml'
    ]
    
    missing_packages = []
    for package in required_packages:
        try:
            __import__(package)
            print(f"✅ {package}")
        except ImportError:
            missing_packages.append(package)
            print(f"❌ {package}")
    
    if missing_packages:
        print(f"❌ 缺少依赖: {', '.join(missing_packages)}")
        print("请运行: pip install -r requirements.txt")
        return False
    
    # 检查Ollama
    try:
        response = requests.get("http://localhost:11434/api/tags", timeout=5)
        if response.status_code == 200:
            models = response.json().get("models", [])
            qwen_models = [m for m in models if "qwen" in m.get("name", "").lower()]
            if qwen_models:
                print(f"✅ Ollama + Qwen: {qwen_models[0]['name']}")
            else:
                print("❌ Qwen模型未安装")
                print("请运行: ollama pull qwen3:7b")
                return False
        else:
            print("❌ Ollama服务不可用")
            return False
    except:
        print("❌ 无法连接到Ollama")
        print("请确保Ollama已启动")
        return False
    
    return True

def download_test_audio():
    """下载测试音频"""
    test_audio_url = "https://www.soundjay.com/misc/sounds/beep-07.wav"
    test_audio_path = Path("data/input/test_audio.wav")
    
    if not test_audio_path.exists():
        print("📥 下载测试音频...")
        try:
            # 创建一个简单的测试音频
            create_test_audio(str(test_audio_path))
            print("✅ 测试音频创建成功")
        except Exception as e:
            print(f"❌ 创建测试音频失败: {e}")
            return None
    
    return str(test_audio_path)

def create_test_audio(output_path):
    """创建测试音频"""
    from pydub import AudioSegment
    from pydub.generators import Sine
    
    # 创建一个简单的测试音频（5秒，440Hz正弦波）
    tone = Sine(440).to_audio_segment(duration=5000)
    tone.export(output_path, format="wav")

def simple_audio_process(input_path, output_path):
    """简单的音频处理演示"""
    print(f"🎵 处理音频: {input_path}")
    
    # 1. 音频分离（使用Demucs）
    print("1️⃣ 音频分离...")
    temp_dir = Path("data/temp")
    temp_dir.mkdir(exist_ok=True)
    
    try:
        # 运行Demucs
        cmd = [
            "python", "-m", "demucs.separate",
            "--two-stems=vocals",
            "--device=cpu",
            f"--out={temp_dir}",
            input_path
        ]
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        # 查找分离后的文件
        input_name = Path(input_path).stem
        vocals_path = temp_dir / "htdemucs" / input_name / "vocals.wav"
        
        if vocals_path.exists():
            print(f"✅ 人声分离成功: {vocals_path}")
        else:
            print("❌ 人声分离失败")
            return False
            
    except Exception as e:
        print(f"❌ 音频分离失败: {e}")
        return False
    
    # 2. 语音转录
    print("2️⃣ 语音转录...")
    try:
        import whisper
        model = whisper.load_model("base")  # 使用较小的模型
        result = model.transcribe(str(vocals_path))
        
        english_text = result["text"]
        print(f"📝 转录结果: {english_text}")
        
        if not english_text.strip():
            print("❌ 转录结果为空")
            return False
            
    except Exception as e:
        print(f"❌ 语音转录失败: {e}")
        return False
    
    # 3. 文本翻译
    print("3️⃣ 文本翻译...")
    try:
        chinese_text = translate_text(english_text)
        print(f"🀄 翻译结果: {chinese_text}")
        
        if not chinese_text.strip():
            print("❌ 翻译结果为空")
            return False
            
    except Exception as e:
        print(f"❌ 文本翻译失败: {e}")
        return False
    
    # 4. 保存结果
    print("4️⃣ 保存结果...")
    try:
        result_data = {
            "input_file": input_path,
            "english_text": english_text,
            "chinese_text": chinese_text,
            "vocals_path": str(vocals_path),
            "timestamp": time.strftime("%Y-%m-%d %H:%M:%S")
        }
        
        result_file = Path(output_path).with_suffix(".json")
        import json
        with open(result_file, 'w', encoding='utf-8') as f:
            json.dump(result_data, f, ensure_ascii=False, indent=2)
        
        print(f"✅ 结果保存到: {result_file}")
        return True
        
    except Exception as e:
        print(f"❌ 保存结果失败: {e}")
        return False

def translate_text(text):
    """翻译文本"""
    prompt = f"""请将以下英文文本翻译为中文，只输出翻译结果：

{text}"""
    
    payload = {
        "model": "qwen3:7b",
        "prompt": prompt,
        "temperature": 0.1,
        "stream": False
    }
    
    try:
        response = requests.post(
            "http://localhost:11434/api/generate",
            json=payload,
            timeout=30
        )
        
        if response.status_code == 200:
            result = response.json()
            return result.get("response", "").strip()
        else:
            raise Exception(f"API调用失败: {response.status_code}")
            
    except Exception as e:
        raise Exception(f"翻译失败: {e}")

def main():
    """主函数"""
    print("🚀 EN2CN 音频转换系统快速启动")
    print("=" * 50)
    
    # 检查依赖
    if not check_dependencies():
        print("❌ 依赖检查失败，请先安装所需依赖")
        sys.exit(1)
    
    print("✅ 所有依赖检查通过")
    print()
    
    # 获取输入文件
    input_path = input("请输入音频文件路径（回车使用测试音频）: ").strip()
    
    if not input_path:
        input_path = download_test_audio()
        if not input_path:
            print("❌ 无法创建测试音频")
            sys.exit(1)
    
    if not Path(input_path).exists():
        print(f"❌ 文件不存在: {input_path}")
        sys.exit(1)
    
    # 设置输出路径
    output_path = Path("data/output") / f"result_{int(time.time())}"
    output_path.parent.mkdir(exist_ok=True)
    
    print(f"📁 输出路径: {output_path}")
    print()
    
    # 开始处理
    print("🎬 开始处理...")
    success = simple_audio_process(input_path, str(output_path))
    
    if success:
        print()
        print("🎉 处理完成！")
        print(f"📊 结果文件: {output_path}.json")
    else:
        print()
        print("❌ 处理失败")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## 2. 使用步骤

### 2.1 环境安装
```powershell
# 1. 克隆或下载项目
git clone https://github.com/your-repo/EN2CN-Audio-Flow.git
cd EN2CN-Audio-Flow

# 2. 运行安装脚本
powershell -ExecutionPolicy Bypass -File install.ps1

# 3. 激活虚拟环境
venv\Scripts\activate

# 4. 安装Ollama和模型
# 下载Ollama: https://ollama.com/download
# 安装后运行:
ollama pull qwen3:7b
```

### 2.2 快速测试
```powershell
# 运行快速启动脚本
python quick_start.py

# 或者直接处理音频文件
python quick_start.py
# 输入音频文件路径，如: data/input/your_audio.wav
```

### 2.3 常见问题解决

#### 2.3.1 Ollama连接失败
```powershell
# 检查Ollama是否运行
ollama list

# 如果没有运行，启动Ollama
ollama serve

# 检查端口
netstat -an | findstr :11434
```

#### 2.3.2 模型下载失败
```powershell
# 手动下载模型
ollama pull qwen3:7b

# 检查已安装的模型
ollama list
```

#### 2.3.3 音频格式问题
```powershell
# 使用FFmpeg转换音频格式
ffmpeg -i input.mp3 -ar 16000 -ac 1 output.wav
```

## 3. 配置文件说明

### 3.1 最小配置 `config/simple.yaml`
```yaml
# 简化配置文件
models:
  # 语音转录
  whisper:
    model: "base"  # 使用较小的模型
    device: "cpu"
  
  # 文本翻译
  ollama:
    model: "qwen3:7b"
    host: "http://localhost:11434"
  
  # 音频分离
  demucs:
    model: "htdemucs"
    device: "cpu"

# 输出设置
output:
  format: "wav"
  sample_rate: 16000
  
# 临时文件
temp_dir: "./data/temp"
```

## 4. 常用命令

### 4.1 单文件处理
```powershell
# 处理单个音频文件
python quick_start.py

# 或者使用参数
python -c "
import sys
sys.path.append('.')
from quick_start import simple_audio_process
simple_audio_process('data/input/audio.wav', 'data/output/result')
"
```

### 4.2 批量处理
```powershell
# 批量处理目录下所有音频文件
python -c "
import os
from pathlib import Path
from quick_start import simple_audio_process

input_dir = Path('data/input')
output_dir = Path('data/output')
output_dir.mkdir(exist_ok=True)

for audio_file in input_dir.glob('*.wav'):
    print(f'处理: {audio_file}')
    output_path = output_dir / audio_file.stem
    simple_audio_process(str(audio_file), str(output_path))
"
```

## 5. 性能优化建议

### 5.1 GPU加速
```powershell
# 如果有NVIDIA GPU，安装CUDA版本的PyTorch
pip uninstall torch torchaudio
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cu118
```

### 5.2 模型选择
```yaml
# 根据硬件选择合适的模型
models:
  whisper:
    # 低端设备: "tiny", "base"
    # 中端设备: "small", "medium"  
    # 高端设备: "large", "large-v2"
    model: "base"
```

### 5.3 并行处理
```python
# 修改quick_start.py，添加并行处理
import concurrent.futures
from pathlib import Path

def batch_process_parallel(input_dir, output_dir, max_workers=2):
    """并行批量处理"""
    input_files = list(Path(input_dir).glob("*.wav"))
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = []
        for input_file in input_files:
            output_path = Path(output_dir) / input_file.stem
            future = executor.submit(simple_audio_process, str(input_file), str(output_path))
            futures.append(future)
        
        for future in concurrent.futures.as_completed(futures):
            try:
                result = future.result()
                print(f"处理完成: {result}")
            except Exception as e:
                print(f"处理失败: {e}")
```

## 6. 故障排除

### 6.1 常见错误
```powershell
# 错误1: 模块未找到
pip install -r requirements.txt

# 错误2: FFmpeg未安装
# Windows: choco install ffmpeg
# 或下载: https://ffmpeg.org/download.html

# 错误3: 显存不足
# 在配置文件中设置device: "cpu"

# 错误4: Ollama服务未启动
ollama serve
```

### 6.2 调试模式
```python
# 修改quick_start.py，启用调试模式
import logging
logging.basicConfig(level=logging.DEBUG)

# 或者在函数中添加更多日志
def simple_audio_process(input_path, output_path):
    print(f"Debug: 开始处理 {input_path}")
    # ... 处理逻辑
    print(f"Debug: 完成处理 {output_path}")
```

## 7. 下一步

完成快速测试后，您可以：

1. **扩展功能**: 参考完整的设计文档实现更多功能
2. **优化性能**: 根据硬件条件调整模型和参数
3. **集成应用**: 将核心功能集成到您的应用中
4. **定制化**: 根据特定需求修改翻译提示词和处理流程

---

*快速上手指南 v1.0*  
*更新日期: 2025年7月5日*
