# 图像处理

## Roop One Pic Swap

### 安装

1. Install [CUDA Toolkit 11.8](https://developer.nvidia.com/cuda-11-8-0-download-archive) and [cuDNN for Cuda 11.x](https://developer.nvidia.com/rdp/cudnn-archive)

2. Install dependencies:

   ```bash
   pip uninstall onnxruntime onnxruntime-gpu
   
   pip install onnxruntime-gpu==1.15.1
   ```

3. Usage in case the provider is available

```bash
python run.py --execution-provider cuda --execution-threads 3
```

`cuDNN`要拷贝到`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8`目录下，并设置好环境变量。

[Install CUDA and CUDNN on Windows & Linux | by Techzizou | Geek Culture | Medium](https://medium.com/geekculture/install-cuda-and-cudnn-on-windows-linux-52d1501a8805)

如果报`zlibwapi.dll`问题，从`"C:\Program Files\NVIDIA Corporation\Nsight Systems 2022.4.2\host-windows-x64\zlib.dll"`拷贝`zlib.dll`放到`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\bin`下并重命名为`zlibwapi.dll`