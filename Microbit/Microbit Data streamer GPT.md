# Microbit Data Stramer GPT 프로그램
* GPT API KEY가 필요합니다.
* python 3.13.2 버젼에서 설계하였습니다.
* 작동 매커니즘은 아래와 같습니다.
   * GPT API KEY를 입력합니다.
   *  [연결] 버튼을 누릅니다.
   *  챗봇과 대화가 가능한 상태가 됩니다.
   *  전처리 된 엑셀파일을 업로드합니다.
   *  Data Steamer로 입력된 데이터인 조도, 온도, 사운드 값만 그래프로 그릴 수 있습니다.
   *  마이크로비트, 아두이노 상관없습니다.
   *  전처리는 time | light | temp | sound 가 1행이어야 하며,
   *  해당 열에는 데이터 값이 깔끔하게 기록되어 있어야 합니다.
   *  전처리 시 100개 정도의 데이터로 이상치 값을 수정하는 것을 추천합니다.
   *  그래프를 3개의 종류로 그릴 수 있으며, 그래프 그린 후, GPT분석을 누르면 분석 결과가 나옵니다.
   *  결과를 바탕으로 대화를하며 분석 도움을 받을 수 있습니다.
* 단순 실습 용이며, 전문적인 분석 보다는 '오프라인'상의 데이터를 수집하고, 인공지능의 도움을 받아 분석을 수행하는 과정을 교육하기 위한 프로그램입니다.

  ---
  # Learnmore Microbit Data Analytics Program
  ## 라이브러리 설치
  ```
  pip install PyQt5 PyQtWebEngine pandas openpyxl requests plotly

  ```

  ## Program code
  ```
  import sys
  import os
  import requests
  import pandas as pd
  from PyQt5.QtWidgets import (
      QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout,
      QLabel, QTextEdit, QLineEdit, QPushButton, QFileDialog, QMessageBox, QComboBox
  )
  from PyQt5.QtCore import QThread, pyqtSignal, QTimer
  from PyQt5.QtWebEngineWidgets import QWebEngineView
  import plotly.graph_objects as go
  
  class AnalysisThread(QThread):
      result_ready = pyqtSignal(str)
  
      def __init__(self, api_key, prompt):
          super().__init__()
          self.api_key = api_key
          self.prompt = prompt
  
      def run(self):
          url = "https://api.openai.com/v1/chat/completions"
          headers = {
              "Authorization": f"Bearer {self.api_key}",
              "Content-Type": "application/json"
          }
          data = {
              "model": "gpt-4o-mini",
              "messages": [
                  {"role": "system",  "content": "당신은 데이터 분석 전문가입니다."},
                  {"role": "user",    "content": self.prompt}
              ]
          }
          try:
              resp = requests.post(url, headers=headers, json=data)
              resp.raise_for_status()
              result = resp.json()['choices'][0]['message']['content']
          except Exception as e:
              result = f"API 오류: {e}"
          self.result_ready.emit(result)
  
  class ChatWindow(QMainWindow):
      def __init__(self):
          super().__init__()
          self.setWindowTitle("Learnmore Microbit Data Analytics program")
          self.setGeometry(100, 100, 900, 700)
          self.api_key = ""
          self.df = None
          self.last_figure = None
          self.analysis_thread = None
  
          # 애니메이션용 변수
          self.loading_timer = QTimer(self)
          self.loading_timer.setInterval(500)  # 0.5초마다
          self.loading_timer.timeout.connect(self.update_loading_animation)
          self.loading_dots = 0
  
          self.initUI()
  
      def initUI(self):
          central = QWidget(self)
          self.setCentralWidget(central)
          layout = QVBoxLayout(central)
  
          # API 키 입력 영역
          api_layout = QHBoxLayout()
          api_layout.addWidget(QLabel("API 키:"))
          self.api_input = QLineEdit()
          api_layout.addWidget(self.api_input)
          btn_api = QPushButton("연결")
          btn_api.clicked.connect(self.confirm_api_key)
          api_layout.addWidget(btn_api)
          self.conn_label = QLabel("API 미연결")
          api_layout.addWidget(self.conn_label)
          layout.addLayout(api_layout)
  
          # 챗봇 대화창
          layout.addWidget(QLabel("챗봇 대화"))
          self.chat = QTextEdit()
          self.chat.setReadOnly(True)
          layout.addWidget(self.chat)
  
          # 상태 메시지 레이블 (말줄임표 애니메이션)
          self.status_label = QLabel("")
          layout.addWidget(self.status_label)
  
          # 메시지 입력 및 전송
          msg_layout = QHBoxLayout()
          self.msg_input = QLineEdit()
          btn_send = QPushButton("전송")
          btn_send.clicked.connect(self.send_message)
          msg_layout.addWidget(self.msg_input)
          msg_layout.addWidget(btn_send)
          layout.addLayout(msg_layout)
  
          # 엑셀 업로드 및 옵션
          data_layout = QHBoxLayout()
          btn_load = QPushButton("엑셀 업로드")
          btn_load.clicked.connect(self.load_excel)
          data_layout.addWidget(btn_load)
          data_layout.addWidget(QLabel("측정값 선택:"))
          self.metric_cb = QComboBox()
          data_layout.addWidget(self.metric_cb)
          data_layout.addWidget(QLabel("그래프 종류:"))
          self.graph_cb = QComboBox()
          self.graph_cb.addItems(["꺾은선", "막대 그래프", "히스토그램"])
          data_layout.addWidget(self.graph_cb)
          btn_plot = QPushButton("그리기")
          btn_plot.clicked.connect(self.draw_plot)
          data_layout.addWidget(btn_plot)
          btn_analyze = QPushButton("GPT 분석")
          btn_analyze.clicked.connect(self.analyze_plot)
          btn_analyze.setEnabled(False)
          data_layout.addWidget(btn_analyze)
          layout.addLayout(data_layout)
  
          # Plotly 웹뷰
          self.webview = QWebEngineView()
          layout.addWidget(self.webview, stretch=1)
  
          # 버튼 초기화
          btn_send.setEnabled(False)
          btn_load.setEnabled(False)
          btn_plot.setEnabled(False)
          self.send_btn = btn_send
          self.load_btn = btn_load
          self.plot_btn = btn_plot
          self.analyze_btn = btn_analyze
  
      def confirm_api_key(self):
          key = self.api_input.text().strip()
          if not key:
              QMessageBox.warning(self, "경고", "API 키를 입력해주세요.")
              return
          self.api_key = key
          self.conn_label.setText("API 연결 완료")
          self.chat.append("System: API 연결이 완료되었습니다.")
          self.send_btn.setEnabled(True)
          self.load_btn.setEnabled(True)
          self.plot_btn.setEnabled(True)
  
      def send_message(self):
          txt = self.msg_input.text().strip()
          if not txt:
              return
          self.chat.append("User: " + txt)
          self.msg_input.clear()
          thread = AnalysisThread(self.api_key, txt)
          thread.result_ready.connect(lambda res: self.chat.append("GPT: " + res))
          thread.start()
  
      def load_excel(self):
          path, _ = QFileDialog.getOpenFileName(self, "엑셀 선택", "", "Excel Files (*.xlsx *.xls)")
          if not path:
              return
          try:
              df = pd.read_excel(path)
          except Exception as e:
              QMessageBox.critical(self, "오류", f"엑셀 로드 실패: {e}")
              return
          if 'time' in df.columns:
              df['시간'] = pd.to_datetime(df['time'])
          else:
              QMessageBox.warning(self, "경고", "time 컬럼이 존재하지 않습니다.")
              return
          if len(df.columns) >= 4:
              df.rename(columns={df.columns[1]: 'temp', df.columns[2]: 'light', df.columns[3]: 'sound'}, inplace=True)
          else:
              QMessageBox.warning(self, "경고", "데이터가 충분하지 않습니다 (4개 열 필요)")
              return
          self.df = df
          self.metric_cb.clear()
          for col in ['temp', 'light', 'sound']:
              if col in df.columns:
                  self.metric_cb.addItem(col)
          self.chat.append(f"System: 데이터 로드 완료 ({df.shape[0]}×{df.shape[1]})")
  
      def draw_plot(self):
          if self.df is None:
              QMessageBox.warning(self, "경고", "먼저 엑셀을 업로드하세요.")
              return
          metric = self.metric_cb.currentText()
          gtype = self.graph_cb.currentText()
          fig = go.Figure()
          if gtype == "꺾은선":
              fig.add_trace(go.Scatter(x=self.df['시간'], y=self.df[metric], mode='lines+markers', name=metric))
          elif gtype == "막대 그래프":
              fig.add_trace(go.Bar(x=self.df['시간'], y=self.df[metric], name=metric))
          else:
              fig = go.Figure(go.Histogram(x=self.df[metric], name=metric))
          fig.update_layout(
              plot_bgcolor='lightgray',
              xaxis=dict(showgrid=True, gridcolor='white'),
              yaxis=dict(showgrid=True, gridcolor='white'),
              title=f"{metric} {gtype} 데이터",
              xaxis_title='시간' if gtype != '히스토그램' else metric,
              yaxis_title=metric if gtype != '히스토그램' else '빈도',
              hovermode='x unified'
          )
          if gtype != '히스토그램':
              y_max = self.df[metric].max() + 10
              fig.update_yaxes(range=[0, y_max])
          self.last_figure = fig
          self.webview.setHtml(fig.to_html(include_plotlyjs='cdn'))
          self.chat.append("GPT: 그래프를 성공적으로 불러와 이해했습니다.")
          self.analyze_btn.setEnabled(True)
  
      def analyze_plot(self):
          if self.df is None:
              QMessageBox.warning(self, "경고", "먼저 엑셀을 업로드하세요.")
              return
  
          # 상태 메시지 초기화 및 애니메이션 시작
          self.loading_dots = 0
          self.status_label.setText("System: 데이터 요약 전송 중")
          self.loading_timer.start()
  
          metric = self.metric_cb.currentText()
          times  = self.df['시간'].dt.strftime('%Y-%m-%d %H:%M:%S').tolist()
          values = self.df[metric].tolist()
          data_text = (
              "시간: [" + ", ".join(times) + "]\n"
              f"{metric} 값: [" + ", ".join(map(str, values)) + "]"
          )
          prompt = (
              "다음은 마이크로비트로 측정한 데이터의 요약입니다.\n"
              f"{data_text}\n\n"
              "이 데이터를 기반으로 중학교 3학년~고등학교 2학년 수준에서 이해할 수 있는 "
              "언어로 변화 추세를 분석하고, 교육적 가치를 포함하여 설명해주세요."
              "설명시 사용하면 안되는 기호는 * 입니다. **기호를 사용하여 강조하지 마세요."
              "설명은 1~7까지의 개조식으로 깔끔하고, 학술적인 문체로 나오도록 튜닝해주세요."
          )
  
          self.analysis_thread = AnalysisThread(self.api_key, prompt)
          self.analysis_thread.result_ready.connect(self.on_analysis_result)
          self.analysis_thread.start()
  
      def update_loading_animation(self):
          # 0~3까지 점 개수 순환
          self.loading_dots = (self.loading_dots + 1) % 4
          dots = "." * self.loading_dots
          self.status_label.setText(f"System: 데이터 요약 전송 중{dots}")
  
      def on_analysis_result(self, result):
          # 애니메이션 중지 및 상태 메시지 클리어
          self.loading_timer.stop()
          self.status_label.clear()
  
          # 결과를 채팅창에 출력
          self.chat.append("GPT: " + result)
  
  if __name__ == "__main__":
      app = QApplication(sys.argv)
      window = ChatWindow()
      window.show()
      sys.exit(app.exec_())
  ```
