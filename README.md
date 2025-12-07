
````markdown
# 📘 ByteTrack 安裝與環境建置手冊（macOS 範例）

---

> **適用平台：** macOS (Intel / Apple Silicon)  
> **適用版本：** Python 3.10–3.11  
> **目的：** 復刻 ByteTrack 論文環境，並建立可執行 YOLOX 追蹤實驗之環境。

---

## 一、建立虛擬環境

為避免與系統套件衝突，建議使用 Python venv 建立獨立環境：

```bash
cd /Users/hans/Documents/vscode
python3 -m venv bytetrack_env
````

啟用虛擬環境：

```bash
source bytetrack_env/bin/activate
```

> 成功後，終端機前方會出現 `(bytetrack_env)`。

---

## 二、下載 ByteTrack 原始碼

```bash
git clone https://github.com/ifzhang/ByteTrack.git
cd ByteTrack
```

確認路徑：

```bash
pwd
# /Users/hans/Documents/vscode/ByteTrack
```

---

## 三、安裝必要依賴套件

### 1️⃣ 安裝核心版本（避免不相容）

```bash
pip install numpy==1.24.4
pip install onnxruntime-silicon  # macOS Apple Silicon 專用
# 若為 Windows / Linux GPU，改用：
# pip install onnxruntime-gpu==1.17.0
pip install filterpy cython blosc2==2.0.0 FuzzyTM
```

### 2️⃣ 更新工具鏈

```bash
pip install --upgrade pip setuptools wheel
```

### 3️⃣ 強制安裝相容版本 (防止 numpy / scipy 衝突)

```bash
pip install --force-reinstall "numpy==1.24.4" "scipy==1.10.1"
python3 -c "import numpy, scipy; print(numpy.__version__, scipy.__version__)"
# → 顯示：1.24.4 1.10.1
```

---

## 四、安裝 ByteTrack 相依套件

```bash
pip install -r requirements.txt --no-deps
```

> `--no-deps` 參數可避免 pip 自動升級套件造成版本衝突。

---

## 五、安裝 ByteTrack 套件本體（開發模式）

由於 `setup.py develop` 已被棄用，建議改用 PEP660 方式：

```bash
pip install -e . --no-build-isolation
```

> `--no-build-isolation` 可確保使用當前虛擬環境中的 torch 與依賴。
> 若仍失敗，可改用：
>
> ```bash
> pip install .
> ```

驗證安裝是否成功：

```bash
python3 -c "import yolox; print('✅ ByteTrack setup OK')"
```

---

## 六、安裝 COCO API（資料集支援）

### 1️⃣ 安裝必要套件

```bash
pip install numpy cython setuptools wheel
```

### 2️⃣ 從 GitHub 安裝 COCO API

```bash
pip install 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI' --no-build-isolation
```

### 3️⃣ 驗證安裝結果

```bash
python3 -c "import pycocotools; print('✅ COCO API installed OK')"
```

---

## 七、額外套件（Bounding Box 工具）

安裝 cython_bbox：

```bash
pip3 install cython_bbox
```

---

## 八、環境版本檢查

建議紀錄環境狀態以便未來復刻：

```bash
pip freeze > requirements_freeze.txt
```

> 可於任何主機上使用：
>
> ```bash
> pip install -r requirements_freeze.txt
> ```
>
> 即可重建相同環境。

---

## 九、常見錯誤排除對應表

| 錯誤訊息                           | 原因                          | 解法                                         |
| ------------------------------ | --------------------------- | ------------------------------------------ |
| No module named 'torch'        | pip 子環境未載入 torch            | 使用 `pip install -e . --no-build-isolation` |
| Failed building wheel for onnx | macOS 無預編譯 wheel            | 改用 `onnxruntime-silicon`                   |
| numpy 2.x incompatible         | pip 自動升級版本過新                | 降回 `numpy==1.24.4`                         |
| clang error during COCO build  | 缺少 Xcode Command Line Tools | 執行 `xcode-select --install`                |
| scipy>=1.11 required           | 舊套件要求不一                     | 可忽略或手動鎖定 `scipy==1.10.1`                   |

---

## 十、下載模型權重並放入 `pretrained/`

建立權重資料夾並進入：

```bash
cd pretrained
```

下載 **ByteTrack Tiny 模型權重**：

```bash
curl -L -o bytetrack_tiny_mot17.pth.tar \
https://github.com/ifzhang/ByteTrack/releases/download/v0.1.3/bytetrack_tiny_mot17.tar
```

確認權重是否下載成功：

```bash
ls -lh
# 應該看到：
# -rw-r--r--  bytetrack_tiny_mot17.pth.tar  (約 25MB)
```

> 📂 **完整路徑示例：**
> `/Users/hans/Documents/vscode/ByteTrack/pretrained/bytetrack_tiny_mot17.pth.tar`

---

## 十一、修正 NumPy 相容性問題

由於新版 NumPy (>=1.24) 移除了 `np.float`，
需手動在 ByteTrack 原始碼中加入相容性修正。

編輯檔案：

```
/Users/hans/Documents/vscode/ByteTrack/yolox/tracker/byte_tracker.py
```

在最上方（`import numpy as np` 之後）加入以下程式：

```python
import numpy as np
np.float = float
```

> ✅ **說明：**
> 這行會在執行時重新定義 `np.float`，
> 以確保舊版程式碼能在 NumPy 1.24+ 上正常執行。

---

## 十二、執行 Tiny 模型範例（CPU 測試）

確認權重與程式修正後，執行下列指令：

```bash
python3 tools/demo_track.py video --device cpu \
  -f ./exps/example/mot/yolox_tiny_mix_det.py \
  -c ./pretrained/bytetrack_tiny_mot17.pth.tar \
  --path ./videos/palace.mp4 --save_result
