---
# YAML Front Matter：文章設定區塊
title: "用本地 AI 幫 PDF/Docx 自動改名：一個實用的批次命名工具"
description: "用Ollama搭配本地模型分析文件內容，自動生成精簡檔名，同時智慧辨識掃描檔，批次處理免去手動重新命名的麻煩"
categories: ["資料分析"]   # 這是您的分類，可以自己決定
tags: ["Gemini","本地端AI"]           # 這是文章的標籤
---

## 內文

如果你的電腦裡也躺著一堆 `掃描_0012.pdf`、`未命名文件(3).docx`，你大概懂那種想找檔案卻只能一個一個點開來看的痛苦。這篇要介紹的，是一個用本地 AI 模型自動幫 PDF、Docx 檔案重新命名的小工具，核心邏輯不複雜，但幾個設計細節處理得很到位，值得拆開來看看。

### 為什麼要用「本地」AI

工具選用 Ollama 搭配 `qwen2.5:3b` 這種輕量模型，而不是呼叫雲端 API。這個選擇背後有兩個很實際的考量：一是檔案內容可能涉及合約、發票、會議紀錄等隱私資料，不上傳到雲端比較安心；二是批次處理動輒上百個檔案，如果每個都打 API，速度和費用都會是問題。本地模型雖然能力不如大型雲端模型，但拿來做「讀懂前幾百字、生成一個檔名」這種輕量任務，其實綽綽有餘。

### 亮點一：先判斷是不是掃描檔，再決定要不要問 AI

這是整個腳本裡我覺得最聰明的一個小設計。`extract_content()` 會先嘗試從 PDF 前兩頁或 Docx 前 15 段抓文字，如果抓出來的內容少於 15 個字，就直接判定是「掃描檔」（也就是純圖片、沒有可提取文字層的 PDF），直接標記為 `掃描檔案_原檔名`，完全不會浪費時間去呼叫 AI。

這個設計避免了兩個常見的坑：一是 OCR 沒做的掃描檔硬塞給語言模型只會得到亂猜的結果；二是省下了不必要的運算資源。用一個簡單的字數門檻做前置判斷，成本很低但效果直接。

### 亮點二：Prompt 設計得夠「收斂」

`NAMING_PROMPT` 明確要求「僅回傳 JSON」、限制檔名不超過 15 字、不能有特殊字元。這種把輸出格式和限制條件寫死在 prompt 裡的作法，對小模型特別重要——3B 參數的模型如果不給清楚的格式規範，很容易生成一堆廢話或格式跑掉的內容。搭配 `temperature=0.1` 讓輸出更穩定、更少發散，這對「命名」這種需要確定性而非創意的任務來說是對的選擇。

### 亮點三：層層容錯，絕不讓程式中斷

批次處理最怕的就是跑到第 87 個檔案時，因為某個特例炸掉整個流程。這支程式在好幾層都做了防呆：

- `extract_content()` 用 `try/except` 包住讀檔邏輯，讀取失敗就回傳空字串，自然會被判定為掃描檔而不是讓程式當掉。
- `get_new_name()` 呼叫 AI 後，先用正則表達式 `re.search(r'\{.*\}', ...)` 從回應中撈出 JSON 片段（因為小模型偶爾會在 JSON 前後加一些廢話），解析失敗就 fallback 成 `分析失敗_原檔名`。
- 就算 AI 完全罷工，程式依然會產生一個可辨識、不重複的檔名，而不是讓整批任務停擺。

這種「寧可結果不完美，也不要讓流程中斷」的設計哲學，在批次處理大量檔案時非常實用。

### 亮點四：檔名清理與碰撞處理

`clean_filename()` 用正則過濾掉非中英數字元，並把連續底線合併、去頭尾，確保產出的檔名在各作業系統上都合法好用。而 `process_file()` 裡處理檔名重複的邏輯也很簡潔：用 `while target_path.exists()` 搭配計數器自動加上 `_1`、`_2` 後綴，避免同名檔案互相覆蓋——這種細節容易被忽略，但在真實批次任務裡出錯率其實不低。

### 亮點五：併發加速 + 使用者友善的執行流程

用 `ThreadPoolExecutor` 搭配 `MAX_WORKERS = 4` 做併發處理，同時考量到本地 Ollama 服務的負載能力（註解裡也提醒不要開太多避免當機），是務實而非一味追求速度的做法。整個 CLI 流程也照顧到使用者體驗：先跑分析、印出前 30 筆命名預覽讓使用者確認，得到 `y` 才真正執行複製，避免「AI 亂命名結果覆蓋原始檔案」的災難情境。搭配 `tqdm` 進度條，長時間跑批次也不會讓人覺得程式卡死。

### 小結

這支工具沒有用到多複雜的技術，但每一個環節——內容提取、AI 命名、容錯、檔名清理、使用者確認——都處理得很紮實，是一個典型「小而完整」的實用腳本範例。如果要進一步優化，可以考慮加入 OCR 處理掃描檔（例如串接 Tesseract）、支援更多副檔名（如 `.doc`、`.txt`），或是把命名規則做成可設定的樣板，讓不同使用情境（發票、合約、會議記錄）套用不同的命名邏輯。

