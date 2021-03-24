# DJANGO

## ORM
- `models.py`에서 object 생성
- `QuerySet` 으로 반환

### QuerySet
- django.db.models.query.QuerySet
- database의 object 집합 표현
- model의 Manager를 통해 획득
- QuerySet 연산의 결과는 QuerySet
- Lazy한 특성 => filter와 exclude가 쌓여도, runtime 때 내용 호출 시 딱 한 번 불림
- Method
    - filter: matching 결과
    - exclude: matching되지 않는 결과
    - get: matching 결과, 결과 없으면 `DoesNotExist Exception`
