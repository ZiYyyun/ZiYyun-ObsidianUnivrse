
### 项目架构
```
				┌──────────────────────────────-┐
				│        Application            │
				│  GPIO / MQTT / LLM / UI       │
				└──────────────▲───────────────-┘
				               │
				┌──────────────┴───────────────-┐
				│      Speech Engine            │
				│  ESP-SR (AFE/WakeNet/VAD)     │
				└──────────────▲───────────────-┘
				               │
				┌──────────────┴───────────────-┐
				│      Audio Pipeline           │
				│    PCM、Opus、重采样、缓存      │
				└──────────────▲──────────────-─┘
				               │
				┌──────────────┴──────-─────────┐
				│     Audio Hardware            │
				│ ES8311、I2S、DMA、Mic          │
				└──────────────────────-────────┘
```

### 工程目录

| 文件                  | 所属层                | 职责                                | 对应 ESP 官方组件                         |
| ------------------- | ------------------ | --------------------------------- | ----------------------------------- |
| `xiaozhi_audio`     | **Audio Hardware** | 管理 ES8311、I2S、PCM 输入输出            | `esp_codec_dev` + `esp_audio_codec` |
| `xiaozhi_sr`        | **Speech Engine**  | AFE、VAD、WakeNet、MultiNet，完成本地语音处理 | `esp-sr`                            |
| `xiaozhi_encoder`   | **Audio Pipeline** | PCM → Opus（或其他编码），为网络传输准备数据       | 编码库（不属于 `esp-sr`）                   |
| `app_main`（或其他业务模块） | **Application**    | 初始化各模块、联网、控制设备、与服务器交互             | 用户自己的应用逻辑                           |


![[AI小智.base]]