```

---

### ✅ 預期輸出

執行時會顯示：

```
2025-11-09 16:58:02 | INFO | __main__:main:326 - Model Summary: Params: 5.03M, Gflops: 24.60
2025-11-09 16:58:02 | INFO | __main__:main:338 - loaded checkpoint done.
2025-11-09 16:58:02 | INFO | imageflow_demo:248 - video save_path is ./YOLOX_outputs/yolox_tiny_mix_det/track_vis/2025_11_09_16_58_02/palace.mp4
```

> 📸 **輸出結果：**
>
> * 影片檔案儲存於：
>
>   ```
>   ./YOLOX_outputs/yolox_tiny_mix_det/track_vis/<日期時間>/palace.mp4
>   ```
> * 影片中可見移動中的人物、車輛等物件
>   並以顏色框標示追蹤軌跡與編號。

---

## 十三、驗證輸出結果

列出輸出目錄：

```bash
ls YOLOX_outputs/yolox_tiny_mix_det/track_vis/
```

打開影片檢視結果（例如在 macOS Finder 直接開啟）：

```bash
open ./YOLOX_outputs/yolox_tiny_mix_det/track_vis/2025_11_09_16_58_02/palace.mp4
```

影片中應能看到：

* 每位移動目標以不同顏色的框追蹤；
* 螢幕左上角顯示追蹤 FPS；
* 左下角顯示偵測目標數量。

---

## 十四、執行完成驗證與手冊附註

若可成功輸出影片，即代表：

✅ Python 環境配置正確
✅ Torch、ONNX、COCO API 均安裝完成
✅ 模型可載入並正確執行
✅ ByteTrack CPU 模式可於 macOS 執行無誤

---

## 📄 附註 MAC（GPU 模式）

若要進一步執行 GPU 模式（於 Linux / Docker 上），
可替換成：
--device cpu change --device mps
```bash
python3 tools/demo_track.py video --device mps \
  -f ./exps/example/mot/yolox_tiny_mix_det.py \
  -c ./pretrained/bytetrack_tiny_mot17.pth.tar \
  --path ./videos/palace.mp4 --save_result

```
可用此測試ＧＰＵ是否開啟
```
python3 - <<'EOF'
import torch
print("PyTorch 版本:", torch.__version__)
print("是否支援 MPS:", torch.backends.mps.is_available())
print("是否可用:", torch.backends.mps.is_built())
EOF

