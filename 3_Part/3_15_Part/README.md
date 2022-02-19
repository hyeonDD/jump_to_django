<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_15_Part/django.png)
-->
# 검색, 정렬 기능 추가하기
여기서는 파이보에 검색 기능과 정렬 기능을 추가할 것이다. 파이보는 질문, 답변 데이터가 계속 쌓이는 게시판 서비스이므로 검색 기능은 필수다. 검색 대상은 '제목', '질문 내용', '질문 글쓴이', '답변 글쓴이'로 정하겠다. 예를 들어 '파이썬'이라고 검색하면 '파이썬'이라는 문자열이 '제목', '질문 내용', '질문 글쓴이', '답변 글쓴이'에 있는지 검사하고, 검사 결과를 화면에 보여 준다.

## 검색 기능 자세히 알아보기
이와 같은 조건으로 검색하려면 질문 목록 조회 코드를 다음과 같이 수저애햐 한다. 아직 파일 수정 단계가 아니므로 눈으로만 살펴보자.
```
from django.db.models import Q

kw = request.GET.get('kw', '')  # 검색어

if kw:
    question_list = question_list.filter(
        Q(subject__icontains=kw) |  # 제목검색
        Q(content__icontains=kw) |  # 내용검색
        Q(author__username__icontains=kw) |  # 질문 글쓴이검색
        Q(answer__author__username__icontains=kw)  # 답글 글쓴이검색
    ).distinct()
```
> 답변 내용도 검색에 포함하려면 Q(answer__content__icontains=kw)를 추가하면 된다.

Q 함수는 OR 조건으로 데이터를 조회하는 장고의 함수이다. 위 코드는 제목, 내용, 글쓴이를 OR 조건으로 검색한다. filter 함수 뒤에 사용한 distinct 함수는 조회 결과의 중복을 제거하여 반환한다.
> 1개의 글에 여러 개의 답변이 있을 때 답변자 중복을 처리하기 위해 distinct 함수를 반드시 사용해야 한다.

다음은 kw를 사용한 코드의 일부이다. 코드를 유심히 보면 kw는 웹 브라우저에서 전달받은 검색어라는 것을 짐작할 수 있으며 GET 방식으로 받아온 값임을 알 수 있다.
```
localhost:8000/?kw=파이썬&page=1
```
```
kw = request.GET.get('kw', '') #검색어
```
> kw는 keywords의 줄임말이다.

kw를 POST 방식으로 전달하는 방법은 추천하지 않는다. 왜냐하면 kw를 POST 방식으로 전달하면 page 역시 POST 방식으로 전달해야 하기 때문이다. 또한 POST 방식으로 검색, 페이징 기능을 만들면 웹 브라우저에서 '새로고침' 또는 '뒤로가기'를 했을 때 '만료된 페이지' 오류를 종종 만나게 된다. POST 방식은 동일한 POST 요청이 발생하면 중복을 방지하려고 오류를 발생시키기 때무닝다.
예를 들어 2페이지에서 3페이지로 갔다가 '뒤로가기'를 하여 2페이지로 갈 때 '만료된 페이지' 오류를 만날 수 있다. 이러한 이유로 게시판을 조회하는 목록 함수는 GET 방식을 사용해야 한다. 이후 알아볼 정렬 기능 역시 GET 방식으로 구현할 것이다. 그러면 본격적으로 검색 기능을 만들어 보자.

## 검색 기능 만들어 보기

1. 질문 목록 화면에 검색창 추가하기
question\question_list.html 템플릿에 검색 창을 추가하자.
```
{% extends 'pybo/base.html' %}
{% load pybo_filter %}
{% block content %}
<div class="container my-3">
    <div class="row justify-content-end my-3">
        <div class="col-4 input-group">
            <input type="text" class="form-control kw" value="{{ kw|default_if_none:'' }}">
            <div class="input-group-append">
                <button class="btn btn-outline-secondary" type="button" id="btn_search">찾기</button>
            </div>
        </div>
    </div>
    <table class="table">
    (... 생략 ...)
```
<table> 위에 검색 창을 생성했다. 그리고 자바스크립트에서 검색 창에 입력된 값을 읽을 수 있도록 input 엘리먼트 class 속성에 kw를 추가했다.

