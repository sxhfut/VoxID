# 🎤 VAD ASR 语音识别服务器

一个基于 Sherpa-ONNX 的高性能语音识别服务，支持实时 VAD（语音活动检测）、多语言识别和声纹识别。

> 📌 **Fork of [quyangminddock/VoxID](https://github.com/quyangminddock/VoxID).** All credit for the original work goes to the upstream author (`quyangminddock`). This fork is maintained under MAC-Lab and curated within the [Ubiquitous Psychological Computing](https://github.com/sxhfut/awesome-affective-computing) ecosystem. Issues and the canonical star history remain at the upstream repository.

## ✨ 特性

- 🌍 **多语言支持**：支持中文、英文、日文、韩文、粤语等多种语言
- 🎯 **智能语音检测**：内置 VAD 自动分段，过滤静音片段
- 🔊 **声纹识别**：支持说话人注册和识别
- ⚡ **实时通信**：基于 WebSocket 低延迟实时传输
- 📊 **健康监控**：提供健康检查、状态监控接口

## 📋 环境要求

### 基础要求
- **操作系统**：Linux / macOS / Windows
- **Go 版本**：1.21 或更高
- **内存**：建议 4GB+
- **磁盘**：至少 2GB 可用空间（用于存放模型文件）

### 依赖库（Linux）
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y libc++1 libc++abi1 build-essential

# CentOS/RHEL
sudo yum install -y libcxx libcxxabi gcc gcc-c++
```

## 🚀 快速开始

#### 1. 克隆项目
```bash
git clone https://github.com/sxhfut/VoxID.git
cd VoxID
```

#### 2. 安装 Go 依赖
```bash
go mod download
```

#### 3. 下载模型文件

> ⚠️ **重要**：本项目需要手动下载以下模型文件才能运行。

##### 3.1 下载 ASR 模型（必需）

**方式 A：使用 wget 下载**
```bash
# 创建目录
mkdir -p models/asr/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17

# 下载模型文件
wget -O models/asr/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17/model.int8.onnx \
  https://huggingface.co/csukuangfj/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17/resolve/main/model.int8.onnx

# 下载 tokens 文件
wget -O models/asr/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17/tokens.txt \
  https://huggingface.co/csukuangfj/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17/resolve/main/tokens.txt
```

**方式 B：使用 git-lfs 克隆（需要先安装 git-lfs）**
```bash
# 安装 git-lfs
sudo apt-get install git-lfs  # Ubuntu/Debian
# 或
brew install git-lfs          # macOS

# 初始化 git-lfs
git lfs install

# 克隆模型仓库
git clone https://huggingface.co/csukuangfj/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17 \
  models/asr/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17
```

**国内镜像加速**
如果 HuggingFace 下载速度慢，可以使用国内镜像：
```bash
# 使用 HF-Mirror 镜像站
export HF_ENDPOINT=https://hf-mirror.com

# 然后执行上面的下载命令
```

##### 3.2 下载 VAD 模型（必需）

Silero VAD 模型文件通常需要从项目中获取：
```bash
mkdir -p models/vad/silero_vad

# 下载 silero_vad.onnx
wget -O models/vad/silero_vad/silero_vad.onnx \
  https://github.com/snakers4/silero-vad/raw/master/files/silero_vad.onnx
```

##### 3.3 下载声纹识别模型（可选）

如果需要声纹识别功能：
```bash
mkdir -p models/speaker

# 下载声纹模型
wget -O models/speaker/3dspeaker_speech_campplus_sv_zh_en_16k-common_advanced.onnx \
  https://huggingface.co/csukuangfj/speaker-embedding-models/resolve/main/3dspeaker_speech_campplus_sv_zh_en_16k-common_advanced.onnx
```

如果不需要声纹识别，可以在 `config.json` 中禁用：
```json
{
  "speaker": {
    "enabled": false,
    ...
  }
}
```

#### 4. 配置动态库（Linux）

```bash
# 复制动态库到系统目录
sudo cp lib/*.so /usr/lib/
sudo cp lib/ten-vad/lib/Linux/x64/libten_vad.so /usr/lib/

# 或者设置 LD_LIBRARY_PATH（推荐）
export LD_LIBRARY_PATH=$PWD/lib:$PWD/lib/ten-vad/lib/Linux/x64:$LD_LIBRARY_PATH
```

#### 5. 创建必要目录
```bash
mkdir -p logs data/speaker
```

#### 6. 运行服务
```bash
# 方式 A：直接运行
go run main.go

# 方式 B：编译后运行
go build -o asr_server
./asr_server
```

#### 7. 验证服务
访问 http://localhost:8080/ 查看测试页面，点击"启动系统"按钮开始语音识别测试。

---

## ⚙️ 配置说明

主要配置文件：`config.json`

### VAD 配置

系统支持两种 VAD 引擎：

#### Silero VAD（默认）
```json
{
  "vad": {
    "provider": "silero_vad",
    "pool_size": 200,
    "threshold": 0.5,
    "silero_vad": {
      "model_path": "models/vad/silero_vad/silero_vad.onnx",
      "min_silence_duration": 0.1,
      "min_speech_duration": 0.25,
      "max_speech_duration": 8.0,
      "window_size": 512,
      "buffer_size_seconds": 10.0
    }
  }
}
```

#### Ten-VAD
```json
{
  "vad": {
    "provider": "ten_vad",
    "ten_vad": {
      "hop_size": 512,
      "min_speech_frames": 12,
      "max_silence_frames": 5
    }
  }
}
```

### ASR 配置
```json
{
  "recognition": {
    "model_path": "models/asr/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17/model.int8.onnx",
    "tokens_path": "models/asr/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17/tokens.txt",
    "language": "auto",
    "num_threads": 16,
    "provider": "cpu"
  }
}
```

### 声纹识别配置
```json
{
  "speaker": {
    "enabled": true,
    "model_path": "models/speaker/3dspeaker_speech_campplus_sv_zh_en_16k-common_advanced.onnx",
    "num_threads": 8,
    "threshold": 0.6,
    "data_dir": "data/speaker"
  }
}
```

### 服务器配置
```json
{
  "server": {
    "port": 8080,
    "host": "0.0.0.0",
    "read_timeout": 20
  }
}
```

更多配置选项请参考 `config.json` 文件。

---

## 🔌 API 使用

### WebSocket API

连接到 `ws://localhost:8080/ws`，发送音频数据（16kHz, 16-bit PCM）：

```javascript
const ws = new WebSocket('ws://localhost:8080/ws');

ws.onopen = () => {
  console.log('WebSocket 连接已建立');
  // 发送音频数据
  ws.send(audioBuffer);
};

ws.onmessage = (event) => {
  const result = JSON.parse(event.data);
  console.log('识别结果:', result);
};

ws.onerror = (error) => {
  console.error('WebSocket 错误:', error);
};

ws.onclose = () => {
  console.log('WebSocket 连接已关闭');
};
```

### HTTP API

#### 健康检查
```bash
curl http://localhost:8080/health
```

#### 状态监控
```bash
curl http://localhost:8080/stats
```

#### 声纹注册
```bash
curl -X POST http://localhost:8080/api/speaker/register \
  -H "Content-Type: application/json" \
  -d '{"speaker_id": "user123", "audio_data": "..."}'
```

#### 声纹识别
```bash
curl -X POST http://localhost:8080/api/speaker/recognize \
  -H "Content-Type: application/json" \
  -d '{"audio_data": "..."}'
```

---

## 🧪 测试

项目提供了测试脚本用于验证功能：

### 单文件测试
```bash
cd test/asr
python audiofile_test.py
```

### 并发压力测试
```bash
cd test/asr
python stress_test.py --connections 100 --audio-per-connection 2
```

参数说明：
- `--connections`: 并发连接数
- `--audio-per-connection`: 每个连接发送的音频文件数

---

## 🏛️ 系统架构

```
┌────────────────────┐    ┌──────────────────────┐    ┌────────────────────┐
│   WebSocket客户端   │    │   VAD语音活动检测池   │    │   ASR识别器模块     │
│                    │    │                      │    │ (动态new stream)   │
│  ┌──────────────┐  │    │  ┌──────────────┐    │    │  ┌──────────────┐  │
│  │  音频流输入   │◄─┼───►│  │   VAD实例    │◄──┼───►│  │ Recognizer   │  │
│  └──────────────┘  │    │  └──────────────┘    │    │  └──────────────┘  │
│  ┌──────────────┐  │    │  ┌──────────────┐    │    │                  │
│  │ 识别结果接收  │  │    │  │  缓冲队列    │    │    │                  │
│  └──────────────┘  │    │  └──────────────┘    │    └────────────────────┘
└────────────────────┘    └──────────────────────┘             │
                                                               ▼
┌────────────────────┐    ┌──────────────────────┐    ┌────────────────────┐
│   会话管理器       │    │   声纹识别模块(可选)  │    │   健康检查/监控    │
│  ┌──────────────┐  │    │  ┌──────────────┐    │    │                    │
│  │ 连接状态管理 │  │    │  │ 说话人注册   │    │    │  监控/状态接口     │
│  └──────────────┘  │    │  └──────────────┘    │    └────────────────────┘
│  ┌──────────────┐  │    │  ┌──────────────┐    │
│  │ 资源分配释放 │  │    │  │ 声纹特征提取 │    │
│  └──────────────┘  │    │  └──────────────┘    │
└────────────────────┘    └──────────────────────┘
```

---

## 📂 项目结构

```
VoxID/
├── main.go                 # 主程序入口
├── config.json             # 配置文件
├── go.mod                  # Go 依赖管理
├── go.sum
├── internal/               # 内部包
│   ├── bootstrap/          # 应用启动初始化
│   ├── logger/             # 日志模块
│   ├── router/             # 路由配置
│   └── ...
├── lib/                    # 动态链接库
│   └── ten-vad/
├── models/                 # 模型文件目录（需自行下载）
│   ├── asr/                # ASR 模型
│   ├── vad/                # VAD 模型
│   └── speaker/            # 声纹模型
├── static/                 # 静态资源
│   ├── index.html          # 测试页面
│   ├── css/
│   └── js/
├── data/                   # 数据存储
│   └── speaker/            # 声纹数据
├── logs/                   # 日志文件
└── test/                   # 测试脚本
    ├── asr/
    └── speaker/
```

---

## 🔧 常见问题

### 1. 模型文件下载失败

**问题**：从 HuggingFace 下载模型缓慢或失败

**解决方案**：
- 使用国内镜像：`export HF_ENDPOINT=https://hf-mirror.com`
- 使用代理下载
- 手动从浏览器下载后放到对应目录

### 2. 动态库加载失败

**问题**：运行时提示找不到 `.so` 文件

**解决方案**：
```bash
# 设置库路径
export LD_LIBRARY_PATH=$PWD/lib:$PWD/lib/ten-vad/lib/Linux/x64:$LD_LIBRARY_PATH

# 或复制到系统目录
sudo cp lib/*.so /usr/lib/
```

### 3. WebSocket 连接失败

**问题**：前端无法连接 WebSocket

**解决方案**：
- 检查防火墙设置，确保 8080 端口开放
- 检查 `config.json` 中的 `server.host` 配置
- 查看日志文件 `logs/app.log`

### 4. 识别结果为空

**问题**：音频发送成功但没有识别结果

**解决方案**：
- 确认音频格式：16kHz, 16-bit PCM
- 调整 VAD 参数（`threshold`、`min_speech_duration`）
- 检查音频是否包含有效语音

### 5. 内存占用过高

**问题**：服务运行一段时间后内存占用较高

**解决方案**：
- 调整 `vad.pool_size` 参数
- 减少 `pool.worker_count`
- 启用 `rate_limit` 限制并发连接数

---

## 📊 性能调优

### 关键参数

| 参数 | 说明 | 推荐值 | 影响 |
|------|------|--------|------|
| `vad.pool_size` | VAD 实例池大小 | 200 | 影响并发处理能力 |
| `recognition.num_threads` | ASR 线程数 | 8-16 | 影响识别速度 |
| `pool.worker_count` | 工作协程数 | 500 | 影响并发连接数 |
| `vad.threshold` | VAD 检测阈值 | 0.5 | 影响语音检测灵敏度 |
| `speaker.threshold` | 声纹相似度阈值 | 0.6 | 影响说话人识别准确度 |

### 优化建议

1. **CPU 优化**：根据 CPU 核心数调整 `num_threads`
2. **内存优化**：减少 `pool_size` 和 `worker_count`
3. **延迟优化**：使用 `ten_vad` 替代 `silero_vad`
4. **并发优化**：启用 `rate_limit` 防止过载

---

## 🤝 贡献指南

欢迎贡献代码！请遵循以下步骤：

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

---

## 📄 开源协议

本项目整体采用 **MIT 许可证**。但请注意：

- 如果使用 **ten-vad** 功能（`vad.provider` 设为 `ten_vad`），需遵守 [ten-vad 的 License](https://github.com/ten-framework/ten-vad/blob/main/LICENSE)
- 如果仅使用 **silero-vad**（`vad.provider` 设为 `silero_vad`），可直接遵循 MIT 许可证

请根据实际使用的 VAD 类型，遵守相应的开源协议。

---

## 🙏 致谢

本项目基于以下优秀的开源项目：

- [Sherpa-ONNX](https://github.com/k2-fsa/sherpa-onnx) - 核心语音识别引擎
- [SenseVoice](https://github.com/FunAudioLLM/SenseVoice) - 多语言语音识别模型
- [Silero VAD](https://github.com/snakers4/silero-vad) - 语音活动检测模型
- [ten-vad](https://github.com/ten-framework/ten-vad) - 高效端点检测算法
- [3D-Speaker](https://github.com/alibaba-damo-academy/3D-Speaker) - 声纹识别模型

---

## 📞 联系与支持

如有问题或建议，欢迎：

- 📝 提交 [Issue](https://github.com/quyangminddock/VoxID/issues)
- 💬 参与 [Discussions](https://github.com/quyangminddock/VoxID/discussions)
- 📧 发送邮件：bbeyond.llove@gmail.com

---

## ⭐ Star History

如果这个项目对你有帮助，欢迎给个 Star ⭐️！

[![Star History Chart](https://api.star-history.com/svg?repos=quyangminddock/VoxID&type=Date)](https://star-history.com/#quyangminddock/VoxID&Date)

---

<div align="center">
  <sub>Built with ❤️ by <a href="https://github.com/quyangminddock">quyangminddock</a></sub>
</div>