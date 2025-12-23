---
# YAML Front Matter：文章設定區塊
title: "Python互動式文字雲實作"
description: "當資料標籤多，重複數量不高，分散，要如何呈現視覺化呢？使用互動式html格式的文字雲一次解決"
categories: ["Coding",'資料分析']   # 這是您的分類，可以自己決定
tags: ["Python","World Cloud","Gemini"]           # 這是文章的標籤
---
# 報名數據分析的新維度：使用動態文字雲 HTML 的四大優勢

在數據分析的視覺化工具中，**動態文字雲（Dynamic Word Cloud）** 是一種能將「文字數據」轉化為「視覺饗宴」的高效方式。特別是將其封裝在 **HTML 檔案** 中，不僅便於分享，還能提供互動性，讓報名數據不再只是冰冷的數字。

以下是使用 HTML 動態文字雲來分析報名數據（如：報名動機、職業、興趣標籤等）的核心優點：

---

## 1. 直觀展現核心趨勢
傳統的報名數據（如 Excel 表格）難以快速閱讀。文字雲透過**字體大小**與**顏色深淺**，直接呈現出現頻率最高的關鍵詞。

* **視覺權重**：字體越大，代表該報名特徵（如「跨領域學習」、「數位轉型」）越集中。
* **快速決策**：決策者無需閱讀數百份報名表，只需掃視一眼即可掌握本屆活動的核心受眾特質。

---

## 2. 強大的互動性與探索深度
相較於靜態的圖片（PNG/JPG），HTML 格式的動態文字雲允許使用者進行即時互動：

* **懸停顯示數據**：當滑鼠游標移至某個詞彙時，可以顯示精確的報名次數或百分比。
* **動態排列**：動畫效果能引導視覺焦點，讓數據呈現更具層次感。
* **連結擴展**：可設定點擊關鍵字後跳轉至相關的詳細分析頁面，增加探索深度。

---

## 3. 跨平台的高可攜性與相容性
將分析結果儲存為 HTML 檔案，具有極高的便利性：

* **無需特定軟體**：接收者只需要有瀏覽器（Chrome, Edge, Safari），就能完整體驗動態效果，不需要安裝專業的 BI 工具。
* **易於整合**：HTML 代碼可以輕易嵌入到公司內部系統、活動官網或 Notion 頁面中。
* **輕量化**：檔案體積小，非常適合透過通訊軟體或電子郵件即時傳送。

---

## 4. 挖掘質性數據中的隱藏價值
報名表中的「開放式問題」（例如：您對本次課程的期待是什麼？）通常是最難分析的部分。

* **需求識別**：文字雲能將散亂的句子轉化為結構化的標籤，幫助主辦單位發現原先未預料到的熱門話題。
* **行銷優化**：若文字雲顯示「實作」一詞出現頻率遠高於「理論」，主辦單位可立即調整行銷文案，強化該特點。

---

### 數據呈現對比表

| 維度 | 傳統表格分析 | HTML 動態文字雲 |
| :--- | :--- | :--- |
| **閱讀速度** | 慢（需逐行掃視） | **極快（視覺焦點集中）** |
| **互動體驗** | 無 | **高（懸停、動態縮放）** |
| **技術門檻** | 中（需處理篩選） | **低（瀏覽器隨開即看）** |
| **感官力** | 一般 | **優（更具專業感與美感）** |

---

## 結果範例

<iframe 
    src="{{ '/assets/charts/wordcloud_interactive.html' | relative_url }}" 
    width="100%" 
    height="500px" 
    frameborder="0" 
    scrolling="no">
</iframe>

---
## 實作步驟

### 1. 導入pandas，並讀取csv
```python
import pandas as pd
df=pd.read_csv("我是檔案名稱.csv")

print(df.info())
print(df.describe())
```

### 2. 用groupby計算人數
```python
df2=df.groupby('我是資料欄名稱')['我是計算欄的名稱'].count().reset_index(name='人數')

#按照數量做篩選
df2=df2.sort_values(by='人數', ascending=False)

display(df2)

total=df2['人數'].sum()
print(f"總參與人數:{total}")
```