2. 질문 목록 템플릿에 form 엘리먼트 추가하기
page와 kw를 동시에 GET 방식으로 요청할 수 있도록 form 엘리먼트를 추가하자.
```
(... 생략 ...)
    <!-- 페이징처리 끝 -->
    <a href="{% url 'pybo:question_create' %}" class="btn btn-primary">질문 등록하기</a>
</div>
<form id="searchForm" method="get" action="{% url 'index' %}">
    <input type="hidden" id="kw" name="kw" value="{{ kw|default_if_none:'' }}">
    <input type="hidden" id="page" name="page" value="{{ page }}">
</form>
{% endblock %}
```
GET 방식으로 요청해야 하므로 method 속성에 "get"을 설정했다. kw와 page는 이전에 요청했던 값을 기억해야 하므로 value 속성에 그 값을 대입했는데, kw와 page값은 질문 목록 함수에서 전달받는다. form 엘리먼트의 action 속성은 '폼이 전송되는 URL'이므로 질문 목록 URL인 "{% url 'index' %}"를 지정했다.

3. 질문 목록의 페이징 수정하기
그리고 기존의 페이징 처리 방식도 ?page=1에서 값을 읽어 요청하는 방식으로 변경해야 한다.
```
(... 생략 ...)
<!-- 페이징처리 시작 -->
<ul class="pagination justify-content-center">
    <!-- 이전페이지 -->
    {% if question_list.has_previous %}
    <li class="page-item">
        <a class="page-link" data-page="{{ question_list.previous_page_number }}" href="#">이전</a>
    </li>
    {% else %}
    <li class="page-item disabled">
        <a class="page-link" tabindex="-1" aria-disabled="true" href="#">이전</a>
    </li>
    {% endif %}
    <!-- 페이지리스트 -->
    {% for page_number in question_list.paginator.page_range %}
    {% if question_list.number|add:-5 <= page_number <= question_list.number|add:5 %}
        {% if page_number == question_list.number %}
        <li class="page-item active" aria-current="page">
            <a class="page-link" data-page="{{ page_number }}" href="#">{{ page_number }}</a>
        </li>
        {% else %}
        <li class="page-item">
            <a class="page-link" data-page="{{ page_number }}" href="#">{{ page_number }}</a>
        </li>
        {% endif %}
    {% endif %}
    {% endfor %}
    <!-- 다음페이지 -->
    {% if question_list.has_next %}
    <li class="page-item">
        <a class="page-link" data-page="{{ question_list.next_page_number }}" href="#">다음</a>
    </li>
    {% else %}
    <li class="page-item disabled">
        <a class="page-link" tabindex="-1" aria-disabled="true" href="#">다음</a>
    </li>
    {% endif %}
</ul>
<!-- 페이징처리 끝 -->
(... 생략 ...)
```
모든 페이지 링크를 href 속성에 직접 입력하는 대신 data-page 속성으로 값을 읽을 수 있도록 변경했다.

4. 질문 목록 템플릿에 페이징과 검색을 위한 자바스크립트 코드 추가하기
페이징과 검색을 처리하는 자바스크립트 코드를 추가하자.
```
(... 생략 ...)
{% endblock %}
{% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    $(".page-link").on('click', function() {
        $("#page").val($(this).data("page"));
        $("#searchForm").submit();
    });

    $("#btn_search").on('click', function() {
        $("#kw").val($(".kw").val());
        $("#page").val(1);  // 검색버튼을 클릭할 경우 1페이지부터 조회한다.
        $("#searchForm").submit();
    });
});
</script>
{% endblock %}
```
class 속성이 "page-link"인 링크를 누르면 이 링크의 data-page 속성값을 읽어 searchForm의 page 필ㄷ에 그 값을 설정하여 폼을 요청하도록 했다. 또한 <검색> 버튼을 누르면 검색 창에 입력된 값을 searchForm의 kw 필드에 설정하여 폼을 요청하도록 했다. 이때 <검색> 버튼을 누르는 경우는 새로운 검색 요청에 해당하므로 searchForm의 page 필드에 항상 1을 설정하여 폼을 요청하도록 했다.

