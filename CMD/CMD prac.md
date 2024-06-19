 * 3,6,9 게임 설계
  ```
  num = int(input("숫자를 입력하세요: "))

  for i in range(1, num + 1):
      number_str = str(i)
      contains_3 = '3' in number_str
      #빈칸
      #빈칸
  
      #빈칸
      #반칸
      #반칸
      else:
          print(i)
  ```

* 함수를 활용한 계산기 제작
  ```
  def add(x, y):
    return x + y

  def subtract(x, y):
    return x - y

  #빈칸(곱하기 합수)
  #빈칸

  #빈칸(나누기 함수)
  #빈칸
  #빈칸
  #빈칸

  def calculator():
    print("계산기 실행:")
    print("1: 덧셈")
    print("2: 뺄셈")
    print("3: 곱셈")
    print("4: 나눗셈")

    #빈칸(원하는 연산식 선택)
  
    if choice in ['1', '2', '3', '4']:
       #빈칸(값 입력받기, 실수로)
       #빈칸(값 입력받기, 실수로)

        if choice == '1':
            print(f"결과: {num1} + {num2} = {add(num1, num2)}")

        #빈칸
        #빈칸

        #빈칸
        #빈칸

        #빈칸
        #빈칸
    else:
        print("잘못된 입력입니다. 다시 시도하세요.")

  # 계산기 실행
  calculator()

  ```
* 3,6,9 게임 설계
  ```
  num = int(input("숫자를 입력하세요: "))

  for i in range(1, num + 1):
      number_str = str(i)
      contains_3 = '3' in number_str
      contains_6 = '6' in number_str
      contains_9 = '9' in number_str
  
      if contains_3 or contains_6 or contains_9:
          clap_count = number_str.count('3') + number_str.count('6') + number_str.count('9')
          print("짝" * clap_count)
      else:
          print(i)
  ```

* 함수를 활용한 계산기 제작
  ```
  def add(x, y):
    return x + y

  def subtract(x, y):
    return x - y

  def multiply(x, y):
    return x * y

  def divide(x, y):
    if y == 0:
        return "오류: 0으로 나눌 수 없습니다."
    return x / y

  def calculator():
    print("계산기 실행:")
    print("1: 덧셈")
    print("2: 뺄셈")
    print("3: 곱셈")
    print("4: 나눗셈")

    choice = input("원하는 연산을 선택하세요 (1/2/3/4): ")

    if choice in ['1', '2', '3', '4']:
        num1 = float(input("첫 번째 숫자를 입력하세요: "))
        num2 = float(input("두 번째 숫자를 입력하세요: "))

        if choice == '1':
            print(f"결과: {num1} + {num2} = {add(num1, num2)}")

        elif choice == '2':
            print(f"결과: {num1} - {num2} = {subtract(num1, num2)}")

        elif choice == '3':
            print(f"결과: {num1} * {num2} = {multiply(num1, num2)}")

        elif choice == '4':
            print(f"결과: {num1} / {num2} = {divide(num1, num2)}")
    else:
        print("잘못된 입력입니다. 다시 시도하세요.")

  # 계산기 실행
  calculator()

  ```