### 3. 使用動態worldcloud展示結果

```python
from pyecharts import options as opts
from pyecharts.charts import WordCloud

# 1. 確保數據完全脫殼（轉為原生 Python 類型）
data = [(str(row.iloc[0]), int(row.iloc[1])) for _, row in df2.iterrows()]

# 2. 建立文字雲物件
c = (
    WordCloud(init_opts=opts.InitOpts(
        width="100%",       # 強制寬度自適應
        height="500px",      # 高度可以固定
        animation_opts=opts.AnimationOpts(animation=True)
    ))
    .add(
        series_name="參與人數", 
        data_pair=data, 
        word_size_range=[12, 55], 
        shape="circle",
        # --- 關鍵修正：解決空白問題並強制橫排 ---
        rotate_step=0,                # 步進設為 0
        pos_left="left",       # 確保中心對齊
        pos_top="left",
    )
    .set_global_opts(
        #title_opts=opts.TitleOpts(title="單位參與人數分析 - 動態文字雲"),
        tooltip_opts=opts.TooltipOpts(is_show=True),
    )
    # --- 透過自定義配置強制修改底層的 rotationRange ---
    .set_series_opts(
        label_opts=opts.LabelOpts(is_show=False)
    )
)

# 3. 如果生成的 HTML 還是有歪掉的字，請用以下這個「強行介入」的方法：
# 這會直接覆蓋 pyecharts 生成的 JSON 配置，確保它是全橫式且不空白
c.options["series"][0].update({
    "rotationRange": [0, 0],
    "rotationStep": 0
})

# 4. 輸出
c.render("wordcloud_interactive.html")
```

## 實作問題排解
### 1. 如果有人重複報名怎麼辦？
用指令duplicated一鍵篩選

```python
# 找出所有重複出現的姓名列（不包含被保留的那一筆）
duplicates = df[df.duplicated(subset=['您的姓名'], keep='last')]
print(duplicates)
```

用drop_duplicates移除，再用同樣的方式列出檢查
```python
# 1. 先按時間排序（由舊到新）
df = df.sort_values(by='填答時間')

# 2. 再執行刪除重複，保留最後一筆
df_final = df.drop_duplicates(subset=['您的姓名'], keep='last')

# 檢查是否還有
# 找出所有重複出現的姓名列（不包含被保留的那一筆）
duplicates = df_final[df_final.duplicated(subset=['您的姓名'], keep='last')]
print(duplicates)
```

### 2. 如果有人填答學校名稱不一怎麼辦？
做資料清理排除，本次清理解決佛光，佛光大學變成兩列資料的問題

```python
import pandas as pd

# 1. 預處理：去除數字
df2['單位_純文字'] = df2['我是資料名稱'].str.replace(r'\d+', '', regex=True)

# 2. 核心邏輯：抓取關鍵字 (注意：這裡不需要包含 '佛光'，除非它沒寫大學)
patterns = r'(^.*?大學|^.*?科大|^.*?學院|雲科大|暨大|佛光)'
df2['標準單位'] = df2['單位_純文字'].str.extract(patterns)[0]

# 3. 補回沒抓到關鍵字的資料
df2['標準單位'] = df2['標準單位'].fillna(df2['單位_純文字'])

# --- 重要修正步驟：精準替換 ---
# 使用 map 或 replace 字典，只針對「完全符合」的字串進行修正
correction_map = {
    "佛光": "佛光大學",
    "雲科大": "國立雲林科技大學",
    "暨大": "國立暨南國際大學"
}

# 這裡使用 replace，它會檢查整個字串，如果剛好是 "佛光" 才會換掉
# 已經是 "佛光大學" 的就不會被觸動，解決 "佛光大學大學" 的問題
df2['標準單位'] = df2['標準單位'].replace(correction_map)

# 移除空格
df2['標準單位'] = df2['標準單位'].str.strip()

# 5. 進行彙整
df3 = df2.groupby('標準單位')['您的姓名'].count().reset_index(name='人數')
df3 = df3.sort_values(by='人數', ascending=False)

display(df3)

total = df3['人數'].sum()
print(f"總參與人數: {total}")
```
