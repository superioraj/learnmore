# GPT Cost 비용을 계산한느 프로그램
* 아래 코드 확인 후 라이브러리 설치
* https://www.koreaexim.go.kr/index 한국 수출입 은행의 환율 API키 발급
* Openai Gpt Model 별 코스트 학습(본 프로그램은 4.1 mini | 4.1 nano | 4o mini 로 Cost가 작은 모델로 확인)
* 학생들이 GPT와 대화시 얼마 만큼의 원화를 소비하는지 확인하기 위한 프로그램 입니다. 
* 단순 교육용입니다. 
* developed by Learnmore & GPT

# Code
```
import sys
import requests
import tiktoken
import webbrowser
from datetime import datetime, timedelta
from PyQt5.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QHBoxLayout,
    QLabel, QTextEdit, QComboBox, QPushButton, QInputDialog
)

# 한국수출입은행 Open API 인증키 및 URL
EXIM_AUTH_KEY = "jpzHerEM9Jln4bfqonJSUP5CC05Vnzms"
# HTTPS, HTTP 두 가지 프로토콜을 시도
EXIM_API_URL_HTTPS = "https://www.koreaexim.go.kr/site/program/financial/exchangeJSON"
EXIM_API_URL_HTTP  = "http://www.koreaexim.go.kr/site/program/financial/exchangeJSON"

# 모델별 토큰당 요금 (USD / 1M tokens)
MODEL_PRICES = {
    "gpt-4.1-mini": 0.40,
    "gpt-4.1-nano": 0.10,
    "gpt-4o-mini": 0.15,
}

# tiktoken 클라이언트 초기화 (cl100k_base)
ENC = tiktoken.get_encoding("cl100k_base")

def count_tokens(text: str) -> int:
    """입력 텍스트를 토큰 수로 변환"""
    return len(ENC.encode(text))


def fetch_exim_rate() -> float:
    """
    한국수출입은행 Open API (AP01) 호출
    HTTPS 실패 시 HTTP로 재시도하여 전날 기준일자 USD 매매기준율(deal_bas_r) 반환
    문자열에 포함된 쉼표(,)를 제거하여 float 변환
    """
    search_date = (datetime.now() - timedelta(days=1)).strftime("%Y%m%d")
    params = {
        "authkey": EXIM_AUTH_KEY,
        "searchdate": search_date,
        "data": "AP01"
    }
    last_exc = None
    for url, verify in [(EXIM_API_URL_HTTPS, True), (EXIM_API_URL_HTTP, False)]:
        try:
            resp = requests.get(url, params=params, timeout=10, verify=verify)
            resp.raise_for_status()
            data = resp.json()
            for entry in data:
                if entry.get("cur_unit") == "USD":
                    rate_str = entry.get("deal_bas_r", "0").replace(',', '')
                    return float(rate_str)
            raise RuntimeError("USD data not found in Exim API response")
        except Exception as e:
            last_exc = e
    raise RuntimeError(f"Exchange rate fetch failed: {last_exc}")

class TokenCostCalculator(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("GPT Token & Cost Calculator")
        self.current_rate = None
        self._build_ui()

    def _build_ui(self):
        layout = QVBoxLayout()

        # 모델 선택
        h_model = QHBoxLayout()
        h_model.addWidget(QLabel("Select Model:"))
        self.model_combo = QComboBox()
        self.model_combo.addItems(MODEL_PRICES.keys())
        h_model.addWidget(self.model_combo)
        layout.addLayout(h_model)

        # 문장 입력
        layout.addWidget(QLabel("Enter Text:"))
        self.text_edit = QTextEdit()
        layout.addWidget(self.text_edit)

        # 버튼 영역
        h_buttons = QHBoxLayout()
        self.btn_manual = QPushButton("Manual")
        self.btn_auto   = QPushButton("Auto")
        self.btn_calc   = QPushButton("Calculate")
        self.btn_pricing = QPushButton("Open Pricing")
        for btn in (self.btn_manual, self.btn_auto, self.btn_calc, self.btn_pricing):
            h_buttons.addWidget(btn)
        layout.addLayout(h_buttons)

        # 결과 표시
        self.result_label = QLabel("")
        layout.addWidget(self.result_label)

        self.setLayout(layout)

        # 시그널 연결
        self.btn_manual.clicked.connect(self.on_manual)
        self.btn_auto.clicked.connect(self.on_auto)
        self.btn_calc.clicked.connect(self.on_calc)
        self.btn_pricing.clicked.connect(self.open_pricing)

    def on_manual(self):
        rate, ok = QInputDialog.getDouble(self, "Manual Rate Input", "1 USD = ? KRW:", decimals=2)
        if ok:
            self.current_rate = rate
            self.result_label.setText(f"Manual rate set: 1 USD = {rate:.2f} KRW")

    def on_auto(self):
        """Exim API 실패 시 HTTP로 폴백, 전날 기준일자 함께 표시"""
        # 계산에 사용된 날짜
        rate_date = (datetime.now() - timedelta(days=1)).strftime("%Y-%m-%d")
        try:
            rate = fetch_exim_rate()
            self.current_rate = rate
            self.result_label.setText(
                f"Auto fetched Exim rate ({rate_date}): 1 USD = {rate:.2f} KRW"
            )
        except Exception as e:
            self.result_label.setText(f"Exchange rate fetch failed: {e}")

    def on_calc(self):
        text = self.text_edit.toPlainText().strip()
        if not text:
            self.result_label.setText("Please enter some text.")
            return

        tokens = count_tokens(text)
        model = self.model_combo.currentText()
        usd_cost = tokens / 1_000_000 * MODEL_PRICES[model]

        if self.current_rate is None:
            self.result_label.setText("Please fetch exchange rate first.")
            return

        krw_cost = usd_cost * self.current_rate
        self.result_label.setText(
            f"Tokens: {tokens}\n"
            f"USD Cost: ${usd_cost:.6f}\n"
            f"KRW Cost: ₩{krw_cost:,.0f}\n"
            f"(Exim Base Rate {rate_date}: 1 USD = {self.current_rate:.2f} KRW)"
        )

    def open_pricing(self):
        webbrowser.open("https://platform.openai.com/docs/pricing")

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = TokenCostCalculator()
    window.show()
    sys.exit(app.exec_())

```
