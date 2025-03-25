### 연수 자동화 프로그램
* 라이브 러리 설치
  ```pip install PyQt5 pyautogui opencv-python numpy pillow```

* 코드
```
import sys
import os
import time
import pyautogui
import shutil
from PyQt5.QtWidgets import (
    QMainWindow, QApplication, QPushButton, QLabel, QVBoxLayout, QWidget, 
    QFileDialog, QListWidget, QLineEdit, QMessageBox, QHBoxLayout
)
from PyQt5.QtCore import QThread, pyqtSignal
from PIL import Image

class Worker(QThread):
    status_signal = pyqtSignal(str)
    log_signal = pyqtSignal(str)
    finished_signal = pyqtSignal()

    def __init__(self, search_images, click_images, interval, parent=None):
        super().__init__(parent)
        self.search_images = search_images
        self.click_images = click_images
        self.interval = interval
        self._running = True

    def run(self):
        self.status_signal.emit("###실행중....###")
        while self._running:
            for s_img_path, c_img_path in zip(self.search_images, self.click_images):
                if not self._running:
                    break

                # PIL을 활용하여 찾을 이미지 열기
                try:
                    search_img = Image.open(s_img_path)
                except Exception as e:
                    self.log_signal.emit(f"찾을 이미지 열기 실패: {s_img_path}, {e}")
                    continue

                # 화면에서 찾을 이미지 검색 (예외 발생 시 None 처리)
                try:
                    location = pyautogui.locateOnScreen(search_img, confidence=0.8)
                except pyautogui.ImageNotFoundException:
                    location = None

                if location is not None:
                    center = pyautogui.center(location)
                    self.log_signal.emit(f"찾은 이미지: {os.path.basename(s_img_path)} 위치: {center}")
                    time.sleep(self.interval)
                    
                    # PIL을 활용하여 클릭할 이미지 열기
                    try:
                        click_img = Image.open(c_img_path)
                    except Exception as e:
                        self.log_signal.emit(f"클릭할 이미지 열기 실패: {c_img_path}, {e}")
                        continue

                    try:
                        click_location = pyautogui.locateOnScreen(click_img, confidence=0.8)
                    except pyautogui.ImageNotFoundException:
                        click_location = None

                    if click_location is not None:
                        click_center = pyautogui.center(click_location)
                        pyautogui.click(click_center)
                        self.log_signal.emit(f"클릭 완료: {os.path.basename(c_img_path)} 위치: {click_center}")
                    else:
                        self.log_signal.emit(f"클릭할 이미지 미발견: {os.path.basename(c_img_path)}")
                time.sleep(0.5)
            time.sleep(0.5)
        self.status_signal.emit("실행해주세요.")
        self.finished_signal.emit()

    def stop(self):
        self._running = False

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("자동 이미지 클릭 프로그램")
        self.search_images = []
        self.click_images = []
        self.worker = None
        self.initUI()

    def initUI(self):
        layout = QVBoxLayout()

        self.status_label = QLabel("실행해주세요.")
        layout.addWidget(self.status_label)

        # 찾을 사진 업로드 및 삭제 컨트롤
        search_layout = QHBoxLayout()
        self.btn_upload_search = QPushButton("찾을 사진 업로드")
        self.btn_upload_search.clicked.connect(self.upload_search_images)
        search_layout.addWidget(self.btn_upload_search)

        self.btn_remove_search = QPushButton("찾을 사진 삭제")
        self.btn_remove_search.clicked.connect(self.remove_search_images)
        search_layout.addWidget(self.btn_remove_search)
        layout.addLayout(search_layout)

        self.list_search = QListWidget()
        layout.addWidget(self.list_search)

        # 클릭할 사진 업로드 및 삭제 컨트롤
        click_layout = QHBoxLayout()
        self.btn_upload_click = QPushButton("클릭할 사진 업로드")
        self.btn_upload_click.clicked.connect(self.upload_click_images)
        click_layout.addWidget(self.btn_upload_click)

        self.btn_remove_click = QPushButton("클릭할 사진 삭제")
        self.btn_remove_click.clicked.connect(self.remove_click_images)
        click_layout.addWidget(self.btn_remove_click)
        layout.addLayout(click_layout)

        self.list_click = QListWidget()
        layout.addWidget(self.list_click)

        # 이미지 클릭 간격 설정
        self.interval_label = QLabel("이미지 클릭 텀(초):")
        layout.addWidget(self.interval_label)
        self.interval_input = QLineEdit()
        self.interval_input.setPlaceholderText("예: 1.5")
        layout.addWidget(self.interval_input)

        # 실행/중지 버튼
        self.btn_execute = QPushButton("실행")
        self.btn_execute.clicked.connect(self.toggle_execution)
        layout.addWidget(self.btn_execute)

        # 로그 라벨
        self.log_label = QLabel("")
        layout.addWidget(self.log_label)

        central_widget = QWidget()
        central_widget.setLayout(layout)
        self.setCentralWidget(central_widget)

        # 초기 창 크기를 가로로 넓게 (예: 800x600)
        self.resize(800, 600)

        # 업로드된 이미지 파일 저장 폴더 생성
        self.upload_search_dir = os.path.join(os.getcwd(), "uploaded_search")
        self.upload_click_dir = os.path.join(os.getcwd(), "uploaded_click")
        os.makedirs(self.upload_search_dir, exist_ok=True)
        os.makedirs(self.upload_click_dir, exist_ok=True)

    def upload_search_images(self):
        if len(self.search_images) >= 5:
            QMessageBox.warning(self, "업로드 제한", "찾을 사진은 최대 5장까지 업로드할 수 있습니다.")
            return
        files, _ = QFileDialog.getOpenFileNames(self, "찾을 사진 업로드", "", "Image Files (*.png *.jpg *.bmp)")
        if files:
            remaining = 5 - len(self.search_images)
            selected_files = files[:remaining]
            for file in selected_files:
                filename = os.path.basename(file)
                dest_path = os.path.join(self.upload_search_dir, filename)
                try:
                    shutil.copy(file, dest_path)
                    self.search_images.append(dest_path)
                    self.list_search.addItem(dest_path)
                except Exception as e:
                    QMessageBox.warning(self, "파일 복사 오류", f"{file} 복사 중 오류 발생: {e}")

    def upload_click_images(self):
        if len(self.click_images) >= 5:
            QMessageBox.warning(self, "업로드 제한", "클릭할 사진은 최대 5장까지 업로드할 수 있습니다.")
            return
        files, _ = QFileDialog.getOpenFileNames(self, "클릭할 사진 업로드", "", "Image Files (*.png *.jpg *.bmp)")
        if files:
            remaining = 5 - len(self.click_images)
            selected_files = files[:remaining]
            for file in selected_files:
                filename = os.path.basename(file)
                dest_path = os.path.join(self.upload_click_dir, filename)
                try:
                    shutil.copy(file, dest_path)
                    self.click_images.append(dest_path)
                    self.list_click.addItem(dest_path)
                except Exception as e:
                    QMessageBox.warning(self, "파일 복사 오류", f"{file} 복사 중 오류 발생: {e}")

    def remove_search_images(self):
        selected_items = self.list_search.selectedItems()
        if not selected_items:
            QMessageBox.warning(self, "삭제 오류", "삭제할 항목을 선택해주세요.")
            return
        for item in selected_items:
            path = item.text()
            if path in self.search_images:
                try:
                    if os.path.exists(path):
                        os.remove(path)
                except Exception as e:
                    QMessageBox.warning(self, "파일 삭제 오류", f"{path} 삭제 중 오류 발생: {e}")
                self.search_images.remove(path)
            self.list_search.takeItem(self.list_search.row(item))

    def remove_click_images(self):
        selected_items = self.list_click.selectedItems()
        if not selected_items:
            QMessageBox.warning(self, "삭제 오류", "삭제할 항목을 선택해주세요.")
            return
        for item in selected_items:
            path = item.text()
            if path in self.click_images:
                try:
                    if os.path.exists(path):
                        os.remove(path)
                except Exception as e:
                    QMessageBox.warning(self, "파일 삭제 오류", f"{path} 삭제 중 오류 발생: {e}")
                self.click_images.remove(path)
            self.list_click.takeItem(self.list_click.row(item))

    def toggle_execution(self):
        if len(self.search_images) != 5 or len(self.click_images) != 5:
            QMessageBox.warning(self, "업로드 오류", "찾을 사진과 클릭할 사진을 각각 5장씩 업로드해주세요.")
            return
        if self.worker is None or not self.worker.isRunning():
            try:
                interval = float(self.interval_input.text())
            except ValueError:
                QMessageBox.warning(self, "입력 오류", "이미지 클릭 텀을 올바르게 입력해주세요.")
                return
            paired_search = self.search_images[:5]
            paired_click = self.click_images[:5]
            self.worker = Worker(paired_search, paired_click, interval)
            self.worker.status_signal.connect(self.update_status)
            self.worker.log_signal.connect(self.update_log)
            self.worker.finished_signal.connect(self.worker_finished)
            self.worker.start()
            self.btn_execute.setText("중지")
        else:
            self.worker.stop()
            self.worker.wait()
            self.worker = None
            self.btn_execute.setText("실행")

    def update_status(self, text):
        self.status_label.setText(text)

    def update_log(self, message):
        current = self.log_label.text()
        self.log_label.setText(current + message + "\n")

    def worker_finished(self):
        self.btn_execute.setText("실행")


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec_())
  ```
