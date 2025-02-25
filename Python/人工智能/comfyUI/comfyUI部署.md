# 流程
- 确认可以安装的cuda版本
- 确定可以安装的驱动版本
- 安装cuda（默认内部会带显卡驱动），如果装不上内部的显卡驱动，就手动安装驱动
- 安装conda（如果conda不存在）
- 下载模型
- 创建conda虚拟环境
- 安装comfy-cli
- 通过comfy-cli安装ComfyUI并启动

~~~shell
#!/bin/bash
# 设置工作目录
WORK_DIR="/app"
mkdir -p $WORK_DIR

cd $WORK_DIR
# 安装系统依赖
sudo apt-get update && sudo apt-get install -y \
    libgl1-mesa-dev \
    libglib2.0-0 \
    libgl1-mesa-glx \
    libglu1-mesa \
    git \
    libgl1 \
    wget \
    libglib2.0-0 \
    build-essential \
    gdebi-core \
    curl \
    libssl-dev
# 安装 Miniconda
wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p /opt/conda
rm ~/miniconda.sh
/opt/conda/bin/conda clean --all -y
# 配置 conda 环境
echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc
echo "conda activate base" >> ~/.bashrc
export PATH="/opt/conda/bin:$PATH"
# 配置 Conda 镜像源
conda config --system --add channels https://mirrors.nju.edu.cn/anaconda/pkgs/main/
conda config --system --add channels https://mirrors.nju.edu.cn/anaconda/pkgs/free/
conda config --system --set show_channel_urls yes
# 创建 Python 环境
conda create -n comfyui python=3.12 -y

# 配置 pip 镜像源
conda run -n comfyui pip config set global.index-url https://mirrors.nju.edu.cn/pypi/web/simple
# 安装 PyTorch
if [ ! -z "$http_proxy" ]; then
    conda run -n comfyui pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124 --proxy $http_proxy
else
    conda run -n comfyui pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
fi

# 安装额外依赖
conda run -n comfyui pip install --no-cache-dir "oss2" "PyOpenGL" "PyOpenGL-accelerate"
conda run -n comfyui pip install --no-cache-dir "modelscope" --force-reinstall
# 设置启动脚本权限
chmod +x start_service_docker.sh
# 设置使用指定那些GPU
export CUDA_VISIBLE_DEVICES=0
echo "Setup completed successfully!"

mkdir -p /app/comfyUI001/models


# 模型下载（例如到HuggingFace下载：https://hf-mirror.com/hakurei/waifu-diffusion-v1-4），模型下载到指定的相对路径（注意下载到comfyUI启动工作路径下的models目录中）
#comfy model download --url <URL> --relative-path <PATH>
comfy model download --url https://hf-mirror.com/hakurei/waifu-diffusion-v1-4/resolve/main/models/wd-1-3-5_80000-fp32.ckpt?download=true --relative-path ./comfyUI001/models/diffusion_models/waifu-diffusion-v1-4

# https://hf-mirror.com/hakurei/waifu-diffusion-v1-4/resolve/main/models/wd-1-3-5_80000-fp32.ckpt?download=true

# 模型移除
#comfy model remove --relative-path <PATH> --model-names <model names>

# 模型列表查看
#comfy model list --relative-path <PATH>

# 激活环境
source activate comfyui 

# comfyUI的安装启动(参考：https://github.com/Comfy-Org/comfy-cli?tab=readme-ov-file)，注意comfyUI的部署操作都是在上述conda创建的comfyui虚拟环境在执行
pip install comfy-cli
# 启用shell自动补全功能,这样可以在输入命令时获得提示
comfy --install-completion

# 安装comfyUI，--workspace参数指定安装的位置，安装位置目录不需要提前创建，在安装时会自动创建
comfy --workspace=/app/comfyUI001 install

# 启动comfyUI，默认情况下，ComfyUI 会从 --workspace 指定的路径（如 /app/comfyUI001）下的 models 文件夹加载模型，--background表示后台启动
comfy --workspace=/app/comfyUI001 launch -- --listen 0.0.0.0 --port 6006
comfy --workspace=/app/comfyUI001 launch --background -- --listen 0.0.0.0 --port 6006

~~~