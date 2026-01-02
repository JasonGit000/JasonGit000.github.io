---
# YAML Front Matter：文章設定區塊
title: "Python套件plotly資料視覺化"
description: "在問卷平台資料沒辦法按照想要的方式視覺化即時呈現怎麼辦？使用pyhton的plotly以html檔案呈現，可以透過滑鼠靠近呈現複雜度較高的資訊"
categories: ["Coding",'資料分析']   # 這是您的分類，可以自己決定
tags: ["Python","Data Analysis","Gemini"]           # 這是文章的標籤
---

## 結果範例

<iframe 
    src="{{ '/assets/charts/data_analysis01.html' | relative_url }}" 
    width="100%" 
    height="3500px" 
    frameborder="0" 
    scrolling="yes">
</iframe>


## 程式碼
加入了字串分析的問題解決辦法，以及合併不同圖表html的辦法，Gemini Pro 儘管可以依照當下的自然語言需求輸出成果，但無法理解到各個部分的細緻差異。建議設定不同的檢核點，讓illusion的部分可以被使用者揭露。

```python
import pandas as pd
import plotly.express as px

# 1. 取得目標欄位 (索引 5 到 14)
target_columns = df2.columns[5:15]

# 定義哪些欄位需要「複選拆分」處理
multi_select_cols = [
    '您所使用的AI工具（可複選）', 
    '您所使用的AI技術（可複選）', 
    '您希望從本活動中獲得什麼？（可複選）'
]

all_html_charts = ""

# 為原始資料加上唯一的 ID，用於後續去重校正
df2_with_id = df2.copy()
df2_with_id['respondent_id'] = range(len(df2_with_id))

for col in target_columns:
    # 檢查是否全空
    if df2_with_id[col].dropna().empty:
        print(f"跳過空欄位: {col}")
        continue

    # 2. 資料整理與去重邏輯
    if col in multi_select_cols:
        # A. 建立臨時 DataFrame 並拆分
        # 將分隔符統一轉換為逗號，然後 explode 展開成多行
        temp_df = df2_with_id[[col, 'respondent_id']].copy()
        temp_df[col] = temp_df[col].astype(str).str.replace(r'[\n；;、]', ',', regex=True)
        
        # 拆分字串並展開
        exploded = temp_df.assign(選項=temp_df[col].str.split(',')).explode('選項')
        exploded['選項'] = exploded['選項'].str.strip()
        
        # B. 關鍵修正：合併破碎標籤名稱
        exploded['選項'] = exploded['選項'].replace({
            '與跨域學生': '與跨域學生、教師共同探索解方',
            '教師共同探索解方': '與跨域學生、教師共同探索解方'
        })
        
        # C. 清洗：排除 nan 和空字串
        cleaned_df = exploded[(exploded['選項'].str.len() > 0) & (exploded['選項'].str.lower() != 'nan')]
        
        # D. 精準去重：確保同一個填答者 (respondent_id) 在同一個選項只算一次
        final_series = cleaned_df.drop_duplicates(subset=['respondent_id', '選項'])['選項']
    else:
        # 一般單選處理
        final_series = df2_with_id[col].astype(str).str.strip()
        final_series = final_series[(final_series.str.len() > 0) & (final_series.str.lower() != 'nan')]

    if final_series.empty:
        continue

    # 3. 計算頻率
    counts_df = final_series.value_counts().reset_index()
    counts_df.columns = ['選項', '計數']

    # 如果是開放式文字，只取前 20 名
    if len(counts_df) > 20:
        counts_df = counts_df.head(20)
        title_suffix = " (僅顯示前 20 名)"
    else:
        title_suffix = ""

    # 4. 繪製 Plotly 圖表
    fig = px.bar(
        counts_df, 
        x='計數', 
        y='選項', 
        orientation='h',
        title=f'<b>{col}{title_suffix}</b>',
        text='計數',
        color='計數',
        color_continuous_scale='Viridis',
        template='plotly_white'
    )

    # 5. 優化視覺佈局
    fig.update_layout(
        height=max(400, len(counts_df) * 35),
        margin=dict(l=300, r=50, t=80, b=50), 
        yaxis={'categoryorder': 'total ascending'},
        xaxis_title="人數 / 次數",
        yaxis_title=None,
        showlegend=False,
        coloraxis_showscale=False
    )
    
    fig.update_traces(textposition='outside', cliponaxis=False)

    # 顯示圖表
    fig.show()
    
    # 儲存到 HTML 串接
    all_html_charts += fig.to_html(full_html=False, include_plotlyjs='cdn')
    all_html_charts += "<br><hr><br>"

# 6. 輸出整合報告
output_file = "AI_Activity_Refined_Analysis.html"
with open(output_file, "w", encoding="utf-8") as f:
    f.write("<html><head><meta charset='utf-8'><title>活動數據精準分析報告</title></head>")
    f.write("<body style='background-color:#f8f9fa; padding:20px; font-family: sans-serif;'>")
    f.write("<h1 style='text-align:center;'>活動問卷分析報告</h1>")
    f.write(all_html_charts)
    f.write("</body></html>")

print(f"✅ 分析完成！精準校正後的報告已儲存至: {output_file}")
```