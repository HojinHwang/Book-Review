# 기본편
- Airflow 파이프라인은 DAG 파일에 파이썬 코드로 DAG를 정의함.
- 각 DAG 파일은 서로 다른 Task와 해당 의존성을 기술하는 하나의 DAG에 대해 정의하고 Airflow가 실행되는 시기와 주기도 정의함.
- 백필(backfilling): 하나의 플로(Airflow는 DAG)를 특정 옵션(기간) 기준으로 다시 실행할 수 있는 기능. 태스크가 며칠 동안 실패하거나 새롭게 만든 플로를 과거의 특정 시점부터 순차적으로 실행하고 싶을 때 수행함.
  - Airflow 3가지 구성 요소
  - Airflow Scheduler: DAG를 분석하고 현재 시점에서 DAG의 스케줄이 지난 경우 Airflow 워커에 DAG의 Task를 예약함.
  - Airflow Worker: 예약된 태스크를 실행함.
  - Airflow Web Server: 스케줄러에서 분석한 DAG를 시각화하고 DAG 실행과 결과를 확인할 수 있는 주요 인터페이스를 제공함.
- Airflow Scherduler 작업 진행 단계
  1. 사용자가 DAG 워크플로를 작성하면, 스케줄러는 DAG 파일을 분석하고 각 DAG Task, 의존성 및 예약 주기를 확인함.
  2. 스케줄러는 마지막 DAG까지 내용을 확인한 후, DAG의 예약 주기가 경과했는지 확인함. 예약 주기가 현재 시간 이전이면 실행되도록 예약함.
  3. 예약된 각 Task에 대해 스케줄러는 해당 Task의 의존성(=업스트림 태스크)을 확인함. 의존성 Task가 완료되지 않았다면 실행 대기열에 추가함
  4. 스케줄러는 1단계로 다시 돌아간 후, 새로운 루프를 잠시동안 대기함.
- Airflow는 태스크 실패 시에 재시도(재실행 시간의 간격 설정도 가능) 할 수 있기 때문에 오류 발생 시에 태스크를 복구할 수 있다.
- Airflow의 스케줄 기능 중 강력한 것은 DAG에 정의된 특정 시점에 트리거할 수 있을 뿐만 아니라 최종 시점과 예상되는 다음 스케줄 주기를 상세하게 알려주는 것이다.
    - 이를 통해, 각각의 주기로 나누고 각 주기별로 DAG를 실행할 수 있음.
- 스케줄 주기를 백필 개념과 결합하여 새로 생성한 DAG를 과거 시점 및 기간에 대해 실행이 가능함.
    - 이 기능을 통해, 과거 특정 기간에 대해 DAG를 실행해 새로운 데이터 세트를 생성할 수 있음.
    - 과거 실행 결과를 삭제한 후, 태스크 코드를 변경한 후에 삭제된 과거 태스크를 재실행 가능함.
- Airflow를 선택하는 이유
  - Airflow가 배치 지향 데이터 파이프라인을 구현하는 데 적합함
    - 파이썬 기반의 Airflow는 쉽게 확장 가능하고 다양한 시스템과 통합이 가능함.
    - 파이썬 언어에서 구현할 수 있는 대부분의 방법을 사용하여 복잡한 커스텀 파이프라인 구현 가능
    - 수많은 스케줄링 기법이 파이프라인을 정기적으로 실행하고 점진적(증분) 처리를 통해, 전체 파이프라인을 재실행할 필요 없는 효율적인 파이프라인 구축 가능
    - 백필 기능을 사용해 과거 데이터를 손쉽게 재처리 가능하기 때문에 코드를 변경한 후, 데이터 재생성이 가능
    - Airflow의 훌륭한 웹 UI가 파이프라인 실행 결과를 모니터링 할 수 있고 오류를 디버깅하기 위한 훌륭한 뷰를 제공해줌.
- Airflow가 적합하지 않은 경우
  - 스트리밍 워크플로 및 해당 파이프라인 처리에 부적합
  - 추가 및 삭제 태스크가 빈번한 동적 파이프라인에 부적합함. 웹 인터페이스는 DAG의 가장 최근 실행 버전에 대한 정의만 표현해주기 때문에
  - 파이썬 경험이 없는 팀
  - 파이썬 코드가 복잡해질 수 있으므로 초기 사용 시점부터 엄격한 관리 필요함