```zsh
import os
import re
import json
import shutil
import concurrent.futures
from pathlib import Path
from tqdm import tqdm
import ollama
from pypdf import PdfReader
from docx import Document

# ─────────────────────────── 設定區 ───────────────────────────
MODEL = "qwen2.5:3b"
MAX_WORKERS = 4      # 建議不要超過 4，避免 Ollama 同時處理太多請求導致當機
PREVIEW_COUNT = 30   # 設定預覽顯示的筆數
SUPPORTED = {".pdf", ".docx"} # 排除 .doc 以避免 python-docx 讀取錯誤

NAMING_PROMPT = """你是一個檔案命名專家。請根據文件內容產生一個精簡的繁體中文檔名。
要求：
1. 檔名須反映內容核心（如：會議紀錄、發票、合約），不超過 15 字。
2. 不要包含特殊字元。
3. 僅回傳 JSON：{{"filename": "產出的檔名"}}

內容：
{content}
"""

# ─────────────────────────── 核心功能 ───────────────────────────

def extract_content(path):
    """提取文字，若字數不足 15 字則視為掃描檔"""
    ext = path.suffix.lower()
    text = ""
    try:
        if ext == ".pdf":
            reader = PdfReader(str(path))
            # 讀取前 2 頁進行判定
            for i in range(min(2, len(reader.pages))):
                page_text = reader.pages[i].extract_text() or ""
                text += page_text
        elif ext == ".docx":
            doc = Document(str(path))
            text = "\n".join(p.text for p in doc.paragraphs[:15])
        
        return re.sub(r'\s+', ' ', text).strip()
    except Exception:
        return ""

def clean_filename(name):
    """清理檔名：移除特殊字元、合併底線、限制長度"""
    # 只允許中文字、英數字、底線
    name = re.sub(r'[^\w\u4e00-\u9fff]', '_', name)
    # 合併重複底線並去頭尾
    name = re.sub(r'_+', '_', name).strip('_')
    return name[:20] # 限制 AI 產出的部分最長 20 字

def get_new_name(p):
    """判斷邏輯：掃描檔標記 vs AI 命名"""
    content = extract_content(p)
    original_stem = p.stem
    
    # 1. 掃描檔判定
    if len(content) < 15:
        return f"掃描檔案_{original_stem}"
    
    # 2. AI 命名分析
    try:
        response = ollama.chat(
            model=MODEL,
            messages=[{"role": "user", "content": NAMING_PROMPT.format(content=content[:1200])}],
            options={"temperature": 0.1}
        )
        
        # 解析 JSON 內容
        res_content = response["message"]["content"]
        match = re.search(r'\{.*\}', res_content, re.DOTALL)
        if match:
            data = json.loads(match.group(0))
            ai_suggested = clean_filename(data.get("filename", ""))
            if ai_suggested:
                return f"{ai_suggested}_{original_stem}"
    except Exception:
        pass
        
    return f"分析失敗_{original_stem}"

def process_file(p, dst_dir):
    """單一檔案處理：生成新路徑並處理檔名碰撞"""
    new_stem = get_new_name(p)
    ext = p.suffix.lower()
    target_path = dst_dir / f"{new_stem}{ext}"
    
    # 處理重複檔名
    counter = 1
    while target_path.exists():
        target_path = dst_dir / f"{new_stem}_{counter}{ext}"
        counter += 1
    return {"old": p, "new": target_path}

# ─────────────────────────── 執行主程式 ───────────────────────────

def main():
    print("\n" + "="*50)
    print(" 🚀 PDF/Docx 智能命名工具 (掃描檔識別版)")
    print("="*50)
    
    src_input = input("📂 來源資料夾路徑：").strip().strip("'\"")
    dst_input = input("📂 輸出資料夾路徑：").strip().strip("'\"")
    
    src_dir, dst_dir = Path(src_input).resolve(), Path(dst_input).resolve()
    
    if not src_dir.exists():
        print(f"❌ 找不到來源路徑！")
        return

    dst_dir.mkdir(parents=True, exist_ok=True)

    # 取得檔案清單
    files = [p for p in src_dir.rglob("*") if p.is_file() and p.suffix.lower() in SUPPORTED]
    if not files:
        print("查無支援的 PDF 或 Docx 檔案。")
        return

    print(f"\n📦 找到 {len(files)} 個檔案，開始分析內容...")
    results = []

    # 使用併發加速
    with concurrent.futures.ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
        futures = {executor.submit(process_file, f, dst_dir): f for f in files}
        for future in tqdm(concurrent.futures.as_completed(futures), total=len(files), desc="分析中"):
            results.append(future.result())

    # 預覽結果
    print(f"\n📝 命名預覽 (前 {PREVIEW_COUNT} 筆)：")
    for r in results[:PREVIEW_COUNT]:
        print(f"  [舊] {r['old'].name}")
        print(f"  [新] {r['new'].name}\n")

    # 最後確認
    confirm = input(f"❓ 確認複製並改名共 {len(results)} 個檔案？ (y/n): ")
    if confirm.lower() == 'y':
        for r in tqdm(results, desc="複製進度"):
            shutil.copy2(r['old'], r['new'])
        print(f"\n✅ 完成！檔案已存至: {dst_dir}")
    else:
        print("操作已取消。")

if __name__ == "__main__":
    main()

```
