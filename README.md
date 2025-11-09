
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