### Chapter2. Airflow DAG 구조
- DAG 클래스는 2개의 인수가 필요
    - Airflow UI에 표시되는 DAG 이름
    - 워크플로가 처음 실행되는 날짜/시간
- 각 오퍼레이터는 하나의 태스크를 수행하고 여러개의 오퍼레이터가 Airflow의 워크플로 또는 DAG를 구성함
- 오퍼레이터는 서로 독립적으로 실행 가능하나 순서를 정의해 실행할 수도 있고 이를 의존성이라고 부름
- 태스크 실패 시, 그래프 뷰와 트리 뷰에 모두 빨간색으로 표시됨
- 앞 태스크의 의존성에 의한 태스크 실패 시, 주황색으로 표시됨
- 실패한 태스크를 클릭한 후, 팝업에서 Clear를 누르면, 초기화된 태스크가 표시되고 Airflow가 이 태스크를 재실행함
- DAG 안에 있는 태스크는 어디에 위치하든 재시작 가능함

### Chapter3. Airflow의 스케줄링

- Airflow는 시작 날짜를 기준으로 첫 번째 DAG의 실행을 스케줄(시작 날짜+간격)하고 그 다음 실행은 첫 번째 스케줄된 간격 이후, 계속해서 해당 간격으로 실행함.
    - **ex) start_date = 2024-01-01이고 간격을 @daily (매일 자정)로 2024년 1월1일 오후 3시에 개발한 후 실행했다면, 2024년 1월 2일 00시가 되기 전까지는 DAG가 작업을 하지 않음**
- Airflow는 cron과 동일한 구문을 사용해 스케줄 간격을 정의할 수 있음
- cron 5개의 구성요소
    - * (분: 0-59) * (시간: 0-23) * (일: 1-31) * (월: 1-12) * (요일: 0-6)
        - 0 * * * * = 매시간(정시에 실행)
        - 0 0 * * * = 매일(자정에 실행)
        - 0 0 * * 0 = 매주(일요일 자정에 실행)
        - 0 0 * * MON, WED, FRI = 매주 월, 화, 금 자정에 실행
        - 0 0 * * MON-FRI = 매주 월~금 자정에 실행
        - 0 0,12 * * * = 매일 자정 및 오후 12시에 실행
- Airflow는 스케줄 간격을 의미하는 약어를 사용한 몇 가지 매크로도 존재
    - @once: 1회만 실행
    - @hourly: 매시간 실행
    - @daily: 매일 자정에 실행
    - @weekly: 매주 일요일 자정에 1회 실행
    - @monthly: 매월 1일 자정에 1회 실행
    - @yearly: 매년 1월 1일 자정에 1회 실행
- cron 식은 특정 빈도마다 스케줄 정의를 할 수 없다는 제약점을 가짐
- 빈도 기반 스케줄을 사용하려면 timedelta(datetime 모듈에 있는) 인스턴스를 사용하면 됨

```python
import datetime as dt

dag=DAG(
	 dag_id = "04_timedelta"
	,schedule_interval = dt.timedelta(days=3) # 3일마다 실행됨
	,start_date = dt.datetime(year=2024, month=1, day=1)
	,end_date = dt.datetime(year=2024, month=1, day=5)	
	)
```
- 과거 전체 데이터를 가져오지 않고 증분 추출을 통해, 특정 기간에 대한 데이터만 가져올 수 있음
- 증분 방식은 데이터 양을 크게 줄이기 때문에 효율적임
- Airflow는 태스크가 실행되는 특정 간격을 정의할 수 있는 매개변수를 제공함
    - execution_date: 스케줄 간격으로 실행되는 시작 시간을 나타내는 타임스탬프
        - 사진 ex) 오늘이 2019-01-04인데, daily 배치가 도는 워크플로가 있으면, 그 워크플로의 execution_date는 2024-01-03임.
    - next_execution_date: 스케줄 간격의 종료 시간