```

---
下面是我幫你整理好的 **ByteTrack GPU 部署完整 Markdown 文件**，
可以直接給教授、放 GitHub 或做你的專案紀錄。

---

# 🚀 ByteTrack (YOLOX + ByteTrack) — GPU Deployment Guide (CUDA 12.2 / PyTorch 2.1 / Ubuntu)

本文件記錄從 **乾淨 VM → RTX 3080 + CUDA 12.2 → ByteTrack 可成功 GPU 推論**
的完整部署流程，含所有踩雷修復紀錄。

---

# 🖥️ 1. 系統環境

| 項目     | 內容                      |
| ------ | ----------------------- |
| GPU    | NVIDIA GeForce RTX 3080 |
| Driver | 535.161.07              |
| CUDA   | 12.2（driver API）        |
| Python | 3.10                    |
| OS     | Ubuntu (cloud VM)       |

確認 GPU：

```bash
nvidia-smi
```

---

# 🧱 2. 建立虛擬環境

```bash
cd ~/vscode
python3 -m venv bytetrack_env
source bytetrack_env/bin/activate
```

---

# 📦 3. 安裝基礎依賴（compiler / cmake）

YOLOX 需要編譯 C++/CUDA extension：

```bash
sudo apt update
sudo apt install -y build-essential gcc g++ make python3-dev cmake
```

---

# 🔥 4. 安裝 PyTorch CUDA 12.1（支援 Driver 535）

CUDA 12.2 driver 只能使用 cu121 版本 wheel：

```bash
pip uninstall -y torch torchvision torchaudio
pip cache purge

pip install torch==2.1.0+cu121 torchvision==0.16.0+cu121 \
  --index-url https://download.pytorch.org/whl/cu121
```

確認：

```bash
python3 - << 'PY'
import torch
print("Torch:", torch.__version__)
print("CUDA available:", torch.cuda.is_available())
print("GPU:", torch.cuda.get_device_name(0))
PY
```

---

# 📦 5. 修復 NumPy / OpenCV 相容性

⚠️ NumPy 2.x 與 OpenCV、YOLOX 不相容，因此要降級：

```bash
pip uninstall -y numpy opencv-python opencv-python-headless
pip install numpy==1.26.4
pip install opencv-python-headless==4.8.1.78
```

測試：

```bash
python3 - << 'PY'
import numpy, cv2
print("numpy:", numpy.__version__)
print("opencv:", cv2.__version__)
PY
```

---

# 📚 6. 安裝 ByteTrack 依賴（修正版 requirements）

不要使用舊 repo 內 onnx==1.8.1（無法支援 Python 3.10）

先編輯 requirements.txt：

```txt
numpy==1.26.4
opencv-python-headless==4.8.1.78
torch>=2.0
torchvision
loguru
scikit-image
tqdm
Pillow
thop
ninja
tabulate
tensorboard
lap
motmetrics
filterpy
h5py

onnx==1.12.0
onnxruntime==1.14.1
onnx-simplifier==0.4.10
```

安裝：

```bash
pip install -r requirements.txt --no-deps
pip install coloredlogs flatbuffers packaging
pip install -U scipy
pip install ninja
```

---

# 🧩 7. 安裝 COCO API（ByteTrack 需要）

```bash
pip install cython_bbox
pip install 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI' --no-build-isolation
```

---

# 🏗️ 8. 安裝 ByteTrack 本體（build YOLOX kernel）

```bash
cd ~/vscode/ByteTrack
pip install -e . --no-build-isolation
```

測試：

```bash
python3 -c "import yolox; print('✅ ByteTrack setup OK')"
```

---

# 📥 9. 下載預訓練模型（正確版 25MB）

⚠️ 你之前下載到的是 9 bytes 的壞檔，此為正確連結：

```bash
mkdir -p pretrained
cd pretrained

wget https://github.com/ifzhang/ByteTrack/releases/download/0.1.1/bytetrack_tiny_mot17.pth.tar
```

確認：

```bash
ls -lh
# => 應該約 22–25 MB
```

---

# 🎬 10. 執行 ByteTrack Demo（GPU 推論）

> 🔥 CPU 會非常非常慢，所以務必用 GPU。

```bash
cd ~/vscode/ByteTrack
python3 tools/demo_track.py video --device gpu \
  -f ./exps/example/mot/yolox_tiny_mix_det.py \
  -c ./pretrained/bytetrack_tiny_mot17.pth.tar \
  --path ./videos/palace.mp4 --save_result
