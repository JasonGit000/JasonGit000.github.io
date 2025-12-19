---
# YAML Front Matter：文章設定區塊
title: "讀《做自己的生命設計師》(Designing Your Life: How to Build a Well-lived, Joyful Life)"
description: "在死亡來臨前還有許多選擇，問題是我們可以怎麼做？"
categories: ["閱讀",'商管']   # 這是您的分類，可以自己決定
tags: ["Bill Burnett","Dave Evans","設計思考"]           # 這是文章的標籤
---
# 如果產品可以設計，那我們的人生呢？

因為工作緣故，時常聽到「設計思考」，用這個方式設計課程，滿足這時代不喜歡上課的學生。平時沒什麼感覺，但今天自己來讀《做自己的生命設計師》，意外有些收穫。以下摘錄非常有洞見的想法，恰好Gemini能夠辦到將書中的實作練習變成網頁功能，馬上設計出來，實在是太酷了。

### 想法1
> 「重擬」是設計師最重要的思維。許多優秀的創新都從重擬的過程開始。設計思考永遠告訴我們：「不要從問題開始，從人開始，從同理心開始。」一旦對產品使用者有同理心，就會知道該從什麼觀點出發，腦力激盪一番，開始打造原型，找出問題中的未知數。這樣的流程一般會啟動重擬的過程，有時也稱為「軸轉」（pivot）。重擬是指依據問題的新資訊，重新描述觀點，接著再度發想、重新打造原型。

### 想法2
> 人生的設計也需要很多重擬的過程，最重要的觀念重擬是瞭解到：「人生無法事先做完美的規畫」。人生不只一個解決方案，沒有標準答案是好事。人生可以有各種設計，每一種設計都能帶來希望，你可以過著有創意、不斷開展、值得體驗的人生。人生不是死的，而是一種體驗，設計並享受那個體驗，將帶來無限樂趣。

### 想法3
> 如果是無法行動的問題，就不是問題。再講一遍，如果是無法行動的問題，就不是問題，而是一種情境、一幕場景、一道人生的現實面。或許就跟重力一樣，它拖住了你。然而，重力是無法解決的問題。

### 想法4
> 一旦開始用設計師的腦袋思考，就會知道人生永遠沒有完工的一天。工作永遠沒做完，遊戲永遠沒結束，愛和健康永遠沒盡頭。只有在死亡降臨的那一刻，我們才會停止設計人生。

#### 最近如何？人生檢視儀表板
1.針對四個領域寫下幾句話，描述目前的狀況。 
<br>
2.在四個領域的儀表板上，標出自己目前的狀態（「零」到「滿格」）。 
<br>
3.問一問自己，在這些領域，是否有想要解決的設計問題。 
<br>
4.再問一問自己，那個「問題」是否為重力問題。

> 一旦找出如何定義「健康」之後，就得好好留意。你有多健康，完全要看你在回答「最近如何？」這個問題時，是如何評估自己的人生品質。


