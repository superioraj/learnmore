# Ringbit Python Samples Code
* [Ringbit 도규먼트](https://wiki.elecfreaks.com/en/microbit/microbit-smart-car/microbit-ringbit-car-v2/ring-bit-python/)
* [마이크로 비트 파이썬 데디터 연결](https://python.microbit.org/v/3/project)
* Project 클릭 - main.py에 아래 코드 복사
```
from microbit import *
from time import sleep_us, sleep
from machine import time_pulse_us

class RINGBIT(object):
    """기본 설명
    링비트(Ring:bit) 메인 보드를 제어하는 클래스입니다.
    """

    def __init__(self, left_wheel_pin=pin1, right_wheel_pin=pin2):
        """
        링비트 로봇을 초기화하는 함수입니다.
        :param left_wheel_pin: 로봇의 왼쪽 바퀴 서보 모터 연결 핀
        :param right_wheel_pin: 로봇의 오른쪽 바퀴 서보 모터 연결 핀
        """
        i2c.init()
        self.__left_wheel_pin = left_wheel_pin
        self.__right_wheel_pin = right_wheel_pin
        self.__right_wheel_pin.set_analog_period(10)
        self.__left_wheel_pin.set_analog_period(10)
        self.__module_pin = pin0
        if self.__left_wheel_pin != pin1 and self.__right_wheel_pin != pin1:
            self.__module_pin = pin1
        if self.__left_wheel_pin != pin2 and self.__right_wheel_pin != pin2:
            self.__module_pin = pin2

    def set_motors_speed(self, left_wheel_speed: int, right_wheel_speed: int):
        """
        좌우 바퀴 전동기의 속도를 설정하는 함수입니다.
        :param left_wheel_speed: 왼쪽 바퀴 속도 (-100 ~ 100)
        :param right_wheel_speed: 오른쪽 바퀴 속도 (-100 ~ 100)
        :return: 없음
        """
        if left_wheel_speed > 100 or left_wheel_speed < -100:
            raise ValueError('속도 오류: -100에서 100 사이의 값을 입력하세요.')
        if right_wheel_speed > 100 or right_wheel_speed < -100:
            raise ValueError('속도 오류: -100에서 100 사이의 값을 입력하세요.')
        if left_wheel_speed > 0:
            left_wheel_speed = ((left_wheel_speed - 0) * (256 - 153.6)) / (100 - 0) + 153.6
            self.__left_wheel_pin.write_analog(left_wheel_speed)
        elif left_wheel_speed < 0:
            left_wheel_speed = ((left_wheel_speed - 0) * (51.2 - 153.6)) / (-100 - 0) + 153.6
            self.__left_wheel_pin.write_analog(left_wheel_speed)
        else:
            self.__left_wheel_pin.write_analog(153.6)

        right_wheel_speed = right_wheel_speed * -1
        if right_wheel_speed > 0:
            right_wheel_speed = ((right_wheel_speed - 0) * (256 - 153.6)) / (100 - 0) + 153.6
            self.__right_wheel_pin.write_analog(right_wheel_speed)
        elif right_wheel_speed < 0:
            right_wheel_speed = ((right_wheel_speed - 0) * (51.2 - 153.6)) / (-100 - 0) + 153.6
            self.__right_wheel_pin.write_analog(right_wheel_speed)
        else:
            self.__right_wheel_pin.write_analog(153.6)

    def get_distance(self, unit: int = 0):
        """
        초음파 센서를 이용하여 거리를 측정하는 함수입니다.
        :param unit: 측정 단위 (0: 센티미터, 1: 피트)
        :return: 측정된 거리
        """
        self.__module_pin.read_digital()
        self.__module_pin.write_digital(1)
        sleep_us(10)
        self.__module_pin.write_digital(0)
        ts = time_pulse_us(self.__module_pin, 1, 25000)

        distance = ts * 9 / 6 / 58
        if unit == 0:
            return distance
        elif unit == 1:
            return distance / 254

    def get_tracking(self):
        """
        라인 트래킹 센서의 현재 상태를 반환하는 함수입니다.
        :return: 00: 모두 흰색, 10: 왼쪽 검정/오른쪽 흰색, 01: 왼쪽 흰색/오른쪽 검정, 11: 모두 검정색
        """
        val = self.__module_pin.read_analog()
        if val < 150:
            return 11
        elif 150 <= val < 235:
            return 10
        elif 235 <= val < 300:
            return 1
        elif 300 <= val < 600:
            return 0
        else:
            print("알 수 없는 오류 발생")

if __name__ == '__main__':
    rb = RINGBIT(pin1, pin2)
    while True:
        rb.set_motors_speed(50, 50)
        sleep(1000)
        rb.set_motors_speed(0, 0)
        sleep(1000)
'''

