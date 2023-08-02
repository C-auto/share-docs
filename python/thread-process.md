<span style="background-color:#fff5b1">Author : robert-min / Last Update : 2023-08-02</span>
# Python 병렬처리 고민. Multi Process? Multi Thread?

고성능 파이썬 백엔드 작업을 위해서는 병렬처리에대한 고민을 한번 쯤 해볼 수 있습니다. 일반적으로 코드 구현에만 집중하여 성능향상에 고민을 해보지 않은 개발자의 경우 병렬처리의 필요성을 경험해보지 못하거나, Process와 Thread의 개념을 이해하지 못하고 있는 경우도 종종 있습니다. 특히 파이썬의 경우 다른 언어에비해 배우기 쉽고 편의성이 좋지만, 실행 속도 측면에서 느리다는 치명적인 단점이 있기 때문에, Thread와 Process에 대한 이해를 바탕으로 Multi Process, Multi Thread 적용에대해 알고있는 것이 중요합니다.

<br>

## #1. Process? Thread? 🤦🏻‍♂️
일반적으로 CS공부를 하면 프로세스와 쓰레드를 아래처럼 암기하는 경우가 많습니다.
- Process : 컴퓨터에서 실행되고 있는 프로그램을 말하며, **운영체제에 의해 할당된 작업**
- Thread : **프로세스 내의 작업의 흐름**

  
위와 같이 암기했을 때 혼돈하기 쉽고 구체적으로 어떤 역할하는지 헷갈릴 수 있습니다. 각각의 역할의 이해하기 위해서는 우선 CPU가 멀티태스킹을 어떻게하는지 부터 확인할 필요가 있습니다. 일반적으로 우리는 컴퓨터를 할 때 파일을 다운로드 받고, 카카오톡도하며 한 번에 여러가지 일을 하며 이는 우리 눈에 보이지는 않는 여러 작업까지 포함하면 수백 개가 넘어갈 수 있습니다. 하지만 일반적으로 컴퓨터의 머리를 담당하는 CPU는 코어 당 한 번에 하나의 프로세스 즉, 하나의 일만 처리할 수 있습니다. 그에반해 실제 컴퓨터는 수 백개의 일을 거의 한 번에 보이는 것처럼 처리하며 이를 이해하기 위해서는 병렬 처리와 병행 처리 두가지 이해가 필요합니다.

- 병렬 처리 : 여러 작업을 동시에 실행하는 방법. 2개 이상의 코어가 각기 다른 프로세스의 명령을 실행해서 각 프로세스가 같은 순간에 실행되도록하는 방법
- 병행 처리 : 하나의 코어가 여러 프로세스를 조금씩 처리하는 것을 의미

즉, 컴퓨터의 코어는 각각 하나의 프로세스를 처리하여 2개의 코어를 가진 컴퓨터의 경우 한 번에 2가지 일을 병렬처리합니다. 그리고 해당 코어는 하나의 프로세스를 처리가 다 끝날 때까지 기다리는 것이 아닌 여러 프로세스를 돌아다니면서 조금씩 병행 처리를 합니다. 여기서 하나의 코어가 여러 프로세스를 병행처리하는 것을 컨택스트 스위칭이라 하며, 여러 코어가 여러개의 프로세스를 함께 진행하는 것을 멀티프로세싱이라 부릅니다.

프로세스의 병행 처리, 병렬 처리로 동시에 여러가지 일을 할 수 있지만, 아직 수백 개의 일을 처리하기에는 부족합니다. 이를 위해 프로세스를 또 나눈 단위인 스레드가 있습니다. 스레드는 한 프로세스 안에 하나 이상 진행될 수 있는 일의 단위이며 햄버거 요리를 하나의 프로세스로 생각하여 설명하면 빵을 데우고, 고기를 굽고, 야채를 써는 과정이 스레드에 의해 프로세스와 같이 컨택스트 스위칭되어 처리됩니다. 즉 스레드가 여러 스레드와 함께 병행 병렬 처리되는 것이 멀티 스레딩이라 합니다.