![image](https://github.com/user-attachments/assets/b477b8f1-54a9-4c55-884c-f778b91ad081)

- execution_date 매개변수는 축약 매개변수로 제공됨
    - ds: YYYY-MM-DD
    - ds_nodash: YYYYMMDD

```python
fetch_events = BashOperator(
    task_id="fetch_events",
    bash_command=(
        "mkdir -p /data/events && "
        "curl -o /data/events.json "
        "http://events_api:5000/events?"
        "start_date={{ds}}&" # execution_date를 따름.
        "end_date={{next_ds}}" # next_execution_date를 따름.
    ),
    dag=dag,
)

```

- 데이터 파티셔닝: 데이터 세트를 더 작고 관리하기 쉬운 조각으로 나누는 작업
    - 태스크의 출력을 해당 실행 날짜의 이름이 적힌 파일에 기록하는 방법 등
- Airflow의 시간 처리는 스케줄 간격에 따라 실행됨
- **start_date ≠ execution_date**
    - execution_date: DAG 예약 간격의 시작 표시
- 태스크에서 previous_execution_date, next_execution_date 매개변수를 사용할 때, 주의할 점은 매개변수가 스케줄 간격 이후의 DAG 실행을 통해서만 정의된다는 것. 수동 실행할 경우에 매개변수 값이 정의되지 않음
- 백필(backfill): Airflow의 과거의 시작 날짜부터 과거의 간격 정의가 된다는 속성을 이용하여 과거 데이터 세트를 로드하거나 분석하기 위해 DAG의 과거 시점을 지정해 실행하는 것.
- 기본적으로 Airflow는 아직 실행되지 않은 과거 스케줄 간격을 예약하고 실행함. 과거 시작 날짜를 지정하고 해당 DAG를 활성화하면 현재 시간 이전에 과거 시작 이후의 스케줄 간격이 생성됨.
    - DAG의 catchup 매개변수에 의해 제어되며, false로 설정해 비활성화 할 수 있음

![image](https://github.com/user-attachments/assets/1764b17b-d099-4c8f-8874-61c6c0f510c1)


- 백필은 코드를 변경한 후, 데이터를 다시 처리하는 데 사용할 수 있음
- ex) 현재, 데이터를 count만 하는 python 함수를 생성하고 Airflow 스케줄을 실행해왔었는데, 데이터 값의 sum도 python 함수에 추가하여 백필(backfill)을 수행하면, 과거 데이터에 count와 sum을 한 값이 모두 생기게 된다.
    - 백필 수행 전 지표: count
    - 백필 수행 후 지표: count, sum
- Airflow 태스크의 가장 중요한 두 속성인 **원자성, 멱등성**
    - 원자성: 모든 것이 완료되거나 or 완료되지 않거나 하는 속성
    - 멱등성: 여러 번 실행해도 동일한 결과르 반환해야하는 속성

### Chapter4. Airflow 콘텍스트를 사용하여 태스크 템플릿 작업하기

- Airflow에서 {{}} (이중 중괄호)는 Jinja 템플릿 문자열을 나타내는 것임
- 템플릿 작성은 런타임 시에 값을 할당하기 위해서 활용함
- 템플릿으로 사용될 수 있는 속성 리스트가 별도로 존재하고 이는 airflow 공식 문서에서 확인 가능함.
- PythonOperator는 문자열 인수 대신 함수를 사용하므로 Jinja 템플릿을 작업을 할 수 없음
- PythonOperator는 콜러블 함수(Callable)에서 추가 인수를 제공하는 방법도 지원함
    - ex) output_path를 입력 가능하게 만들어 작업에 따라 출력 경로를 변경하기 위해 전체 함수를 복사하는 대신, output_path만 별도로 구성 가능함
    
    ```python
    def _get_data(output_path, **context):
    	year, month, day, hour, *_ = context[execution_date].timetuple()
    	url = {
    			"https://dumps.wikimedia.org/other/pageviews/"
    			f"{year}/{year}-{month:0>2}/pageviews-{year}{month:0>2}{day:0>2}-{hour:0>2}0000.gz"
    				}
    	request.urlretrieve(url, output_path)
    	
    # method1
    get_data = PythonOperator(
    		 task_id = "get_data"
    		,python_callable = _get_data
    		,op_args = ["/tmp/wikipageviews.gz"] # _get_data("/tmp/wikipageviews.gz")와 같음
    		,dag = dag
    		)
    		
    # method2
    get_data = PythonOperator(
    		 task_id = "get_data"
    		,python_callable = _get_data
    		,op_kwargs = ["output_path": "/tmp/wikipageviews.gz"] # _get_data("/tmp/wikipageviews.gz")와 같음
    		,dag = dag
    		)
    
    ```
