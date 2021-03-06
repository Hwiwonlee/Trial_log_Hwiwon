---
title: "dplyr 시리즈 (1) : dplyr패키지의 기초"
author: "Hwiwon Lee"
date: "2020-08-17"
summary: "tidy-select과 data-masking, 그리고 dplyr 패키지의 기초 함수 용법"
categories:
  - tidyverse
tags:
  - R
  - tidyverse
  - dplyr
output: 
  # https://blog.zarathu.com/posts/2019-01-03-rmarkdown/
    bookdown::html_document2:
      number_sections: FALSE
      fig_caption: TRUE
      fig_height: 6
      fig_width: 10
      highlight: textmate
      theme: cosmo
      toc: yes
      toc_depth: 4
      toc_float: yes
      css: "post_style.css"
# https://stackoverflow.com/questions/535616/can-css-choose-a-different-default-font-and-size-depending-on-language

---





# **dplyr 패키지** 


데이터 분석의 첫 단계는 데이터셋을 분석에 적합하도록 가공하는 전처리(pre-process)입니다. 오늘은 R에서 **효율적인 전처리**를 도와주는 패키지, [**dplyr**](https://dplyr.tidyverse.org/articles/dplyr.html)를 알아보려고 합니다. 사실 전처리에 사용할 수 있는 함수가 **dplyr**에만 있는 것은 아닙니다. R에서 머신러닝을 지원하는 `caret` 패키지의 `preProcess()`나 생물분석을 지원하는 `Biconductor`의 `preprocessCore` 패키지로도 전처리는 가능합니다. 


그럼에도 **dplyr**를 소개해드리는 것은 **dplyr**는 전처리 뿐 아니라 [tidyverse](https://www.tidyverse.org/)를 이용한 코딩의 근간을 이루는  **파이프라인**(pipeline, %>%)을 제공한다는 강점을 갖기 때문입니다. R을 이용한 분석 코딩은 파이프라인 이전과 이후로 나뉜다고 봐도 무방할 정도로 파이프라인은 가독성 에서 엄청난 이점을 갖는 코딩 방법입니다. 이젠 **tidyverse** 뿐 아니라 다른 패키지들에서도 파이프라인의 사용을 지원하고 있으며 파이프라인을 이용하지 않는 코드를 찾는 것이 더 어려울 정도로 대중화된 방법이기도 합니다. 


따라서 파이프라인을 이용한 코딩의 기초가 되는 **dplyr**을 이용해 전처리, 데이터 핸들링, 요약 통계량을 이용한 표 만들기 등의 예제로 **dplyr**의 사용법을 소개해드리겠습니다. 먼저, 사용할 예제 데이터부터 보시죠. 


<div style="margin-bottom:60px;">
</div>


## **Marvel Superheroes**
[Marvel Superheroes](https://www.kaggle.com/dannielr/marvel-superheroes)는 캐글(kaggle)에 업로드된 오픈 데이터셋으로, 이름과는 다르게 Marvel 코믹스 뿐 아니라 DC 코믹스에 등장하는 캐릭터들의 정보 또한 포함하고 있습니다. 위의 링크를 통해 간단히 다운받을 수 있으며 저는 8개의 파일 중 `marvel_dc_characters.csv`만 로드해서 사용하도록 하겠습니다. 





```r
marvel_dc_characters <- read_csv("marvel_dc_characters.csv")
marvel_dc_characters
```

```
#> # A tibble: 39,648 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100001 Clau~ Secret   Neutral   Hazel    Brown     Male   Living           2
#>  2 100002 Elek~ Secret   Neutral   Blue     Black     Female Living         280
#>  3 100003 Thom~ Secret   Neutral   Black    <NA>      Male   Living           1
#>  4 100004 Mogu~ <NA>     <NA>      <NA>     Bald      <NA>   Living          NA
#>  5 100005 Deni~ Secret   Good      Brown    Black     Female Living           4
#>  6 100006 Aphr~ Secret   Good      Blue     Blond     Female Living          50
#>  7 100007 Jeve~ Public   Good      Brown    Brown     Male   Living           4
#>  8 100008 Fibe~ Secret   Neutral   Brown    Brown     Male   Living           1
#>  9 100009 Tran~ <NA>     Good      <NA>     No        <NA>   Living          NA
#> 10 100010 Snag~ No Dual  Bad       Yellow   Black     Female Living           2
#> # ... with 39,638 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```


39648개의 관측치와 12개의 feature를 가진 `marvel_dc_characters`를 이용해 dplyr의 함수들을 알아보도록 하겠습니다. 

<div style="margin-bottom:40px;">
</div>

## **하나의 데이터 테이블에서 사용할 수 있는 함수들**

[**Introduction to dplyr**](https://dplyr.tidyverse.org/articles/dplyr.html#single-table-verbs-1)에서는 다음의 세 가지 카테고리를 이용해 dplyr에서 지원하는 대표적임 함수들을 구분하고 있습니다. 

  + 행 :  
    + [**`filter()`**](https://dplyr.tidyverse.org/reference/filter.html) : 열의 값에 근거해 행을 선택하는 함수  
    + [`slice()`](https://dplyr.tidyverse.org/reference/slice.html) : 위치에 근거해 행을 선택하는 함수   
    + [`arrange()`](https://dplyr.tidyverse.org/reference/arrange.html) : 행의 순서를 바꾸는 함수   

  + 열 : 
    + [`select()`](https://dplyr.tidyverse.org/reference/select.html) : 열을 선택하거나 제외하는 함수  
    + [`rename()`](https://dplyr.tidyverse.org/reference/rename.html) : 열의 이름을 바꾸는 함수  
    + [**`mutate()`**](https://dplyr.tidyverse.org/reference/mutate.html) : 열의 값들을 바꾸거나 새로운 열을 추가하는 함수
    + [`relocate()`](https://dplyr.tidyverse.org/reference/relocate.html) : 열의 순서를 바꾸는 함수  

  + 행들의 그룹 : 
    + [**`summarise()`**](https://dplyr.tidyverse.org/reference/summarise.html) : 함수를 이용해 그룹을 하나의 행으로 요약하는 함수  


저도 위의 순서를 따라 예제와 함께 각 함수의 사용법을 간단히 보여드리겠습니다.

<div style="margin-bottom:40px;">
</div>

### 함수의 사용법을 알아보기 전에

**dplyr**의 함수 뿐 아니라 **tidyverse**의 함수들을 사용할 때 알아두면 좋은 개념을 간단히만 짚고 넘어가려고 합니다. 바로 **tidy-select**와 **data-masking** 입니다. 

**tidy-select**와 **data-masking**는 parameter type으로 특정 함수를 사용할 때 필요한 parameter입니다. 위에서 본 **dplyr**의 함수들은 모두 **tidy-select**나 **data-masking**의 parameter를 반드시 선언해주어야 합니다. 

패키지 안에 포함된 함수는 많지만 반드시 있어야 하는 parameter type이 같게 설계되어 있다는 것은 사용자 입장에서 굉장한 장점입니다. 함수는 다르지만 parameter type이 같다면 같은 형식으로 설정해도 함수를 실행할 수 있다는 말이기 때문입니다. 두 개념은 [Programming with dplyr](https://dplyr.tidyverse.org/articles/programming.html)에 정리되어 있으며 다른 문서에서 저도 정리해볼 생각입니다. 이번엔 간단히 용례만 살펴보고 지나가겠습니다. 

<div style="margin-bottom:25px;">
</div>

#### **tidy-select**

**tidy-select**는 변수의 자리나 이름, 타입을 이용해 열을 선택하는 방법으로 아래와 같이 사용합니다.  **tidy-select**를 사용하는 대표적인 함수로 `across()`, `relocate()`, `rename()`, `select()`, `pull()` 등이 있습니다.

  - c(열 이름 1, 열 이름 2, ...) 처럼 `c()`를 이용하는 방법  
  - 대상이 되는 열 이름들을 저장한 오브젝트와 `all_of()`, `any_of()`를 사용하는 방법
  - 열 이름 1:열 이름 n 처럼 연속적인 열을 선택하기 위해 `:`를 이용하는 방법
  - [tidyselect](https://tidyselect.r-lib.org/reference/language.html)에서 볼 수 있는 것처럼 `starts_with()`이나 `ends_with()` 또는 `contatins()`나 `matches()`를 사용하는 방법
  - `is.factor`, `is.numeric` 처럼 열이 가진 value들의 type을 이용하는 방법
  - 위의 방법에 `!`, `&`, `|`를 적용, 조건을 이용해 선택하는 방법

<div style="margin-bottom:15px;">
</div>

#### **data-masking**

**data-masking**은 데이터의 변수를 환경 변수처럼 사용할 수 있도록 해 가독성과 함수 활용성을 높히는 방법입니다. **data-masking**을 사용하는 대표적인 함수로 `arrange()`, `count()`, `filter()`, `group_by()`,` mutate()`, `summarise()` 등이 있습니다. 
  
  - 예를 들어, **data-masking**한 `filter(dataset, x1 == 0 & x2 != 0)`와 **data-masking**를 사용하지 않은 `dataset[dataset$x1 == 0 & datasetx2 != 0, ]`는 같은 결과를 리턴하지만 가독성에서도, 함수 활용성에서도 차이를 보입니다. 

<div style="margin-bottom:15px;">
</div>

### [`filter()`](https://dplyr.tidyverse.org/reference/filter.html)를 이용한 행 선택  


[`filter()`](https://dplyr.tidyverse.org/reference/filter.html)는 그 이름에 맞게 필터링을 위한 함수입니다. 열은 건드리지 않고 **조건**에 해당하는 행만 추출하거나 제외하고 싶을 때 사용합니다. `filter()`를 이용해 `marvel_dc_characters`에서 `Marvel` 코믹스의 캐릭터만 추출해보겠습니다. 


```r
# a. filter()를 이용해 조건에 맞는 행 추출하기
marvel_dc_characters %>% 
  filter(Universe == "Marvel")
```

```
#> # A tibble: 16,376 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100002 Elek~ Secret   Neutral   Blue     Black     Female Living         280
#>  2 100008 Fibe~ Secret   Neutral   Brown    Brown     Male   Living           1
#>  3 100009 Tran~ <NA>     Good      <NA>     No        <NA>   Living          NA
#>  4 100012 Para~ <NA>     <NA>      <NA>     <NA>      Male   Living           1
#>  5 100013 Clif~ Public   <NA>      Brown    Black     Male   Decea~           4
#>  6 100014 Oya ~ Secret   Good      Brown    White     Female Living           4
#>  7 100015 Jame~ Public   Bad       <NA>     White     Male   Decea~           4
#>  8 100016 Hard~ Secret   Neutral   Black    No        Male   Living           4
#>  9 100017 Vale~ No Dual  <NA>      Brown    Red       Female Living           4
#> 10 100021 Cili~ Public   Neutral   <NA>     <NA>      Female Living          34
#> # ... with 16,366 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```


논리연산자를 이용해 이중, 삼중 등 조건을 중첩시키는 방법 또한 가능하며 그 안에 함수를 사용하는 것도 가능합니다. 



```r
# b. filter()를 이용해 두 개의 조건을 논리연산자로 묶어 행 추출하기
marvel_dc_characters %>% 
  # stringr::str_detect()를 이용해 조건 (1) 선언, ==를 이용해 조건 (2) 선언
  # &로 두 조건의 교집합에 해당하는 행을 추출하도록 함 
  filter(str_detect(Universe, "Marvel") & Status == "Living")
```

```
#> # A tibble: 12,608 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100002 Elek~ Secret   Neutral   Blue     Black     Female Living         280
#>  2 100008 Fibe~ Secret   Neutral   Brown    Brown     Male   Living           1
#>  3 100009 Tran~ <NA>     Good      <NA>     No        <NA>   Living          NA
#>  4 100012 Para~ <NA>     <NA>      <NA>     <NA>      Male   Living           1
#>  5 100014 Oya ~ Secret   Good      Brown    White     Female Living           4
#>  6 100016 Hard~ Secret   Neutral   Black    No        Male   Living           4
#>  7 100017 Vale~ No Dual  <NA>      Brown    Red       Female Living           4
#>  8 100021 Cili~ Public   Neutral   <NA>     <NA>      Female Living          34
#>  9 100022 Arie~ <NA>     Bad       <NA>     <NA>      Male   Living           1
#> 10 100023 Clar~ Public   <NA>      <NA>     Brown     Male   Living           4
#> # ... with 12,598 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```


더 나아가 `filter()`안에 연산 함수를 이용해 조건을 추가할 수도 있습니다. 



```r
# c. 연산 함수를 이용해 filter()에 조건 추가하기
marvel_dc_characters %>% 
  filter(
    !is.na(HairColor) & # HairColor가 NA가 아닌 행
      Year > 2000 & # 2000년 이후로 등장
      Appearances >= mean(Appearances, na.rm = TRUE) # 등장횟수의 평균보다 적은 횟수로 등장하는 캐릭터만 추출
  )
```

```
#> # A tibble: 716 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100147 "Mad~ No Dual  Good      Green    Red       Female Living          37
#>  2 100170 "Mil~ Public   Good      White    Black     Female Living          43
#>  3 100263 "Sal~ No Dual  <NA>      Brown    Black     Female Living          41
#>  4 100370 "Mar~ Secret   Good      Brown    Black     Female Living          31
#>  5 100377 "Vic~ No Dual  Neutral   Hazel    Black     Female Decea~          98
#>  6 100457 "Sid~ Public   <NA>      Multiple No        Male   Decea~          27
#>  7 100544 "Mic~ Secret   Good      Brown    Black     Male   Living          35
#>  8 100622 "Edw~ Secret   Bad       Green    No        Male   Decea~          35
#>  9 100654 "Mic~ Public   Good      Blue     Brown     Male   Living          27
#> 10 100709 "Kel~ Secret   Good      Blue     Blond     Female Living          32
#> # ... with 706 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```
등장횟수, Appearances의 평균은 18.2으로 위와 같이 여러 조건과 더불어 평균 이상 등장한 캐릭터를 추출할 수도 있습니다. 

<div style="margin-bottom:25px;">
</div>

### [`slice()`](https://dplyr.tidyverse.org/reference/slice.html)를 이용한 행 선택


위치를 이용해 행을 선택할 수 있는 함수인 `slice()`는 `filter()`보다 자주 쓰이진 않지만 index를 사용해 특정 행을 추출하고자 할 때 이용할 수 있습니다. 


```r
# a. row index 1부터 10까지의 행 추출하기
marvel_dc_characters %>% 
  slice(1:10)
```

```
#> # A tibble: 10 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100001 Clau~ Secret   Neutral   Hazel    Brown     Male   Living           2
#>  2 100002 Elek~ Secret   Neutral   Blue     Black     Female Living         280
#>  3 100003 Thom~ Secret   Neutral   Black    <NA>      Male   Living           1
#>  4 100004 Mogu~ <NA>     <NA>      <NA>     Bald      <NA>   Living          NA
#>  5 100005 Deni~ Secret   Good      Brown    Black     Female Living           4
#>  6 100006 Aphr~ Secret   Good      Blue     Blond     Female Living          50
#>  7 100007 Jeve~ Public   Good      Brown    Brown     Male   Living           4
#>  8 100008 Fibe~ Secret   Neutral   Brown    Brown     Male   Living           1
#>  9 100009 Tran~ <NA>     Good      <NA>     No        <NA>   Living          NA
#> 10 100010 Snag~ No Dual  Bad       Yellow   Black     Female Living           2
#> # ... with 3 more variables: FirstAppearance <chr>, Year <dbl>, Universe <chr>
```

`slice()`의 arrange인 `slice_head()`와 `slice_tail()`을 이용하면 행 index의 기준점을 바꿀 수 있습니다. 


```r
# b. slice_tail()을 이용해 끝에서부터 행 추출하기
marvel_dc_characters %>% 
  slice_tail(n = 10)
```

```
#> # A tibble: 10 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 139639 "Rob~ No Dual  Neutral   <NA>     Black     Male   Living           1
#>  2 139640 "Coo~ Public   Bad       <NA>     Blond     Male   Living           3
#>  3 139641 "Ast~ <NA>     Good      <NA>     <NA>      Female Decea~           3
#>  4 139642 "Aqu~ Secret   Bad       <NA>     Blond     Male   Living           8
#>  5 139643 "Mar~ Public   Good      <NA>     Brown     Female Living           1
#>  6 139644 "Iva~ Public   Bad       <NA>     Black     Male   Living           2
#>  7 139645 "Tom~ <NA>     Bad       Brown    Bald      Male   Living           9
#>  8 139646 "Ben~ Secret   Bad       <NA>     <NA>      Male   Decea~          29
#>  9 139647 "Mer~ Secret   Bad       <NA>     <NA>      Male   Living           1
#> 10 139648 "Bru~ Secret   Bad       <NA>     Black     Male   Living           3
#> # ... with 3 more variables: FirstAppearance <chr>, Year <dbl>, Universe <chr>
```

`slice_min()`과 `slice_max()`로 열의 데이터 값을 기준으로 n개의 행을 선택할 수도 있습니다. 


```r
# c. silce_max()를 이용해 열 데이터 값을 기준으로 행 선택하기 
marvel_dc_characters %>% 
  filter(!is.na(Appearances)) %>% # Appearance에 NA 제거
  slice_max(Appearances, n = 10) # Appearance가 큰 순서대로 10개의 행 선택
```

```
#> # A tibble: 10 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 108474 "Spi~ Secret   Good      Hazel    Brown     Male   Living        4043
#>  2 124908 "Spi~ Secret   Good      Hazel    Brown     Male   Living        4043
#>  3 126082 "Cap~ Public   Good      Blue     White     Male   Living        3360
#>  4 136719 "Cap~ Public   Good      Blue     White     Male   Living        3360
#>  5 107773 "Bat~ Secret   Good      Blue     Black     Male   Living        3093
#>  6 104882 "Wol~ Public   Neutral   Blue     Black     Male   Living        3061
#>  7 136331 "Wol~ Public   Neutral   Blue     Black     Male   Living        3061
#>  8 107862 "Iro~ Public   Good      Blue     Black     Male   Living        2961
#>  9 115031 "Iro~ Public   Good      Blue     Black     Male   Living        2961
#> 10 116286 "Sup~ Secret   Good      Blue     Black     Male   Living        2496
#> # ... with 3 more variables: FirstAppearance <chr>, Year <dbl>, Universe <chr>
```

추가로 `slice_sample(prop = 0.x)`을 이용해 랜덤하게 데이터셋의 `x%`만큼의 행을 선택할 수도 있습니다. 

<div style="margin-bottom:25px;">
</div>

### [`arrange()`](https://dplyr.tidyverse.org/reference/arrange.html)를 이용해 행을 기준으로 정렬


`arrange()`는 데이터 핸들링 초반보다 요약 통계량으로 채워진 테이블을 정렬할 때 더 많이 사용됩니다. 제가 지금 사용하고 있는 `marvel_dc_characters` 만 해도 행의 개수가 약 4만 개에 달하므로 단순히 정렬하는 것으로 무언가를 발견하기는 어렵기 때문입니다. 그러므로 간단히 사용 방법에 대해 알아보겠습니다. 



```r
# a. Appearances를 기준으로 오름차순 정렬
marvel_dc_characters %>% 
  arrange(Appearances)
```

```
#> # A tibble: 39,648 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100003 Thom~ Secret   Neutral   Black    <NA>      Male   Living           1
#>  2 100008 Fibe~ Secret   Neutral   Brown    Brown     Male   Living           1
#>  3 100012 Para~ <NA>     <NA>      <NA>     <NA>      Male   Living           1
#>  4 100018 Rimi~ Public   Neutral   Green    Red       Female Living           1
#>  5 100022 Arie~ <NA>     Bad       <NA>     <NA>      Male   Living           1
#>  6 100024 Geec~ Secret   Bad       <NA>     Black     Male   Living           1
#>  7 100025 Turh~ Public   Good      <NA>     Black     Male   Living           1
#>  8 100027 Niko~ <NA>     Bad       <NA>     <NA>      Male   Living           1
#>  9 100028 Myrm~ <NA>     <NA>      <NA>     <NA>      <NA>   Living           1
#> 10 100030 Ense~ Public   Neutral   <NA>     Brown     Male   Living           1
#> # ... with 39,638 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```


```r
# b. HairColor를 기준으로 내림차순 정렬
marvel_dc_characters %>% 
  arrange(desc(HairColor))
```

```
#> # A tibble: 39,648 x 12
#>        ID Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100298 D'Ka~ Secret   Bad       Yellow   Yellow    Male   Living          NA
#>  2 100466 Prey~ Public   Bad       Green    Yellow    Male   Living           5
#>  3 101249 Stra~ Secret   Good      Red      Yellow    Male   Living          12
#>  4 104447 Igor~ Public   Bad       Black    Yellow    Male   Living          34
#>  5 105346 Frid~ Secret   Good      Blue     Yellow    Female Living           1
#>  6 105712 Igor~ Public   Bad       Black    Yellow    Male   Living          34
#>  7 106400 Harp~ <NA>     Bad       <NA>     Yellow    Female Living           1
#>  8 107056 Man ~ Secret   Neutral   Red      Yellow    Male   Living           2
#>  9 107828 Harp~ <NA>     Bad       <NA>     Yellow    Female Living           1
#> 10 108085 Scou~ Secret   Good      Green    Yellow    Male   Living           3
#> # ... with 39,638 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```

**b.**에서 확인할 수 있는 것처럼, `arrange()`는 문자열에도 사용 가능하며 `NA` 때문에 에러를 리턴하지도 않습니다. 


<div style="margin-bottom:25px;">
</div>

### [`select()`](https://dplyr.tidyverse.org/reference/select.html)를 이용한 열 선택


지금까지 본 dplyr의 함수처럼 `select()` 또한 직관적인 사용 방법을 가진 유용한 함수입니다. `select()`를 통해 **feature selection**을 하진 않지만 **EDA**(Exploratory Data Analysis) 단계에서 필요한 열만 가지고 계산된 요약 통계량이나 시각화 결과를 보여주기 위해 사용되곤 합니다. 

보통의 데이터셋이 많은 수의 feature를 가지고 있으므로 열을 선택하는 `select()`은 몇 가지 효율적인 선택 방법을 지원합니다. 예제를 통해 알아보겠습니다. 


```r
# a. 기초적인 select() 활용법
# a.a) 열 이름을 통한 선택
marvel_dc_characters %>% 
  select(ID, Name, Identity)
```

```
#> # A tibble: 39,648 x 3
#>        ID Name                          Identity
#>     <dbl> <chr>                         <chr>   
#>  1 100001 Claude Potier (Earth-616)     Secret  
#>  2 100002 Elektra Natchios (Earth-616)  Secret  
#>  3 100003 Thomas Williams (Earth-616)   Secret  
#>  4 100004 Mogul (Earth-616)             <NA>    
#>  5 100005 Denise Havens (Earth-616)     Secret  
#>  6 100006 Aphrodite Ourania (Earth-616) Secret  
#>  7 100007 Jeven Ognats (Post-Zero Hour) Public  
#>  8 100008 Fiber (Earth-616)             Secret  
#>  9 100009 Tranta (Earth-616)            <NA>    
#> 10 100010 Snaggi (Earth-616)            No Dual 
#> # ... with 39,638 more rows
```

```r
# a.b) 열의 위치를 이용한 선택
marvel_dc_characters %>% 
  select(1:3)
```

```
#> # A tibble: 39,648 x 3
#>        ID Name                          Identity
#>     <dbl> <chr>                         <chr>   
#>  1 100001 Claude Potier (Earth-616)     Secret  
#>  2 100002 Elektra Natchios (Earth-616)  Secret  
#>  3 100003 Thomas Williams (Earth-616)   Secret  
#>  4 100004 Mogul (Earth-616)             <NA>    
#>  5 100005 Denise Havens (Earth-616)     Secret  
#>  6 100006 Aphrodite Ourania (Earth-616) Secret  
#>  7 100007 Jeven Ognats (Post-Zero Hour) Public  
#>  8 100008 Fiber (Earth-616)             Secret  
#>  9 100009 Tranta (Earth-616)            <NA>    
#> 10 100010 Snaggi (Earth-616)            No Dual 
#> # ... with 39,638 more rows
```


```r
# b. 함수를 통한 선택 방법
marvel_dc_characters %>% 
  select(starts_with("I"), matches("Color$|^A"))
```

```
#> # A tibble: 39,648 x 6
#>        ID Identity Alignment EyeColor HairColor Appearances
#>     <dbl> <chr>    <chr>     <chr>    <chr>           <dbl>
#>  1 100001 Secret   Neutral   Hazel    Brown               2
#>  2 100002 Secret   Neutral   Blue     Black             280
#>  3 100003 Secret   Neutral   Black    <NA>                1
#>  4 100004 <NA>     <NA>      <NA>     Bald               NA
#>  5 100005 Secret   Good      Brown    Black               4
#>  6 100006 Secret   Good      Blue     Blond              50
#>  7 100007 Public   Good      Brown    Brown               4
#>  8 100008 Secret   Neutral   Brown    Brown               1
#>  9 100009 <NA>     Good      <NA>     No                 NA
#> 10 100010 No Dual  Bad       Yellow   Black               2
#> # ... with 39,638 more rows
```

**b.**에서 사용한 두 함수 외에도 `contains()`와 `ends_with()` 또한 사용 가능합니다. 이 함수들과 정규 표현식(Regular Expression)을 사용한다면 수 많은 feature 사이에서도 원하는 열을 추출해내는 것이 가능합니다. 

추가로 일정한 숫자패턴(ex, x01, x02, ...)를 가진 열을 선택하기 위한 [`num_range()`](https://tidyselect.r-lib.org/reference/starts_with.html), 데이터 값의 형식으로 열을 선택할 수 있는 [`where()`(https://tidyselect.r-lib.org/reference/where.html)] 등도 사용할 수 있습니다. 


`select()`를 이용해 열 이름을 바꿀 수도 있습니다. 


```r
# c. select()로 열 이름 바꾸기
marvel_dc_characters %>% 
  select(Index = ID) # new name = old name
```

```
#> # A tibble: 39,648 x 1
#>     Index
#>     <dbl>
#>  1 100001
#>  2 100002
#>  3 100003
#>  4 100004
#>  5 100005
#>  6 100006
#>  7 100007
#>  8 100008
#>  9 100009
#> 10 100010
#> # ... with 39,638 more rows
```


<div style="margin-bottom:25px;">
</div>

### [`rename()`](https://dplyr.tidyverse.org/reference/rename.html)으로 열 이름 바꾸기 


데이터 분석을 하면 종종 데이터셋의 열 이름을 바꿔야 하는 경우가 있습니다. 데이터셋의 열 이름이 해당 분야의 전문용어로 된 경우가 대표적인 경우입니다. 데이터를 축적하거나 저장한 곳과 분석하는 곳의 소속이나 단체가 다른 경우라면 이런 상황은 더욱 빈번히 발생합니다. 

`rename()` 그리고 `rename_with()`은 열 이름을 쉽고 효과적으로 바꾸기 위한 함수들입니다. 예제를 이용해 찬찬히 살펴보겠습니다. 


```r
# a. rename()의 기본적인 사용법
marvel_dc_characters %>% 
  rename(Index = ID, ) # new name = old name
```

```
#> # A tibble: 39,648 x 12
#>     Index Name  Identity Alignment EyeColor HairColor Gender Status Appearances
#>     <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 100001 Clau~ Secret   Neutral   Hazel    Brown     Male   Living           2
#>  2 100002 Elek~ Secret   Neutral   Blue     Black     Female Living         280
#>  3 100003 Thom~ Secret   Neutral   Black    <NA>      Male   Living           1
#>  4 100004 Mogu~ <NA>     <NA>      <NA>     Bald      <NA>   Living          NA
#>  5 100005 Deni~ Secret   Good      Brown    Black     Female Living           4
#>  6 100006 Aphr~ Secret   Good      Blue     Blond     Female Living          50
#>  7 100007 Jeve~ Public   Good      Brown    Brown     Male   Living           4
#>  8 100008 Fibe~ Secret   Neutral   Brown    Brown     Male   Living           1
#>  9 100009 Tran~ <NA>     Good      <NA>     No        <NA>   Living          NA
#> 10 100010 Snag~ No Dual  Bad       Yellow   Black     Female Living           2
#> # ... with 39,638 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```

```r
# a. rename()의 기본적인 사용법
marvel_dc_characters %>% 
  select(ID, Alignment) %>% 
  rename(Index = ID, Propensity = Alignment) # new name = old name
```

```
#> # A tibble: 39,648 x 2
#>     Index Propensity
#>     <dbl> <chr>     
#>  1 100001 Neutral   
#>  2 100002 Neutral   
#>  3 100003 Neutral   
#>  4 100004 <NA>      
#>  5 100005 Good      
#>  6 100006 Good      
#>  7 100007 Good      
#>  8 100008 Neutral   
#>  9 100009 Good      
#> 10 100010 Bad       
#> # ... with 39,638 more rows
```

`rename_with()`은 `rename()`의 확장판 격으로 좀 더 다양한 상황에서 열 이름을 바꿀 수 있는 기능들을 지원합니다. 


```r
# b. rename_with()을 사용해 열 이름 바꾸기
# b.a) parameter, .fn와.col를 사용해서 열 이름 바꾸기
marvel_dc_characters %>% 
  # select()로 열 순서 바꾸기 
  select(EyeColor, HairColor, everything()) %>% 
  rename_with(.fn = toupper,
              .cols = matches("Color"))
```

```
#> # A tibble: 39,648 x 12
#>    EYECOLOR HAIRCOLOR     ID Name  Identity Alignment Gender Status Appearances
#>    <chr>    <chr>      <dbl> <chr> <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 Hazel    Brown     100001 Clau~ Secret   Neutral   Male   Living           2
#>  2 Blue     Black     100002 Elek~ Secret   Neutral   Female Living         280
#>  3 Black    <NA>      100003 Thom~ Secret   Neutral   Male   Living           1
#>  4 <NA>     Bald      100004 Mogu~ <NA>     <NA>      <NA>   Living          NA
#>  5 Brown    Black     100005 Deni~ Secret   Good      Female Living           4
#>  6 Blue     Blond     100006 Aphr~ Secret   Good      Female Living          50
#>  7 Brown    Brown     100007 Jeve~ Public   Good      Male   Living           4
#>  8 Brown    Brown     100008 Fibe~ Secret   Neutral   Male   Living           1
#>  9 <NA>     No        100009 Tran~ <NA>     Good      <NA>   Living          NA
#> 10 Yellow   Black     100010 Snag~ No Dual  Bad       Female Living           2
#> # ... with 39,638 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```


```r
# b.b) parameter, .fn를 사용해서 열 이름 바꾸기
marvel_dc_characters %>% 
  # select()로 열 순서 바꾸기 
  select(EyeColor, HairColor, everything()) %>% # EyeColor, HairColor를 앞으로 당겨오기 위한 select()
  rename_with(.fn = ~ tolower(.x))
```

```
#> # A tibble: 39,648 x 12
#>    eyecolor haircolor     id name  identity alignment gender status appearances
#>    <chr>    <chr>      <dbl> <chr> <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 Hazel    Brown     100001 Clau~ Secret   Neutral   Male   Living           2
#>  2 Blue     Black     100002 Elek~ Secret   Neutral   Female Living         280
#>  3 Black    <NA>      100003 Thom~ Secret   Neutral   Male   Living           1
#>  4 <NA>     Bald      100004 Mogu~ <NA>     <NA>      <NA>   Living          NA
#>  5 Brown    Black     100005 Deni~ Secret   Good      Female Living           4
#>  6 Blue     Blond     100006 Aphr~ Secret   Good      Female Living          50
#>  7 Brown    Brown     100007 Jeve~ Public   Good      Male   Living           4
#>  8 Brown    Brown     100008 Fibe~ Secret   Neutral   Male   Living           1
#>  9 <NA>     No        100009 Tran~ <NA>     Good      <NA>   Living          NA
#> 10 Yellow   Black     100010 Snag~ No Dual  Bad       Female Living           2
#> # ... with 39,638 more rows, and 3 more variables: firstappearance <chr>,
#> #   year <dbl>, universe <chr>
```


**b.b)**의 `.fn`에서 사용된 `~`, 물결표(tilde)와 `.x`에 대한 설명을 간단히 하겠습니다. `~`와 `.x`를 이용해 함수를 parameter로 선언하는 것은 [**purrr 패키지**에서 지원하는 **익명 함수 사용 방법**](https://purrr.tidyverse.org/reference/map.html)입니다. 

말은 어렵지만 선형모델을 이용한 적합에서 사용하는 `y ~ x`, 즉 `y에 대한 x의 함수`라는 의미로 이해하실 수도 있으며 이 방법으로 이해하신다면 **"`rename_with(`)에 대한 `.x`의 함수로 `tolower()`를 사용한다."**가 될 것입니다. 

**익명함수**나 위의 용법에 대해서는 **purrr패키지**를 설명해드리거나 **`across()`**를 이용한 **dplyr** 함수 사용을 설명해드릴 때 조금 더 자세히 설명해보겠습니다. 


<div style="margin-bottom:25px;">
</div>


### [`relocate()`](https://dplyr.tidyverse.org/reference/relocate.html)로 열 순서 바꾸기

`rename()`의 **b.**에서 `select()`와 `everything()`을 이용해 열 순서를 바꿔보았습니다. 사실, **열의 순서를 바꾼다**는 목적에 있어서는 `relocate()`의 효율이 더 좋습니다. `relocate()`가 가진 parameter, `.before`과 `.after`로 다양한 상황에 대처할 수 있기 때문입니다. 예제를 통해 알아보도록 하겠습니다. 



```r
# a. relocate의 일반적인 사용법 
marvel_dc_characters %>% 
  relocate(c(Identity, Alignment), .before = ID)
```

```
#> # A tibble: 39,648 x 12
#>    Identity Alignment     ID Name  EyeColor HairColor Gender Status Appearances
#>    <chr>    <chr>      <dbl> <chr> <chr>    <chr>     <chr>  <chr>        <dbl>
#>  1 Secret   Neutral   100001 Clau~ Hazel    Brown     Male   Living           2
#>  2 Secret   Neutral   100002 Elek~ Blue     Black     Female Living         280
#>  3 Secret   Neutral   100003 Thom~ Black    <NA>      Male   Living           1
#>  4 <NA>     <NA>      100004 Mogu~ <NA>     Bald      <NA>   Living          NA
#>  5 Secret   Good      100005 Deni~ Brown    Black     Female Living           4
#>  6 Secret   Good      100006 Aphr~ Blue     Blond     Female Living          50
#>  7 Public   Good      100007 Jeve~ Brown    Brown     Male   Living           4
#>  8 Secret   Neutral   100008 Fibe~ Brown    Brown     Male   Living           1
#>  9 <NA>     Good      100009 Tran~ <NA>     No        <NA>   Living          NA
#> 10 No Dual  Bad       100010 Snag~ Yellow   Black     Female Living           2
#> # ... with 39,638 more rows, and 3 more variables: FirstAppearance <chr>,
#> #   Year <dbl>, Universe <chr>
```

```r
  # relocate(c(Identity, Alignment), .after = ID)
```


```r
# b. 함수를 이용한 사용법 
marvel_dc_characters %>% 
  # where()을 이용한 사용법
  relocate(where(is.numeric), .before = is.character)
```

```
#> # A tibble: 39,648 x 12
#>        ID Appearances  Year Name  Identity Alignment EyeColor HairColor Gender
#>     <dbl>       <dbl> <dbl> <chr> <chr>    <chr>     <chr>    <chr>     <chr> 
#>  1 100001           2  2000 Clau~ Secret   Neutral   Hazel    Brown     Male  
#>  2 100002         280  1981 Elek~ Secret   Neutral   Blue     Black     Female
#>  3 100003           1  2002 Thom~ Secret   Neutral   Black    <NA>      Male  
#>  4 100004          NA  1970 Mogu~ <NA>     <NA>      <NA>     Bald      <NA>  
#>  5 100005           4  1995 Deni~ Secret   Good      Brown    Black     Female
#>  6 100006          50  1948 Aphr~ Secret   Good      Blue     Blond     Female
#>  7 100007           4  1994 Jeve~ Public   Good      Brown    Brown     Male  
#>  8 100008           1    NA Fibe~ Secret   Neutral   Brown    Brown     Male  
#>  9 100009          NA  1991 Tran~ <NA>     Good      <NA>     No        <NA>  
#> 10 100010           2    NA Snag~ No Dual  Bad       Yellow   Black     Female
#> # ... with 39,638 more rows, and 3 more variables: Status <chr>,
#> #   FirstAppearance <chr>, Universe <chr>
```

```r
  # matches()를 이용한 사용법
  # relocate(matches("Color"), .before = is.numeric)
```

`relocate()`의 `.before`과 `.after` 옵션을 이용하면 **tidy-select**로 선택한 열들의 순서를 좀 더 유연하게 설정할 수 있습니다. 

<div style="margin-bottom:25px;">
</div>

### [`mutate()`](https://dplyr.tidyverse.org/reference/mutate.html)를 이용해 열 편집 혹은 생성하기

`mutate()`는 **dplyr**에서 가장 많이 쓰이는 함수가 아닌가 싶습니다. `mutate()`는 새로운 열의 추가부터 데이터 값의 편집까지 데이터 분석에서 필요로 하는 대부분의 처리를 지원하는 함수입니다. **지원하는 함수**라는 것은 `mutate()` 하나로 할 수 있는 것이 없다는 의미이며 이는 곧 **여러 함수를 섞어 써야 한다는 의미**이기도 합니다. 따라서 `mutate()`의 활용은 사용자의 숙련도에 따라 크게 달라집니다. 예제로 `mutate()`의 활용성을 알아보겠습니다. 



```r
# a. mutate에서 유용하게 사용할 수 있는 함수들을 이용한 예제
# https://dplyr.tidyverse.org/reference/mutate.html
marvel_dc_characters %>% 
  mutate(ID = row_number(ID), 
         fame = 
           case_when(
             Appearances <= 10 ~ "유명하지 않음",
             Appearances <= 100 ~ "조금 유명함",
             Appearances <= 200 ~ "꽤 유명함",
             Appearances > 200 ~ "누구나 알 수 있음",
           )
         ) %>% 
  select(fame, Appearances, everything())
```

```
#> # A tibble: 39,648 x 13
#>    fame  Appearances    ID Name  Identity Alignment EyeColor HairColor Gender
#>    <chr>       <dbl> <int> <chr> <chr>    <chr>     <chr>    <chr>     <chr> 
#>  1 유명하지~           2     1 Clau~ Secret   Neutral   Hazel    Brown     Male  
#>  2 누구나 ~         280     2 Elek~ Secret   Neutral   Blue     Black     Female
#>  3 유명하지~           1     3 Thom~ Secret   Neutral   Black    <NA>      Male  
#>  4 <NA>           NA     4 Mogu~ <NA>     <NA>      <NA>     Bald      <NA>  
#>  5 유명하지~           4     5 Deni~ Secret   Good      Brown    Black     Female
#>  6 조금 유~          50     6 Aphr~ Secret   Good      Blue     Blond     Female
#>  7 유명하지~           4     7 Jeve~ Public   Good      Brown    Brown     Male  
#>  8 유명하지~           1     8 Fibe~ Secret   Neutral   Brown    Brown     Male  
#>  9 <NA>           NA     9 Tran~ <NA>     Good      <NA>     No        <NA>  
#> 10 유명하지~           2    10 Snag~ No Dual  Bad       Yellow   Black     Female
#> # ... with 39,638 more rows, and 4 more variables: Status <chr>,
#> #   FirstAppearance <chr>, Year <dbl>, Universe <chr>
```

```r
# 임의의 함수 생성
square <- function(x){
  y <- x^2 
  return(y)
}

# b. 여러가지 함수와 mutate()를 이용한 열 생성 및 수정
marvel_dc_characters %>% 
  mutate(ID = seq(1, nrow(.), 1),
         Name = str_remove(Name, " \\(.*\\)$"), 
         Fame = square(Appearances)
         ) %>% 
  select(ID, Name, Fame, Appearances, everything())
```

```
#> # A tibble: 39,648 x 13
#>       ID Name   Fame Appearances Identity Alignment EyeColor HairColor Gender
#>    <dbl> <chr> <dbl>       <dbl> <chr>    <chr>     <chr>    <chr>     <chr> 
#>  1     1 Clau~     4           2 Secret   Neutral   Hazel    Brown     Male  
#>  2     2 Elek~ 78400         280 Secret   Neutral   Blue     Black     Female
#>  3     3 Thom~     1           1 Secret   Neutral   Black    <NA>      Male  
#>  4     4 Mogul    NA          NA <NA>     <NA>      <NA>     Bald      <NA>  
#>  5     5 Deni~    16           4 Secret   Good      Brown    Black     Female
#>  6     6 Aphr~  2500          50 Secret   Good      Blue     Blond     Female
#>  7     7 Jeve~    16           4 Public   Good      Brown    Brown     Male  
#>  8     8 Fiber     1           1 Secret   Neutral   Brown    Brown     Male  
#>  9     9 Tran~    NA          NA <NA>     Good      <NA>     No        <NA>  
#> 10    10 Snag~     4           2 No Dual  Bad       Yellow   Black     Female
#> # ... with 39,638 more rows, and 4 more variables: Status <chr>,
#> #   FirstAppearance <chr>, Year <dbl>, Universe <chr>
```

**a.**의 예제는 [`mutate()`](https://dplyr.tidyverse.org/reference/mutate.html)의 페이지를 참고해 `mutate()`와 함께 사용하면 유용한 함수 중 임의로 2개를 골라 사용해보았습니다. 특별히 `**case_when()**`은 SQL의 `case_when()`과 같습니다. R에서는 이미 `if_else()`가 존재하지만 조건문이 여러 개인 경우, 저는 가독성 측면에서 `case_when()`의 사용을 권장합니다. 

**b.**는 좀 더 다양한 함수 활용을 보여드리기 위해 pipeline 바깥에서 사용자 정의 함수를 설정한 뒤 pipeline에서 사용해보았습니다. 사용자 정의 함수를 포함한 많은 함수를 `mutate()`안에서 사용하실 수 있습니다. 

`mutate()`를 이용한 데이터핸들링에 익숙해지는 방법은 많이 보고 연습하는 방법 밖에 없습니다. 자신의 분석에서 어떤 열을 추가하고 싶거나 데이터 값을 수정하고 싶을 때, 구글링을 통해서 아이디어를 얻거나 따라해보시면서 차근차근 익히시는 것을 추천해드립니다. 


<div style="margin-bottom:25px;">
</div>

### [**`summarise()`**](https://dplyr.tidyverse.org/reference/summarise.html)를 이용해 요약 통계량 테이블 만들기

`summarise()`는 그 이름처럼 데이터 테이블의 요약을 위해 사용되는 함수입니다. 일반적으로 사용되는 요약 통계량, 평균, 중간값, 최소, 최대, 분위수 등 여러 함수들을 동시에 사용할 수 있으므로 일목요연하게 변수들의 통계적 특징을 나타낼 수 있습니다. 특히, `summarise()`는 명목형 변수의 각 수준으로 데이터셋을 묶어주는 `group_by()`와 함께 사용할 때 더욱 효과적입니다. 예제를 보며 설명드리겠습니다. 


```r
# a. summairse()를 이용해 만든 Appearances의 요약 통계량 테이블
marvel_dc_characters %>% 
  # Alignment에 따른 요약 통계량 계산을 위한 group_by
  group_by(Alignment) %>% 
  # 요약 통계량 계산, 평균, 중간값, 최소, 최대, 1사분위수, 3사분위수
  summarise(mean = mean(Appearances, na.rm = TRUE), 
            median = median(Appearances, na.rm = TRUE), 
            min = min(Appearances, na.rm = TRUE),
            max = max(Appearances, na.rm = TRUE),
            `1st quantile` = quantile(Appearances, 0.25, prob = 0.25, na.rm = TRUE), 
            `3nd quantile` = quantile(Appearances, 0.75, prob = 0.75, na.rm = TRUE)) %>% 
  # transpose를 위한 pivoting
  pivot_longer(-Alignment) %>%  
  pivot_wider(names_from = Alignment, values_from = value)
```

```
#> # A tibble: 6 x 6
#>   name            Bad   Good Neutral `Reformed Criminals`   `NA`
#>   <chr>         <dbl>  <dbl>   <dbl>                <dbl>  <dbl>
#> 1 mean           8.73   35.5    19.8                 29.7   7.98
#> 2 median         3       6       3                    6     3   
#> 3 min            1       1       1                    5     1   
#> 4 max          721    4043    3061                   78   605   
#> 5 1st quantile   1       2       1                    5.5   1   
#> 6 3nd quantile   7      17      10                   42     6
```
먼저 `group_by()`를 이용해 `Alignment`의 수준 별로 grouping 해준 후 `Appearances`열을 대상으로 `summarise()`를 적용해보았습니다. `summarise()` 안에 평균, 중간값, 최소, 최대, 1사분위수, 3사분위수를 계산하는 함수를 넣어 요약 통계량 테이블을 만들었습니다. 요약 통계량을 근거로 `Good` 진영의 캐릭터 등장횟수가 높은 편임을 알 수 있습니다. 


<div style="margin-bottom:25px;">
</div>

### dplyr 함수의 다양한 버전

지금까지 보신 **dplyr** 함수 중 대부분은 여러 버전을 갖습니다. 가령, `summarise()`의 경우 `summarise_at()`, `summarise_all()`, `summarise_each()`, `summarise_if()` 등의 버전으로 세분화됩니다. 이 버전들은 모두 용도에 따라 구분된 것으로 접미사에 따라 특별한 parameter를 갖기도 합니다. 

데이터 핸들링에서 필요한 여러 기능들을 함수로 지원하는 것은 꽤 만족스러운 일이지만 함수의 종류가 지나치게 많아 사용자의 부담이 늘어나는 것 또한 사실입니다. 

그래서일까요? 최근 dplyr패키지가 1.0.0으로 업데이트 되면서 `across()`를 활용하는 방법이 추가되었습니다. 따라서 다음 글에서 함수의 세부 버전보다는 최근 추가된 `across()`를 이용한 dplyr 패키지 함수 사용법에 대해 알아보도록 하겠습니다. 

<div style="margin-bottom:25px;">
</div>


# **Summary**

**dplyr**은 R 기반의 효율적인 코딩을 지원하는 **pipeline**을 비롯해 데이터핸들링에 필요한 여러 함수들을 지원하는 패키지입니다. 많은 함수을 포함하고 있지만 위에서 소개해드린 dplyr의 대표적인 함수정도만 숙지하셔도 데이터 핸들링에 큰 어려움은 없을 것이라 생각합니다. 함수의 용법을 알고 넘어가는 것도 중요하지만 **tidyverse** 전반에서 통용되는 **tidy-select**와 **data-masking**에 대해서도 한 번 더 읽고 넘어가시는 것을 추천해드립니다. 

`across()`를 이용한 **dplyr** 패키지 함수 사용법으로 찾아뵙겠습니다. 감사합니다. 