```

輸出會在：

```
./YOLOX_outputs/yolox_tiny_mix_det/track_vis/YYYY_MM_DD_HH_MM_SS/palace.mp4
```

---

# 🎉 11. 你成功了！

GPU 已成功推論 YOLOX + ByteTrack。
整個流程包含：

* CUDA + PyTorch 相容性修復
* NumPy / OpenCV ABI 衝突修復
* YOLOX C++ kernel build
* ByteTrack 依賴調整
* 壞掉的模型檔重新下載
* CPU 卡住 → 改 GPU 正常運行

這份文件可以完整說明你部署的過程。

---
新增bash 腳本 需要先安裝好nvdia-smi
```
#!/bin/bash
# ----------------------------------------------------------------------
# ByteTrack GPU 部署腳本 (CUDA 12.2 / PyTorch 2.1) - 最終修正版
# 注意：此版本假設在 **ROOT** 環境下執行，並修復所有依賴問題。
# ----------------------------------------------------------------------

# Configuration Variables
export PYTHON_VERSION="3.10"
export VENV_NAME="bytetrack_env"
export REPO_URL="https://github.com/ifzhang/ByteTrack.git"
export REPO_DIR="ByteTrack"
export INSTALL_ROOT="/root/vscode" 

echo "======================================================================="
echo "🚀 啟動 ByteTrack GPU 部署流程 (Root 環境)..."
echo "目標環境: Python $PYTHON_VERSION, PyTorch 2.1.0+cu121"
echo "-----------------------------------------------------------------------"

# 函數: 檢查並執行命令
check_cmd() {
    if ! eval "$1"; then
        echo "❌ 錯誤: $2"
        exit 1
    fi
}

# ----------------------------------------------------------------------
# 🖥️ 1. 系統環境檢查與工具安裝
# ----------------------------------------------------------------------
echo "## 1. 系統環境與工具安裝"
check_cmd "apt update" "無法更新 apt 來源"
# 確保 python3.10-venv, pip, 和編譯工具都安裝
check_cmd "apt install -y python3.10-venv pip build-essential gcc g++ make python3-dev cmake git wget" "無法安裝核心系統依賴項"

# 檢查 nvidia-smi 狀態
echo "--- 執行 nvidia-smi 狀態檢查 ---"
nvidia-smi
echo "---------------------------------"

# ----------------------------------------------------------------------
# 🧱 2. 建立虛擬環境與激活
# ----------------------------------------------------------------------
echo "## 2. 建立虛擬環境與激活"
mkdir -p "$INSTALL_ROOT"
cd "$INSTALL_ROOT" || exit 1

check_cmd "python3 -m venv $VENV_NAME" "無法建立 Python 虛擬環境"
source "$VENV_NAME/bin/activate"
echo "✅ 虛擬環境 '$VENV_NAME' 已激活"

# 確保 pip 升級 (對應您的第 28 行)
check_cmd "pip install --upgrade pip setuptools wheel" "無法升級 pip/setuptools"

# ----------------------------------------------------------------------
# 🔥 3. 安裝 PyTorch CUDA 12.1 (相容性修正)
# ----------------------------------------------------------------------
echo "## 3. 安裝 PyTorch CUDA 12.1 (相容性修正)"
pip uninstall -y torch torchvision torchaudio
pip cache purge

check_cmd "pip install torch==2.1.0+cu121 torchvision==0.16.0+cu121 --index-url https://download.pytorch.org/whl/cu121" "無法安裝 PyTorch/TorchVision"

echo "--- PyTorch 環境確認 ---"
python3 - << 'PY'
import torch
print("Torch:", torch.__version__)
print("CUDA available:", torch.cuda.is_available())
if torch.cuda.is_available():
    print("GPU:", torch.cuda.get_device_name(0))
PY
echo "------------------------"

# ----------------------------------------------------------------------
# 📦 4. 修復 NumPy / OpenCV 相容性 (降級)
# ----------------------------------------------------------------------
echo "## 4. 修復 NumPy / OpenCV 相容性 (降級)"
pip uninstall -y numpy opencv-python opencv-python-headless
check_cmd "pip install numpy==1.26.4" "無法安裝 numpy 1.26.4"
check_cmd "pip install opencv-python-headless==4.8.1.78" "無法安裝 opencv-python-headless 4.8.1.78"

