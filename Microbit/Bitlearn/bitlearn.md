# bitlearn.py 파일
```
# 코드 출처 : elecfreaks ringbit
# https://github.com/elecfreaks/EF_Produce_MicroPython/blob/master/Ringbit.py

from microbit import *
from time import sleep_us
from machine import time_pulse_us


class bitlearn(object):
    """기본 설명

    Ring:bit (bitlearn) 제어용 마이크로비트 확장 보드
    """

    def __init__(self, left_wheel_pin=pin1, right_wheel_pin=pin2):
        """
        bitlearn 로봇 초기화
        :param left_wheel_pin: 왼쪽 다리에 연결된 핀
        :param right_wheel_pin: 오른쪽 다리에 연결된 핀
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
        좌우 바퀴 속도 설정
        :param left_wheel_speed: 왼쪽 다리 속도 -100~100
        :param right_wheel_speed: 오른쪽 다리 속도 -100~100
        """
        if left_wheel_speed > 100 or left_wheel_speed < -100:
            raise ValueError('speed error, -100~100')
        if right_wheel_speed > 100 or right_wheel_speed < -100:
            raise ValueError('speed error, -100~100')

        if left_wheel_speed > 0:
            pwm = ((left_wheel_speed * (256 - 153.6)) / 100) + 153.6
            self.__left_wheel_pin.write_analog(int(pwm))
        elif left_wheel_speed < 0:
            pwm = ((left_wheel_speed * (51.2 - 153.6)) / -100) + 153.6
            self.__left_wheel_pin.write_analog(int(pwm))
        else:
            self.__left_wheel_pin.write_analog(int(153.6))

        right_wheel_speed = -right_wheel_speed
        if right_wheel_speed > 0:
            pwm = ((right_wheel_speed * (256 - 153.6)) / 100) + 153.6
            self.__right_wheel_pin.write_analog(int(pwm))
        elif right_wheel_speed < 0:
            pwm = ((right_wheel_speed * (51.2 - 153.6)) / -100) + 153.6
            self.__right_wheel_pin.write_analog(int(pwm))
        else:
            self.__right_wheel_pin.write_analog(int(153.6))

    def get_distance(self, unit: int = 0):
        """
        초음파 센서를 사용하여 거리 측정
        :param unit: 단위 선택 (0: 센티미터, 1: 인치)
        :return: 거리 값
        """
        self.__module_pin.read_digital()
        self.__module_pin.write_digital(1)
        sleep_us(10)
        self.__module_pin.write_digital(0)
        ts = time_pulse_us(self.__module_pin, 1, 25000)
        if ts <= 0:
            return -1  # 오류 처리

        distance = ts / 38.6  # cm 기준 계산
        if unit == 0:
            return distance
        elif unit == 1:
            return distance / 2.54  # cm -> inch 변환

    def get_tracking(self):
        """
        라인트레이서 상태 읽기
        :return: 00 - 둘 다 흰색
                 10 - 왼쪽 검정, 오른쪽 흰색
                 01 - 왼쪽 흰색, 오른쪽 검정
                 11 - 둘 다 검정
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
            print("Unknown ERROR")
            return -1


if __name__ == '__main__':
    rb = bitlearn(pin1, pin2)

    while True:
        rb.set_motors_speed(50, 50)
        sleep(1000)
        rb.set_motors_speed(0, 0)
        sleep(1000)
```