5. index 함수 수정하기
검색어가 질문 목록 조회에 적용될 수 있도록 views\base_views.py 파일을 열어 index 함수를 수정하자.
```
(... 생략 ...)
from django.db.models import Q
(... 생략 ...)

def index(request):
    """
    pybo 목록출력
    """
    # 입력 파라미터
    page = request.GET.get('page', '1')  # 페이지
    kw = request.GET.get('kw', '')  # 검색어

    # 조회
    question_list = Question.objects.order_by('-create_date')
    if kw:
        question_list = question_list.filter(
            Q(subject__icontains=kw) |  # 제목검색
            Q(content__icontains=kw) |  # 내용검색
            Q(author__username__icontains=kw) |  # 질문 글쓴이검색
            Q(answer__author__username__icontains=kw)  # 답변 글쓴이검색
        ).distinct()

    # 페이징처리
    paginator = Paginator(question_list, 10)  # 페이지당 10개씩 보여주기
    page_obj = paginator.get_page(page)

    context = {'question_list': page_obj, 'page': page, 'kw': kw}
    return render(request, 'pybo/question_list.html', context)
(... 생략 ...)
```
Q 함수에 사용된 subject__icontains=kw는 제목에 kw 문자열이 포함되었는지를 의미한다. answer__author__username__icontains은 답변을 작성한 사람의 이름에 포함되는지를 의미한다. filter 함수에서 모델 필드에 접근하려면 이처럼 __를 이용하면 된다.
> subject__contains=kw 대신 subject__icontains=kw을 사용하면 대소문자를 가리지 않고 찾아 준다.

그리고 입력으로 받은 page와 kw값을 템플릿 searchForm에 전달하기 위해 context 안에 'page', 'kw'를 각각 page, kw으로 추가했다. 검색 창에 '마크다운'이라고 검색어를 입력한 다음 <찾기> 버튼을 눌러 보자. 그러면 다음과 같은 검색 결과가 나올 것이다.

![3-15_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_15_Part/3-15_1.png)

## 정렬 기능 만들어 보기
이번에는 질문 목록을 정렬할 수 있는 기능을 추가해 보자. 정렬 기준은 다음과 같다.

---

* 최신순: 최근 등록된 질문을 먼저 보여 주는 방식
* 추천순: 추천을 많이 받은 질문을 먼저 보여 주는 방식
* 인기순: 질문에 등록된 답변이 많은 질문을 먼저 보여 주는 방식

---

이러한 정렬 기준에 해당하는 파라미터 역시 GET 방식으로 요청해야 페이징, 검색, 정렬 기능이 잘 작동한다.

1. 질문 목록 화면에 정렬 조건 추가하기
question\question_list.html 파일에 정렬 조건을 추가하자.
```
{% extends 'pybo/base.html' %}
{% load pybo_filter %}
{% block content %}
<div class="container my-3">
    <div class="row justify-content-between my-3">  <!-- 양쪽정렬 justify-content-between로 변경 -->
        <div class="col-2">
            <select class="form-control so">
                <option value="recent" {% if so == 'recent' %}selected{% endif %}>최신순</option>
                <option value="recommend" {% if so == 'recommend' %}selected{% endif %}>추천순</option>
                <option value="popular" {% if so == 'popular' %}selected{% endif %}>인기순</option>
            </select>
        </div>
        <div class="col-4 input-group">
            <input type="text" class="form-control kw" value="{{ kw|default_if_none:"" }}">
            <div class="input-group-append">
                <button class="btn btn-outline-secondary" type="button" id="btn_search">찾기</button>
            </div>
        </div>
    </div>
    <table class="table">
    (... 생략 ...)
```
우선 div 엘리먼트를 오른쪽 정렬(justify-content-end)에서 양쪽 정렬(justify-content-between)으로 변경했다. 그런 다음 왼쪽에는 정렬 기준을 추가하고 오른쪽에는 검색 조건을 추가했다. 또한 '현재 선택된 정렬 기준'을 읽을 수 있도록 select 엘리먼트의 class를 so로 지정했다.