사실 스레드는 프로세스를 단순히 나눈 단위가 아닌 중요한 차이가 있는데 그것은 바로 "메인 메모리를 어떻게 함께 사용하는지"입니다. 멀티 프로세싱에서 각각의 프로세스는 각각 독립된 메모리 공간을 사용하며, 한번에 실행되는 프로세스가 많다는 의미는 그 만큼 많은 메인 메모리를 사용한다는 것을 의미합니다. 반면 스레드는 동일한 프로세스 내에 모든 스레드가 메모리를 공유하게 됩니다. 즉 아무리 스레드가 많아져도 메모리를 차지 않고 프로세스와 달리 메모리를 옮겨 다닐 필요가 없기 때문에 컨택스트 스위칭의 부담이 덜하는 장점이 있습니다. 이렇게 보았을 때 성능 상 스레드가 유리하지만 같은 메모리 공간을 여러 스레드가 공유해서 사용하는 만큼 발생하는 오류에 대해  추가적인 프로그래밍이 필요하다는 문제점이 있습니다.

<br>

## #2. Multi processing
멀티프로세싱은 여러 개의 프로개램이 각각의 프로세서에서 하나의 작업만 처리하는 것이 아니라 다수의 작업을 처리하며, 하나의 작업은 하나의 프로세서에 의해 처리되는 것이 아닌 다수의 프로세서에 의해 처리됩니다. 멀티 프로세스의 장점 중 하나는 여러 개의 프로세스를 처리해야하는 데 동일한 데이터가 하나의 디스크(메모리 아님!!)에 처리될 경우 모든 프로세스가 이를 공유하게 되면 비용적으로 저렴해집니다.(프로세스는 동일한 메모리를 공유하지 않기 때문에 메모리를 공유하는 작업은 오히려 더 큰 비용이 발생합니다.) 또한, 하나의 프로세스가 하나의 작업만 처리한다면 특정 프로세스에 장애가 날 경우 다른 프로세스가 해당 작업을 처리하기 때문에 작업이 정지되는 문제를 방지할 수 있습니다.

### Multiprocessing 모듈로 멀티프로세싱 구현
Multi processing과 Multi threading을 비교하기 위해 1에서 100000000까지 더하는 함수를 각각 두개의 프로세스와 쓰레드로 실행하는 코드를 작성합니다. Multiprocess는 파이썬 multiprocessing 라이브러리를 사용했습니다.