# ----------------------------------------------------------------------
# 📚 5. 下載 ByteTrack Repo 與安裝核心依賴
# ----------------------------------------------------------------------
echo "## 5. 下載 ByteTrack Repo 與安裝核心依賴"
if [ ! -d "$REPO_DIR" ]; then
    check_cmd "git clone $REPO_URL" "無法 clone ByteTrack 倉庫"
    cd "$REPO_DIR" || exit 1
else
    cd "$REPO_DIR" || exit 1
fi

# 創建修正版 requirements.txt
echo -e "numpy==1.26.4\nopencv-python-headless==4.8.1.78\ntorch>=2.0\ntorchvision\nloguru\nscikit-image\ntqdm\nPillow\nthop\nninja\ntabulate\ntensorboard\nlap\nmotmetrics\nfilterpy\nh5py\n\nonnx==1.12.0\nonnxruntime==1.14.1\nonnx-simplifier==0.4.10" > requirements_fixed.txt

check_cmd "pip install -r requirements_fixed.txt --no-deps" "無法安裝主要依賴項"
check_cmd "pip install coloredlogs flatbuffers packaging" "無法安裝額外依賴項"
# 確保手動安裝的依賴項被涵蓋 (thop, tabulate, scipy, lap, cython_bbox 等)
check_cmd "pip install -U scipy thop tabulate lap cython_bbox" "無法安裝或更新額外的追蹤依賴項"

# ----------------------------------------------------------------------
# 🧩 6. 安裝 COCO API
# ----------------------------------------------------------------------
echo "## 6. 安裝 COCO API"
# 雖然 cython_bbox 已在前面安裝，但此步驟仍保持，以防萬一
check_cmd "pip install 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI' --no-build-isolation" "無法安裝 COCO API"
# 確保 pycocotools 相關也安裝 (對應您的第 53 行)
check_cmd "pip install pycocotools" "無法安裝 pycocotools"

# ----------------------------------------------------------------------
# 🏗️ 7. 安裝 ByteTrack 本體 (build YOLOX kernel)
# ----------------------------------------------------------------------
echo "## 7. 安裝 ByteTrack 本體 (build YOLOX kernel)"
# 使用 -e . --no-build-isolation 確保編譯成功 (對應您的第 32 行)
check_cmd "pip install -e . --no-build-isolation" "無法安裝 ByteTrack (yolox 核心編譯失敗)"

python3 -c "import yolox; print('✅ ByteTrack setup OK')"

# ----------------------------------------------------------------------
# 📥 8. 下載預訓練模型
# ----------------------------------------------------------------------
echo "## 8. 下載預訓練模型 (修正連結)"
mkdir -p pretrained
cd pretrained || exit 1
check_cmd "wget https://github.com/ifzhang/ByteTrack/releases/download/0.1.1/bytetrack_tiny_mot17.pth.tar" "無法下載模型檔"
echo "--- 模型檔大小確認 ---"
ls -lh bytetrack_tiny_mot17.pth.tar
echo "------------------------"
cd .. # 返回 ByteTrack 根目錄

# ----------------------------------------------------------------------
# 🎬 9. 執行 ByteTrack Demo (GPU 推論)
# ----------------------------------------------------------------------
echo "## 9. 執行 ByteTrack Demo (GPU 推論測試)"
echo "--- 執行中...這可能需要一些時間 ---"

# 確保 videos/palace.mp4 存在
if [ ! -f "./videos/palace.mp4" ]; then
    echo "⚠️ 警告: 範例影片 './videos/palace.mp4' 不存在。請手動準備影片。"
    exit 0
fi

# 執行 Demo (需要手動修改 demo_track.py 避免 cv2.waitKey/imshow 錯誤)
echo "*** 請注意：您可能需要手動修改 demo_track.py 移除 cv2.waitKey/imshow 呼叫 ***"
DEMO_COMMAND="python3 tools/demo_track.py video --device gpu -f ./exps/example/mot/yolox_tiny_mix_det.py -c ./pretrained/bytetrack_tiny_mot17.pth.tar --path ./videos/palace.mp4 --save_result"
check_cmd "$DEMO_COMMAND" "ByteTrack GPU 推論執行失敗"

echo "======================================================================="
echo "🎉 部署完成！"
echo "✅ GPU 推論測試啟動成功。"
echo "輸出影片路徑: ./YOLOX_outputs/yolox_tiny_mix_det/track_vis/"
echo "======================================================================="
deactivate
```