2. 질문 목록 템플릿의 searchForm 수정하기
searchForm에 정렬 기준을 입력할 수 있도록 input 엘리먼트를 추가하자.
```
(... 생략 ...)
<form id="searchForm" method="get" action="{% url 'index' %}">
    <input type="hidden" id="kw" name="kw" value="{{ kw|default_if_none:"" }}">
    <input type="hidden" id="page" name="page" value="{{ page }}">
    <input type="hidden" id="so" name="so" value="{{ so }}">
</form>
(... 생략 ...)
```

3. 질문 목록 템플릿의 자바스크립트 코드 수정하기
그리고 정렬 기준 콤보박스를 변경할 때 searchForm 요청이 발생하도록 다음과 같이 제이쿼리 자바스크립트를 추가하자.
```
(... 생략 ...)
{% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    (... 생략 ...)
    $(".so").on('change', function() {
        $("#so").val($(this).val());
        $("#page").val(1);
        $("#searchForm").submit();
    });
});
</script>
{% endblock %}
```
class가 so인 엘리먼트, 즉 정렬 조건에 해당하는 select의 값이 변경되면 그 값을 searchForm의 so 필드에 저장하여 searchForm을 요청하도록 코드를 수정했다.

4. index 함수 수정하기
so에 입력된 값을 이용하여 질문 목록을 정렬할 수 있도록 base_views.py 파일의 index 함수를 다음처럼 수정하자.
```
(... 생략 ...)
from django.db.models import Q, Count
(... 생략 ...)
def index(request):
    """
    pybo 목록출력
    """
    # 입력 파라미터
    page = request.GET.get('page', '1')  # 페이지
    kw = request.GET.get('kw', '')  # 검색어
    so = request.GET.get('so', 'recent')  # 정렬기준

    # 정렬
    if so == 'recommend':
        question_list = Question.objects.annotate(num_voter=Count('voter')).order_by('-num_voter', '-create_date')
    elif so == 'popular':
        question_list = Question.objects.annotate(num_answer=Count('answer')).order_by('-num_answer', '-create_date')
    else:  # recent
        question_list = Question.objects.order_by('-create_date')

    # 검색
    question_list = Question.objects.order_by('-create_date')
    if kw:
        question_list = question_list.filter(
            Q(subject__icontains=kw) |  # 제목검색
            Q(content__icontains=kw) |  # 내용검색
            Q(author__username__icontains=kw) |  # 질문 글쓴이검색
            Q(answer__author__username__icontains=kw)  # 답글 글쓴이검색
        ).distinct()

    # 페이징처리
    paginator = Paginator(question_list, 10)  # 페이지당 10개씩 보여주기
    page_obj = paginator.get_page(page)

    context = {'question_list': page_obj, 'page': page, 'kw': kw, 'so': so}
    return render(request, 'pybo/question_list.html', context)
(... 생략 ...)
```
정렬 기준이 추천순(recommend)인 경우에는 충천 수가 큰 것부터 정렬하므로 order_by에 추천 수 -num_voter를 입력했다. 추천 수는 장고의 annotate 함수와 Count 함수를 사용하여 구현했다. Question.objects.annotate(num_voter=Count('voter'))에서 사용한 annotate 함수는 Question 모델의 기존 필드인 author, subject, content, create_date, modify_date, voter에 질문의 추천 수에 해당하는 num_voter 필드를 임시로 추가해 주는 함수이다. 이렇게 annotate 함수로 num_voter 필드를 추가하면 임시로 추가해 주는 함수이다. 이렇게 annotate 함수로 num_voter 필드를 추가하면 filter 함수나 order_by 함수에서 num_voter 필드를 사용할 수 있게 된다. 여기서 num_voter는 Count('voter')와 같이 Count 함수를 사용하여 만들었다. Count('voter')는 해당 질문의 추천 수이다.
order_by('-num_voter', '-create_date')와 같이 order_by 함수에 두 개 이상의 인자가 전달되는 경우 1번째 항목부터 우선수위를 매긴다. 즉, 추천 수가 같으면 최신순으로 정렬한다.
그리고 page, kw와 마찬가지로 목록 화면에서 정렬 기능을 사용해 보자. '인기순'으로 정렬하면 답변수가 많은 게시물부터 보여주는 것을 확인할 수 있을 것이다.

![3-15_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_15_Part/3-15_2.png)