<div id="life-design-dashboard" style="font-family: 'PingFang TC', 'Microsoft JhengHei', sans-serif; background: #ffffff; padding: 30px; border-radius: 20px; border: 1px solid #eaeaea; max-width: 650px; margin: 20px auto; color: #444; box-shadow: 0 10px 30px rgba(0,0,0,0.05);">
    
    <h2 style="text-align: center; font-weight: 400; letter-spacing: 4px; color: #555; margin-bottom: 35px;">LIFE DASHBOARD</h2>

    <div style="display: flex; flex-direction: column; gap: 25px; margin-bottom: 40px;">
        <div class="input-item">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px;">
                <span style="font-weight: 600;">健康 <small style="font-weight: normal; color: #999; margin-left: 8px;">Health</small></span>
                <span id="txt-health" style="font-family: monospace; background: #eee; padding: 2px 8px; border-radius: 4px;">5</span>
            </div>
            <input type="range" min="0" max="10" value="5" class="range-slider" id="in-health" oninput="updateDashboard()">
            <div style="font-size: 1.00em; color: #aaa; margin-top: 4px;">現在身、心、靈的健康程度</div>
        </div>

        <div class="input-item">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px;">
                <span style="font-weight: 600;">工作 <small style="font-weight: normal; color: #999; margin-left: 8px;">Work</small></span>
                <span id="txt-work" style="font-family: monospace; background: #eee; padding: 2px 8px; border-radius: 4px;">5</span>
            </div>
            <input type="range" min="0" max="10" value="5" class="range-slider" id="in-work" oninput="updateDashboard()">
            <div style="font-size: 1.00em; color: #aaa; margin-top: 4px;">有薪工作，無薪工作，照顧家庭，做家事，都算是工作</div>
        </div>

        <div class="input-item">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px;">
                <span style="font-weight: 600;">遊戲 <small style="font-weight: normal; color: #999; margin-left: 8px;">Play</small></span>
                <span id="txt-play" style="font-family: monospace; background: #eee; padding: 2px 8px; border-radius: 4px;">5</span>
            </div>
            <input type="range" min="0" max="10" value="5" class="range-slider" id="in-play" oninput="updateDashboard()">
            <div style="font-size: 1.00em; color: #aaa; margin-top: 4px;">就是做了會很開心也會滿足，不計結果為何的事情。人人都需要遊戲</div>
        </div>

        <div class="input-item">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px;">
                <span style="font-weight: 600;">愛 <small style="font-weight: normal; color: #999; margin-left: 8px;">Love</small></span>
                <span id="txt-love" style="font-family: monospace; background: #eee; padding: 2px 8px; border-radius: 4px;">5</span>
            </div>
            <input type="range" min="0" max="10" value="5" class="range-slider" id="in-love" oninput="updateDashboard()">
            <div style="font-size: 1.00em; color: #aaa; margin-top: 4px;">連結感、親密關係、歸屬感。愛就是讓你的世界運轉的力量。任何能投射情感的事物</div>
        </div>
    </div>

    <div id="visual-dashboard" style="display: flex; flex-direction: column; gap: 18px; padding: 20px; background: #fdfdfd; border-radius: 12px;">
    
        <div style="text-align: center; margin-bottom: 15px; border-bottom: 1px dashed #ccc; padding-bottom: 10px;">
        <span style="font-size: 1.2em; font-weight: bold; color: #666; letter-spacing: 1px;">
            目前人生狀態分析結果
        </span>
        </div>

        
        <div style="display: flex; align-items: center; gap: 15px;">
            <div id="lbl-health" style="width: 50px; font-weight: bold; transition: color 0.4s;">健康</div>
            <div style="flex-grow: 1; height: 12px; background: #f0f0f0; border-radius: 6px; overflow: hidden;">
                <div id="bar-health" style="width: 50%; height: 100%; transition: all 0.5s cubic-bezier(0.18, 0.89, 0.32, 1.28);"></div>
            </div>
        </div>

        <div style="display: flex; align-items: center; gap: 15px;">
            <div id="lbl-work" style="width: 50px; font-weight: bold; transition: color 0.4s;">工作</div>
            <div style="flex-grow: 1; height: 12px; background: #f0f0f0; border-radius: 6px; overflow: hidden;">
                <div id="bar-work" style="width: 50%; height: 100%; transition: all 0.5s cubic-bezier(0.18, 0.89, 0.32, 1.28);"></div>
            </div>
        </div>

        <div style="display: flex; align-items: center; gap: 15px;">
            <div id="lbl-play" style="width: 50px; font-weight: bold; transition: color 0.4s;">遊戲</div>
            <div style="flex-grow: 1; height: 12px; background: #f0f0f0; border-radius: 6px; overflow: hidden;">
                <div id="bar-play" style="width: 50%; height: 100%; transition: all 0.5s cubic-bezier(0.18, 0.89, 0.32, 1.28);"></div>
            </div>
        </div>

        <div style="display: flex; align-items: center; gap: 15px;">
            <div id="lbl-love" style="width: 50px; font-weight: bold; transition: color 0.4s;">愛</div>
            <div style="flex-grow: 1; height: 12px; background: #f0f0f0; border-radius: 6px; overflow: hidden;">
                <div id="bar-love" style="width: 50%; height: 100%; transition: all 0.5s cubic-bezier(0.18, 0.89, 0.32, 1.28);"></div>
            </div>
        </div>

    </div>

    <style>
        .range-slider {
            -webkit-appearance: none; width: 100%; height: 4px; background: #ddd; border-radius: 2px; outline: none;
        }
        .range-slider::-webkit-slider-thumb {
            -webkit-appearance: none; width: 16px; height: 16px; background: #666; cursor: pointer; border-radius: 50%; border: 2px solid #fff; box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
    </style>

    <script>
        function updateDashboard() {
            const metrics = ['health', 'work', 'play', 'love'];
            // 定義文青色標 (莫蘭迪色系)
            const colors = {
                low: '#d66d6d',    // 深珊瑚紅
                mid: '#e9a37e',    // 灰調橘
                high: '#9db4a0'    // 鼠尾草綠
            };
            
            metrics.forEach(m => {
                const val = document.getElementById('in-' + m).value;
                document.getElementById('txt-' + m).innerText = val;
                
                const bar = document.getElementById('bar-' + m);
                const lbl = document.getElementById('lbl-' + m);
                
                bar.style.width = (val * 10) + '%';
                
                let activeColor = '';
                if (val <= 3) activeColor = colors.low;
                else if (val <= 7) activeColor = colors.mid;
                else activeColor = colors.high;
                
                bar.style.backgroundColor = activeColor;
                lbl.style.color = activeColor; // 讓標籤顏色跟著變
            });
        }
        updateDashboard();
    </script>
</div>