- [공식문서](https://docs.python.org/ko/3/library/multiprocessing.html)

```python
import multiprocessing
import time

def calculate_sum(start, end):
    result = 0
    for num in range(start, end + 1):
        result += num
    return result

def calculate_total_sum_parallel():
    start_time = time.time()
    total = 0
    num_processes = 2

    # Calculate the range for each process
    total_range = 100000000
    chunk_size = total_range // num_processes
    ranges = [(i * chunk_size + 1, (i + 1) * chunk_size) for i in range(num_processes)]

    # Create a pool of processes
    pool = multiprocessing.Pool(processes=num_processes)

    # Use map to distribute the workload to processes
    results = pool.starmap(calculate_sum, ranges)

    # Combine the results from different processes
    total = sum(results)

    pool.close()
    pool.join()

    end_time = time.time()
    execution_time = end_time - start_time

    return total, execution_time

if __name__ == "__main__":
    total_sum, execution_time = calculate_total_sum_parallel()
    print(f"Total sum: {total_sum}")
    print(f"Execution time: {execution_time:.4f} seconds")
```

- 실행 결과 : Execution time: **5.8471** seconds

<br>

## #3. Multi threading
멀티쓰레딩은 하나의 프로세스 아래의 여러 쓰레드가 하나의 데이터 자원을 공유하며 하나의 작업을 여러 쓰레드에 의해 처리됩니다. 여러 개의 쓰레드는 하나의 데이터 자원(메모리)를 공유하기 때문에 메모리에 대한 효율성을 가질 수 있습니다.

### concurrent.futures 모듈로 Multi Threading 구현
멀티 쓰레딩을 구현할 수 있는 여러 모듈 중 concurrent.futures를 사용하여 멀티프로세싱과 동일한 역할을 하는 예시 구현
```python
import concurrent.futures
import time

def calculate_sum(start, end):
    result = 0
    for num in range(start, end + 1):
        result += num
    return result

def calculate_total_sum_parallel():
    start_time = time.time()
    total = 0
    num_threads = 2

    # Calculate the range for each thread
    total_range = 100000000
    chunk_size = total_range // num_threads
    ranges = [(i * chunk_size + 1, (i + 1) * chunk_size) for i in range(num_threads)]

    # Create a ThreadPoolExecutor with the desired number of threads
    with concurrent.futures.ThreadPoolExecutor(max_workers=num_threads) as executor:
        # Use submit to schedule each task (calculate_sum) in the executor
        futures = [executor.submit(calculate_sum, start, end) for start, end in ranges]

        # Use as_completed to get the results as the tasks complete
        for future in concurrent.futures.as_completed(futures):
            total += future.result()

    end_time = time.time()
    execution_time = end_time - start_time

    return total, execution_time

if __name__ == "__main__":
    total_sum, execution_time = calculate_total_sum_parallel()
    print(f"Total sum: {total_sum}")
    print(f"Execution time: {execution_time:.4f} seconds")
```

- 실행 결과 : Execution time: **6.6625** seconds

## #4. Multi processing, Multi threading 결과비교 
값을 계속 result 라는 변수에 할당하여 메모리를 사용하는 작업으로 생각해 멀티프로세싱보다 멀티쓰레딩이 더 빠르게 동작할 것이라 생각할 수 있습니다. 하지만 실제 결과를 보면 멀티프로세싱이 멀티쓰레딩보다 더 빠르게 동작한 것을 확인할 수 있습니다. 해당 결과에는 원인은 **파이썬 GIL(Global Interpreter Lock) 정책으로 멀티 쓰레드를 구성했지만 실제 하나의 쓰레드로만 해당 동작해 단일쓰레도 처리했기 때문**입니다. 파이썬은 하나의 프로세스 안에 모든 자원의 락을 글로벌하게 관리함으로써 한 번에 하나의 쓰레드만 자원을 컨트롤 하기 때문에 사실한 result라는 메모리에 저장된 변수는 하나의 쓰레드로만 접근하게 됩니다. 그에 반해 멀티프로세싱은 각각의 고유한 메모리 영역을 가지고 있어 쓰레드에 비해 메모리 사용은 크지만 두개의 프로세스로 처리해 실행시간이 현저히 감소하게 됩니다. 

<br>

## #5. 결론
위에 결과로 보았을 때 결과적으로 파이썬 정책 때문에 멀티쓰레드는 사실상 제대로된 역할을 하지 못하는 것이라 생각할 수 있습니다. 멀티 쓰레드가 파이썬에서 메모리 접근에 제한이 있지만, 메모리에 접근하는 작업이 적고 디스크 등의 I/O 작업이 많은 병렬처리 작업에서는 쓰레드가 가볍기 때문에 더 높은 성능을 보일 수 있습니다. 또, 멀티 프로세스는 메모리에대한 분산 처리로 속도에 이점을 볼 수 있지만 더 많은 메모리를 필요하다는 단점이 존재합니다. 즉, **파이썬에서 병렬처리 작업 시에는 해당 작업이 어떤 작업을 처리하고 있는지 고민해보고 적절한 병렬처리를 사용하는 것이 중요합니다.**

<br>


### #참고. Process 구조
프로세스는 동적 영역인 Stack, Heap과 정적 영역인 데이터 영역(Bss segment, Data segment), 코드 영역(Code segment)로 구분됩니다. 동적 영역인 스탭은 위에서부터 메모리가 할당되며, 힙은 아래에서 부터 사용되지 않는 메모리에 할당됩니다.

<p align="center"><img src="https://github.com/C-auto/share-docs/assets/91866763/f6d760a6-cde1-4840-9c86-a9fa8e5bc033" width="600"></p>

- 스택 : 지역변수, 매개변수, 함수가 저장되며 컴파일 시에 크기가 결정.
- 힙 : 코드에서 동적으로 생성된 데이터가 저장되며, 동적 할당할 때 사용되며 런타임 시 크기가 결정.
- 데이터 영역 : 전역변수, 정적변수가 저장되며 정적인 특징을 가진 프로그램이 종료되면 사라지는 변수가 들어 있는 영역
  - 데이터 영역 중 BSS는 초기화 되지 않는 변수가 0으로 초기화되어 저장되며, Data 영역은 0이 아닌 다른 값으로 할당된 변수들이 저장
- 코드 영역 : 프로그램에 내장되어 있는 소스 코드가 들어가는 영역. 이 영역은 수정 불가능한 기계어로 저장되어 있으며 정적인 특징을 가짐. 


