<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_2_Part/django.png)
-->
# 게시판 페이징 기능 추가하기

지금까지 만든 파이보의 질문 목록에는 페이징 기능이 없었다. 페이징 기능이 없으면 어떻게 될까? 만약 게시물이 300개 작성되면 질문 목록 화면에 게시물이 300개 그대로 표시될것이다. 이런 경우 한 화면에 표시할 게시물이 많아져서 스크롤바를 내려야 하는 등의 불편함이 생기므로 페이징 기능은 필수다. 페이징 기능을 추가하는 방법을 알아보자.

## 임시 질문 데이터 300개 생성하기
페이징을 구현하기 전에 페이징을 테스트할 정도로 충분한 데이터를 생성하자. 여기서는 300개의 테스트 데이터를 생성한다. 대량의 테스트 데이터를 만드는 가장 좋은 방법은 장고셸을 이용하는 것이다.

1. 장고 셸 실행하고 필요한 모듈 임포트하기
다음처럼 장고 셰릉 실행하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py shell
Python 3.8.5 (tags/v3.8.5:580fbb0, Jul 20 2020, 15:57:54) [MSC v.1924 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> 
```
이어서 질문 데이터를 생성하기 위한 모듈을 임포트하자.
```
>>> from pybo.models import Question
>>> from django.utils import timezone
```

2. for 문으로 테스트 데이터 300개 만들기
for 문을 이용하여 다음과 같이 300개의 테스트 데이터를 생성하자.
```
>>> for i in range(300):
...     q = Question(subject='테스트 데이터입니다:[%03d]' % i, content='내용무', create_date=timezone.now())
...     q.save()
...
>>>  
```
이제 장고 셸을 종료하고 개발 서버를 실행한 다음 웹 브라우저에서 질문 목록 페이지를 호출하면 1~2단계로 등록한 300개의 테스트 데이터가 보인다. 그리고 앞에서 언급했듯 한없이 이어지는 게시물이 문제임을 금방 알아챌 것이다. 이게 바로 페이징이 필요한 이유이다.

![3-02_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_2_Part/3-02_1.png)

## 페이징 기능 살짝 구현해 보기
질문 목록 조회를 위한 index 함수에 페이징 기능을 추가해 보자.
```
from django.core.paginator import Paginator  


def index(request):
    """
    pybo 목록출력
    """
    # 입력 파라미터
    page = request.GET.get('page', '1')  # 페이지

    # 조회
    question_list = Question.objects.order_by('-create_date')

    # 페이징처리
    paginator = Paginator(question_list, 10)  # 페이지당 10개씩 보여주기
    page_obj = paginator.get_page(page)

    context = {'question_list': page_obj}
    context = {'question_list': question_list}
    return render(request, 'pybo/question_list.html', context)

(... 생략 ...)
```

index 함수의 형태를 자세히 살펴보자. page = request.Get.get('page', '1')은 다음과 같은 GET 방식 요청 URL에서 page값을 가져올 때 사용한다.
```
localhost:8000/pybo/?page=1
```

get('page', '1')에서 '1'은 /pybo/처럼 ?page=1과 같은 page 파라미터가 없는 URL을 위해 기본값으로 1을 지정한 것이다. 페이징 구현에 사용한 클래스는 Paginator이다.
```
paginator = Paginator(question_list, 10)
page_obj = paginator.get_page(page)
```
Paginator 클래스는 question_list를 페이징 객체 paginator로 변환한다. 두 번째 파라미터인 10은 페이지당 보여줄 게시물 개수를 의미한다. page_obj = paginator.get_page(page)로 만들어진 page_obj 객체에는 다음과 같은 속성들이 있다. 장고의 Paginator 클래스를 이용하면 별다른 수고 없이 다음 속성들을 사용할 수 있으므로 페이징 처리가 아주 쉬워진다.

| 항목 | 설명 |
| :--- | :--- |
| paginator.count | 전체 게시물 개수 |
| paginator.per_page | 페이지당 보여줄 게시물 개수 |
| paginator.page_range | 페이지 범위 |
| number | 현재 페이지 번호 |
| previous_page_number | 이전 페이지 번호 |
| next_page_number | 다음 페이지 번호 |
| has_previous | 이전 페이지 유무 |
| has_next | 다음 페이지 유무 |
| start_index	| 현재 페이지 시작 인덱스(1부터 시작) |
| end_index | 현재 페이지의 끝 인덱스(1부터 시작) |

>Paginator의 속성들은 템플릿에서 페이징을 처리할 때 사용된다.

이와 같이 index 함수를 수정한 후에 질문 목록 페이지에 접속하면 이제 300개의 질문 데이터가 표시되지 않고 페이징 기능으로 한 페이지에 10건씩 출력됨을 확인할 수 있다.

## 페이징 적용하기
이제 한 페이지에 10건씩 게시물을 출력할 수 있게 되었지만 페이지 이동 기능이 없어서 여전히 불편하다. 이번에는 페이지 이동 기능을 템플릿에 구현해 보자.

1. 질문 목록 템플릿에 페이징 기능 적용하기
question_list.html 템플릿 파일의 </table> 태그 바로 밑에 다음 코드를 추가하자.
```
(... 생략 ...)
</table>
    <!-- 페이징처리 시작 -->
    <ul class="pagination justify-content-center">
        <!-- 이전페이지 -->
        {% if question_list.has_previous %}
        <li class="page-item">
            <a class="page-link" href="?page={{ question_list.previous_page_number }}">이전</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">이전</a>
        </li>
        {% endif %}
        <!-- 페이지리스트 -->
        {% for page_number in question_list.paginator.page_range %}
            {% if page_number == question_list.number %}
            <li class="page-item active" aria-current="page">
                <a class="page-link" href="?page={{ page_number }}">{{ page_number }}</a>
            </li>
            {% else %}
            <li class="page-item">
                <a class="page-link" href="?page={{ page_number }}">{{ page_number }}</a>
            </li>
            {% endif %}
        {% endfor %}
        <!-- 다음페이지 -->
        {% if question_list.has_next %}
        <li class="page-item">
            <a class="page-link" href="?page={{ question_list.next_page_number }}">다음</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">다음</a>
        </li>
        {% endif %}
    </ul>
    <!-- 페이징처리 끝 -->
```
템플릿에 사용된 {{ question_list }}가 바로 views.py 파일의 page_obj이다. views.py파일에 다음과 같이 컨텍스트 정보를 입력했고, 이 정보가 템프릿으로 전달되었따.
```
context = {'question_list': page_obj}
```

다시 말해 템플릿의 {{ question_list.previous_page_number }}는 page_obj.previous_page_number와 동일하다.
> 여기서는 페이징(1,2,3, ...)을 보기 좋게 표시하기 위해 부트스트랩의 pagination 컴포넌트를 사용했다.
> 부트스트랩 pagination 컴포넌트 공식 문서: https://getbootstrap.com/docs/4.5/components/pagination

만약 이전 페이지가 있다면 '이전' 리읔가 활성화되지만, 반대로 이전 페이지가 없으면 '이전'링크는 비화럿ㅇ화된다. '다음' 링크의 경우도 마찬가지다. 그리고 {% for page_number in question_list.paginator.page_rnage %}와 {% endfor %} 템플릿 태그 사이에서는 페이지 리스트를 돌면서 해당 페이지로 이동할 수 있는 링크를 생성했다. 이때 현재 페이지 번호는 부트스트랩의 active 클래스를 적용하여 강조 표시도 했다. 코드의 양이 많아서 얼핏 복잡해 보이지만, 찬찬히 살펴보면 if 문과 for 문을 조합한 것이므로 생각보다 어렵지 않으니 천천히 분석해 보기 바란다.

![3-02_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_2_Part/3-02_3.png)

## 페이지 표시 제한 기능 구현하기
다만 여러분이 구현한 페이징 기능에는 1가지 문제가 있다. 앞에서 보듯 페이지 개수가 30까지 늘어나면 이동할 수 있는 페이지가 모두 표시되어 덜 다듬어진 서비스처럼 보인다는 점이다.

1. 페이징 템플릿 수정하기
이 문제를 해결하기 위해 다음과 같이 템플릿을 수정하자.
```
(... 생략 ...)
<!-- 페이지리스트 -->
{% for page_number in question_list.paginator.page_range %}
{% if page_number >= question_list.number|add:-5 and page_number <= question_list.number|add:5 %}
    {% if page_number == question_list.number %}
    <li class="page-item active" aria-current="page">
        <a class="page-link" href="?page={{ page_number }}">{{ page_number }}</a>
    </li>
    {% else %}
    <li class="page-item">
        <a class="page-link" href="?page={{ page_number }}">{{ page_number }}</a>
    </li>
    {% endif %}
{% endif %}
{% endfor %}
(... 생략 ...)
```

{% for page_number in question_list.paginator.page_range %} 바로 아래에 다음 코드를 삽입하여 페이지 표시 제한 기능을 구현했다. {% if ... %} ~ {% endif %}로 {% endif %}까지 입력해야 하니 코드 누락에 주의하자.
```
{% if page_number >= question_list.number|add:-5 and page_number <= question_list.number|add:5 %}
(... 생략 ...)
{% endif %}
```
위 코드는 페이지 번호가 현재 페이지 기준으로 좌우 5개씩 보이도록 만든다. question_list.number보다 5만큼 그너가 작은 값만 표시되도록 만든 것이다. | add:-5는 5만큼 빼라는 의미이고 |add:5는 5만큼 더하라는 의미이다. 만약 현재 페이지가 15라면 오른쪽 그림과 같이 페이지 번호가 나타날 것이다.

![3-02_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_2_Part/3-02_4.png)

축하한다! 페이징 기능이 완성되었다. 페이징은 사실 구현하기가 무척 어려운 기술이다. Paginator 클래스의 도움이 없었다면 아마 이렇게 쉽게 해내기는 힘들었을 것이다.
> 지금까지 만든 페이징 기능에 '처음'과 '마지막' 링크를 추가하고 '...'생략 기호까지 추가하면 더 완성도 높은 페이징 기능이 될 것이다.