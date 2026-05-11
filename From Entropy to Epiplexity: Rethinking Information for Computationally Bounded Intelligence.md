# From Entropy to Epiplexity: Rethinking Information for Computationally Bounded Intelligence

**核心問題與動機（Core Problem and Motivation）**

論文的核心問題是：在計算資源受限的觀察者（computationally bounded intelligence，如現代神經網路）眼中，資料中「可學習的結構性資訊」（structural information）該如何量化？古典資訊理論（如Shannon entropy與Kolmogorov complexity）假設無限計算能力，導致三個看似矛盾的悖論，與深度學習實務嚴重脫節：

1. **資訊無法透過確定性過程增加**（Paradox 1）：資料處理不等式（Data Processing Inequality）指出確定性轉換不會增加資訊。但現實中，合成資料、自玩（self-play，如AlphaZero）、偽隨機生成器、混沌系統模擬、數學推導等，都能「創造」有用知識或能力。
2. **資訊與資料因式分解順序無關**（Paradox 2）：Shannon與Kolmogorov下，X→Y與Y→X的總資訊相同。但LLM在正向英文 vs. 反向英文、加密函數的正向 vs. 反向預測、棋局不同排序的學習效果明顯不同。
3. **似然建模只是分布匹配**（Paradox 3）：完美模型就是生成過程本身，無法從資料中「萃取」更多結構。但Conway's Game of Life中，簡單規則產生複雜「 glider」等 emergent structures，計算受限模型必須學習這些高階模式才能有效預測。

**動機**：現代AI追求廣泛OOD泛化（out-of-distribution generalization），成功關鍵在於「資料選擇」（data selection）而非僅模型選擇。現有理論無法指導如何評估資料價值、合成資料效益、或不同預訓練資料的轉移能力。論文提出「epiplexity」（epistemic complexity，認知複雜度）作為計算受限觀察者能從資料中萃取的結構性資訊測度，補足古典理論的不足，為資料策展提供理論基礎。

**結果/成果（Results/Achievements）**

- **理論貢獻**：
  - 引入 **time-bounded entropy**（時間受限熵）：計算受限觀察者眼中資料的「隨機不可預測」部分（如多項式時間無法破解的偽隨機或單向函數逆向）。
  - 引入 **epiplexity**（S_T(X)）：最小化描述長度（MDL）中，用來描述「模型/程式」本身的位元數，捕捉結構性內容。總描述長度 = epiplexity + time-bounded entropy。
  - 證明在單向置換（one-way permutations）下，不同因式分解（factorization）會產生時間受限熵的差距（Theorem 13 等）。
  - 「Limited Epiplexity Increase Property」：即使允許更多計算，epiplexity 仍有界限增長，但可明顯大於生成程式的長度（與Kolmogorov的O(1)差距形成對比）。
  - 資訊是observer-dependent：同樣資料對無限計算觀察者可能是低熵，對受限觀察者可能是高epiplexity或高time-bounded entropy。

- **實證貢獻與估計方法**：
  - **Prequential coding**（簡單啟發式）：訓練loss曲線下方與最終loss之間的面積 ≈ epiplexity。高epiplexity資料有緩慢衰減但最終低loss的曲線。
  - **Requential coding**（更嚴謹）：累積teacher-student KL divergence，模擬訓練過程傳輸模型。
  - 透過Chinchilla式scaling找到compute-optimal模型大小，確保測量反映資料本質而非超參數。

- **實驗展示**：
  - **Rule 30 / ECA cellular automata**：正向容易達到Shannon entropy，反向有明顯time-bounded entropy gap；模型能建模但逆向困難。
  - **Chess資料不同排序**：從最終棋盤預測移動序列（reverse）比正向產生更高time-bounded entropy與更高epiplexity，且提升OOD表現（如 puzzle solving）。
  - 自然資料比較：文字遠高於影像（CIFAR），解釋語言預訓練轉移性更好。
  - 資料介入（如shuffle pixels）大幅降低epiplexity，追蹤OOD性能。

**分析與洞見（Analysis and Insights）**

- **資訊可被「創造」**：透過計算過程（dynamical systems、self-play），確定性轉換能產生emergent structures。無限計算觀察者可直接模擬規則（低epiplexity），但受限模型被迫學習高階啟發式（如gliders、invariant measure），這正是epiplexity增加的來源。合成資料因此有價值，而非DPI預測的「零新增」。
- **因式分解/ordering的影響**： harder factorization（逆向、複雜預測）迫使模型內化更深層結構，即使訓練loss較差，也提升epiplexity與泛化。對應「arrow of time」與加密中的計算不對稱。
- **似然最大化超越分布匹配**：模型權重成為包含sub-circuits（induction heads等）的程式，這些可重用於OOD任務。epiplexity測量「壓縮」過程中吸收的結構，而非僅最終perplexity（perplexity混合noise與structure）。
- **實務意涵**：
  - 資料選擇優先高epiplexity來源（rich long-range dependencies，而非純random或過度redundant）。
  - 解釋為何某些合成/自玩資料有效、為何文字優於影像、為何特定ordering或curriculum更好。
  - 邊緣案例：純noise（高time-bounded entropy、低epiplexity）無幫助；過簡單資料（低兩者）也無幫助；高epiplexity不保證特定任務相關性，僅提供可重用結構的「潛力」。
  - 與模型選擇對比：epiplexity聚焦data-centric AI，提供量化指導。

**結論（Conclusion）**

論文成功架起古典資訊理論與現代深度學習之間的橋樑，主張對計算受限智能而言，資訊本質上是observer-dependent且可透過計算過程「建構」。epiplexity提供一個可操作、可估計的框架，解決古典理論的悖論，並為資料選擇、合成資料生成、預訓練策略奠定理論基礎。

未來方向可能包括：更精準的epiplexity估計、跨模態應用、與scaling law的整合、或用於意識/湧現相關討論。
