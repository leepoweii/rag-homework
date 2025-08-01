# 夢酒館 RAG 品牌大使

## 專案背景說明

我在金門開了一間酒吧叫做夢酒館，這個專案本來是學校作業，當時剛好在寫金門的補助案，想到可以做一個 RAG 的內部資料集，因為常常在申請政府案子，很常需要重複整理有關公司資訊等等的，因此認為可以把相關資訊都全部放進 RAG，未來可以根據不同的專案，提取資料並且快速整合撰寫成不同方向的計劃書。

目前因為是作業實作，先將機器人設定為是對外的品牌介紹人員，分享給朋友體驗的時候會比較有趣。

### 目前資料庫內容
1. 品牌介紹
2. 調酒酒單介紹
3. 相關媒體報導

## 發現比較大的問題與解法

### 複合式問句問題
**發現問題**: RAG 系統一次只能處理單一問題，對於包含多個子句的複合式問句，無法精確理解並提供完整答案。

**目前解法**: 為了解決這個問題，利用 LLM 先將複合式問句拆解成多個子問題，分別進行檢索，最後再整合所有答案。

### 內容組合錯誤
**發現問題**: 由於 RAG 系統將內容切割成 chunks，在檢索過程中，偶爾會出現內容組合錯誤，例如詢問 A 調酒卻得到 B 調酒的介紹。目前已發現問題，但仍在尋找最佳解法。

**目前解法**: 有發現問題但暫時還沒辦法完全理解和實作出解法，因此還沒有更新到程式內。

## 技術架構

### 重新設計的檢索和回答流程

1. **語意拆解**: 使用大型語言模型（LLM）將使用者輸入的複合式問題，拆解成多個語意清晰、可獨立檢索的子問題。
2. **個別檢索**: 針對每個拆解後的子問題，分別從知識庫中檢索相關文件。
3. **整合回答**: 根據原始問句，以及步驟二中檢索到的所有相關文件，使用 LLM 整合生成最終的完整回覆。

### 核心技術
- **語意拆解**: 自訂 prompt 透過 OpenAI GPT-4.1 nano 將複合問句拆解成子問題
- **向量檢索**: 自訂 E5 embedding 類別 + FAISS 向量資料庫
- **檢索策略**: 使用 top k = 3 讓資料更完整
- **UI**: Gradio 快速打造 Web App

## 快速開始

### 環境設置
```bash
# Clone 專案
git clone https://github.com/leepoweii/rag-homework.git
cd rag-homework

# 安裝核心依賴
uv sync

# 如果需要開發環境（Jupyter notebook 等）
uv sync --extra dev

# 設置環境變數
cp .env.example .env
# 編輯 .env 添加你的 OpenAI API Key
```

### 運行專案
```bash
# 如果已安裝開發依賴，啟動 Jupyter Notebook
uv run jupyter notebook

# 打開 酒吧RAG品牌大使.ipynb 並執行

# 或者直接在 VS Code 執行也可以
```

## 使用示範

### 測試案例
```python
user_input = "我喜歡清爽氣泡的調酒，請你推薦我一杯調酒。同時也請你跟我說明夢酒館在地方所做的貢獻，還有夢酒館登過的國際媒體報導。"
```

### 系統處理流程
1. **語意拆解結果**:
   - 子問句1：我喜歡清爽氣泡的調酒。
   - 子問句2：請你推薦我一杯調酒。
   - 子問句3：請你跟我說明夢酒館在地方所做的貢獻。
   - 子問句4：請你說明夢酒館登過的國際媒體報導。

2. **個別檢索**: 針對每個子句分別檢索相關資料

3. **整合回答**: 根據原始問句整合生成完整回覆

## 技術細節

### 自訂 E5 Embedding 類別
```python
class CustomE5Embedding(HuggingFaceEmbeddings):
    def embed_documents(self, texts):
        texts = [f"passage: {t}" for t in texts]
        return super().embed_documents(texts)

    def embed_query(self, text):
        return super().embed_query(f"query: {text}")
```

### 語意拆解系統
使用專門設計的 system prompt，讓 AI 工具能夠：
- 識別複合問句中的多個子句
- 拆解成可獨立檢索的子問題
- 保持原始語意不被改寫

### 品牌大使設定
系統設定為「把夢酒館介紹給旅客的品牌大使」，具備：
- 雞尾酒專業知識
- 地方文化策展力
- 溫暖專業的回答語氣

## 目前狀況

### 已解決問題
- [x] 複合式問句拆解與檢索
- [x] 多階段檢索整合
- [x] 中文語意理解優化
- [x] Gradio 網頁界面

### 仍在改善的問題
- [ ] 內容組合錯誤（已發現但尚未完全解決）
- [ ] 檢索準確度持續優化

## 依賴套件

根據 `pyproject.toml` 配置：

### 核心依賴
- Python >= 3.13
- faiss-cpu >= 1.11.0
- gradio >= 5.38.0
- langchain >= 0.3.26
- langchain-community >= 0.3.27
- openai >= 1.97.1
- python-dotenv >= 1.0.0
- "sentence-transformers>=5.0.0"
- tqdm >= 4.67.1

### 開發依賴 (可選)
- ipykernel >= 6.30.0
- jupyter >= 1.1.1
- notebook >= 7.4.4

使用 `uv sync --extra dev` 安裝完整開發環境。

## 專案結構
```
rag-homework/
├── 酒吧RAG品牌大使.ipynb    # 主要實作內容
├── faiss_db/                # FAISS 向量資料庫
├── .env.example             # 環境變數範例
├── pyproject.toml           # 專案配置
└── README.md               # 專案說明
```

## 未來改善方向

1. 解決內容組合錯誤問題
2. 優化 chunking 策略
3. 增加評估機制
4. 模組化程式碼結構