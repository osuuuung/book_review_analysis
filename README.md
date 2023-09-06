# book_review_analysis
## 데이터 수집
1. 도서 정보나루 사이트의 오픈 API 활용
   * 조건을 조합해서 여러 카테고리 별 그룹 만들기
     ```params = {
      "startDt": "2018-01-01",
      "endDt": "2018-12-31",
      "gender": '0',
      "age" : '0',
      "kdc100" : '0',
      "format" : 'json}
     ```
   * 그룹 별 인기 대출 순위에 따른 도서 목록 받기
    
2. 위에서 얻은 그룹 별 개별 도서를 '교보 문고' 웹사이트에 겁색 후, '책 소개' 웹 스크래핑
  * 책 소개를 선택한 이유 : 모든 도서에 존재, 다양한 정보를 얻을 수 있음
  * 특정 도서 검색 함수 search_book
    ```
    def search_book(driver,name):
        base_url = 'https://search.kyobobook.co.kr/search?keyword={}&gbCode=TOT&target=total'
        driver.get(base_url.format(name))
        wait = WebDriverWait(driver, 10) 
        try :
            book_link = wait.until(EC.presence_of_element_located((By.XPATH,'//*[@id="shopData_list"]/ul/li[1]/div[1]/div[2]/div[2]/div[1]/div/a')))
            book_link.click()
        except:
            book_link = wait.until(EC.presence_of_element_located((By.XPATH,'//*[@id="shopData_list"]/ul/li/div[1]/div[2]/div[2]/div[1]/div/a')))
            book_link.click()
    ```

도서를 검색했을 때, 책이 하나 나오는 경우와 두 개 이상이 나오는 경우를 나눠서 타겟팅함

  * '책 소개' 스크래핑 코드
    ```
    def collect_review(driver, name) :
        wait = WebDriverWait(driver, 10)
        comment = ''
        review_tag = wait.until(EC.presence_of_all_elements_located((By.CSS_SELECTOR,'div.intro_bottom  div.info_text')))
    
        if len(review_tag) == 1 :
            comment = review_tag[0].text
        elif len(review_tag) >= 2 :
            for review in review_tag:
                comment += review.text
        else:
            comment = None
        return comment
    ```

'책 소개' 부분이 한 개의 div 태그로 구성되어있거나 두 개 이상으로 구성되어있는 경우, 혹은 아예 없는 경우를 나눠서 타겟팅함

3. 크롤링 과정 중에서 발생할 수 있는 다양한 에러들에 대비하기 위한 함수들
   * 재시도 함수 retry : 모종의 이유로 에러가 난 경우 해당 과정을 3번 반복함
     ```
     def retry(max_retries=3, delay_in_seconds=2):
        def decorator(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                retries = 0
                while retries < max_retries:
                    try:
                        return func(*args, **kwargs)
                    except Exception as e:
                        retries += 1
                        if retries < max_retries:
                            time.sleep(delay_in_seconds)
                        else:
                            raise e
            return wrapper
        return decorator
     ```

   데코레이션 함수로써 searvh_book 혹은 collect_review 함수 선언 시 사용

  *  truncate_after_delimeters : 검색의 정확도를 높이기 위해, 제목을 가공하는 코드
  *  restart_driver : 주기적으로 드라이버를 재시도하기 위한 함수
  *  save_intermediate_result_to_ecxel : 에러가 난다면 현재 진행상황까지 저장하는 코드

