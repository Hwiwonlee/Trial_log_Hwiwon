---
title: "Tidyr과 pivoting"
author: "Hwiwon Lee"
date: "2020-08-15"
summary: "Tidyr의 pivot_longer과 pivot_wider로 pivoting 해보기"
categories:
  - tidyverse
tags:
  - R
  - tidyverse
  - tidyr
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




# **Pivoting?**
사전에서 **pivot**을 찾으면 (축을 중심으로) 회전하다; 회전시키다의 정의를 찾을 수 있듯이, 데이터 핸들링에서의 **pivoting**은 데이터셋을 **어떠한 기준**에 따라 넓게 펴보거나 길게 좁혀보는 방법을 뜻합니다. pivoting의 아주 좋은 예시를 `tidyr`의 로고에서 찾을 수 있습니다. 

> 1.1.1의 tidyr에서 로고가 바뀌었습니다. 아래의 로고는 1.1이하의 로고입니다.


<div class="figure" style="text-align: center">
<img src="https://ih1.redbubble.net/image.543365563.2254/flat,750x1000,075,f.u2.jpg" alt="tidyr의 로고. 행과 열의 기준에 따라 데이터셋의 형태를 바꾸는 pivoting을 보여주고 있다" width="30%" height="30%" />
<p class="caption">Figure 1: tidyr의 로고. 행과 열의 기준에 따라 데이터셋의 형태를 바꾸는 pivoting을 보여주고 있다</p>
</div>

[tidyr의 메인 페이지에서 볼 수 있는 것](https://tidyr.tidyverse.org/index.html)과 같이 pivoting은 tidyr의 다섯 가지 핵심 기능 중 하나입니다. 다섯 가지를 간단히 나열해보면 아래와 같습니다.    

  * 데이터셋을 세로형 혹은 가로형으로 변환하는 pivoting의 `pivot_longer()`, `pivot_wider()`
  * 여러 수준으로 이뤄진 리스트를 tibble의 행과 열로 정리하는 Rectangling의 `unnest_longer()`, `unnest_wider()`, `unnest_auto()`, `hoist()`
  * 중첩된 데이터 프레임을 다루는 `nest()`, `unnest()`
  * 문자열을 값으로 갖는 열을 다루는 `separate()`, `extract()`, `unite()`
  * 결측치를 다루는 `complete()`, `drop_na()`, `fill()`, `replace_na()`
  
pivoting은 `pivot_longer()`, `pivot_wider()`이 아닌 `gather()`과 `spread()` 혹은 `melt()`와 `dcast()`를 사용해도 할 수 있는 작업입니다. 그러나  [**pivoting**](https://tidyr.tidyverse.org/articles/pivot.html) 페이지에서 `pivot_longer()`, `pivot_wider()`을 사용하면 좀 더 편리하고 직관적으로 pivoting할 수 있다고 하고 제 개인적으로도 `pivot_` 시리즈를 사용하는 것이 더 편했기 때문에 이 함수를 더 알아보자는 차원에서 포스트를 작성하게 되었습니다. 이제 예제를 통해 `pivot_longer()`와 `pivot_wider()`의 사용법을 알아보도록 하겠습니다. 


# **Longer**
`pivot_longer()`은 데이터셋의 여러 변수들을 하나의 변수로 중첩시켜 **세로로 긴 형태의 데이터셋**으로 만드는 함수입니다. 세로로 긴 형태의 데이터셋, 즉 세로형 데이터셋이 필요한 대표적인 경우는 여러 변수들을 하나의 명목형 변수(categorical variable 혹은 factor)로 만들 수 있을 때입니다. 예제를 통해 어떻게 변하는지 보며 이해해보도록 하겠습니다. 



## 예제를 통해 알아보는 `pivot_longer()`의 효과


천 명의 신용평가자료인 `GermanCredit` 데이터셋을 이용해서 설명해보겠습니다. 


```r
library(caret)
data(GermanCredit)
tibble(GermanCredit) # 1000 x 62
#> # A tibble: 1,000 x 62
#>    Duration Amount InstallmentRate~ ResidenceDurati~   Age NumberExistingC~
#>       <int>  <int>            <int>            <int> <int>            <int>
#>  1        6   1169                4                4    67                2
#>  2       48   5951                2                2    22                1
#>  3       12   2096                2                3    49                1
#>  4       42   7882                2                4    45                1
#>  5       24   4870                3                4    53                2
#>  6       36   9055                2                4    35                1
#>  7       24   2835                3                4    53                1
#>  8       36   6948                2                2    35                1
#>  9       12   3059                2                4    61                1
#> 10       30   5234                4                2    28                2
#> # ... with 990 more rows, and 56 more variables: NumberPeopleMaintenance <int>,
#> #   Telephone <dbl>, ForeignWorker <dbl>, Class <fct>,
#> #   CheckingAccountStatus.lt.0 <dbl>, CheckingAccountStatus.0.to.200 <dbl>,
#> #   CheckingAccountStatus.gt.200 <dbl>, CheckingAccountStatus.none <dbl>,
#> #   CreditHistory.NoCredit.AllPaid <dbl>, CreditHistory.ThisBank.AllPaid <dbl>,
#> #   ...
```


```r
names(GermanCredit)[1:20]
#>  [1] "Duration"                       "Amount"                        
#>  [3] "InstallmentRatePercentage"      "ResidenceDuration"             
#>  [5] "Age"                            "NumberExistingCredits"         
#>  [7] "NumberPeopleMaintenance"        "Telephone"                     
#>  [9] "ForeignWorker"                  "Class"                         
#> [11] "CheckingAccountStatus.lt.0"     "CheckingAccountStatus.0.to.200"
#> [13] "CheckingAccountStatus.gt.200"   "CheckingAccountStatus.none"    
#> [15] "CreditHistory.NoCredit.AllPaid" "CreditHistory.ThisBank.AllPaid"
#> [17] "CreditHistory.PaidDuly"         "CreditHistory.Delay"           
#> [19] "CreditHistory.Critical"         "Purpose.NewCar"
```

```r
names(GermanCredit)[grepl("Purpose.", names(GermanCredit))]
#>  [1] "Purpose.NewCar"              "Purpose.UsedCar"            
#>  [3] "Purpose.Furniture.Equipment" "Purpose.Radio.Television"   
#>  [5] "Purpose.DomesticAppliance"   "Purpose.Repairs"            
#>  [7] "Purpose.Education"           "Purpose.Vacation"           
#>  [9] "Purpose.Retraining"          "Purpose.Business"           
#> [11] "Purpose.Other"
```


62개의 열을 갖고 있는 `GermanCredit`의 열 이름에 일정한 규칙이 있음을 찾으셨나요? 여러 번 데이터셋을 분석해보신 분이라면 `변수의 이름.변수의 수준`으로 구성되어 있음을 쉽게 보실 수 있으실 겁니다. 예를 들어 대출 목적을 의미하는 `Purpose`의 경우, `Newcar`, `UsedCar`, `Furniture.Equipment`, `Radio.Television`, ... 등으로 합칠 수 있겠습니다. 한번 볼까요? 



```r
GermanCredit %>% 
  mutate(index = seq(1, 1000, 1)) %>%  
  pivot_longer(cols = matches("Purpose"), # 하나의 열로 길게 합칠 열들을 선택하고
               names_to = "Purpose", # `하나의 열`의 이름을 정해준 다음
               names_prefix = "Purpose.", # `하나의 열`에 들어갈 합쳐질 열의 이름들에서 빼고 싶은 부분을 정해주고
               values_to = "Purpose value") %>% # 길게 합쳐질 열들이 갖고 있던 값이 저장될 열의 이름까지 설정해주면 끝.
  select(index, Purpose, `Purpose value` , everything())
#> # A tibble: 11,000 x 54
#>    index Purpose `Purpose value` Duration Amount InstallmentRate~
#>    <dbl> <chr>             <dbl>    <int>  <int>            <int>
#>  1     1 NewCar                0        6   1169                4
#>  2     1 UsedCar               0        6   1169                4
#>  3     1 Furnit~               0        6   1169                4
#>  4     1 Radio.~               1        6   1169                4
#>  5     1 Domest~               0        6   1169                4
#>  6     1 Repairs               0        6   1169                4
#>  7     1 Educat~               0        6   1169                4
#>  8     1 Vacati~               0        6   1169                4
#>  9     1 Retrai~               0        6   1169                4
#> 10     1 Busine~               0        6   1169                4
#> # ... with 10,990 more rows, and 48 more variables: ResidenceDuration <int>,
#> #   Age <int>, NumberExistingCredits <int>, NumberPeopleMaintenance <int>,
#> #   Telephone <dbl>, ForeignWorker <dbl>, Class <fct>,
#> #   CheckingAccountStatus.lt.0 <dbl>, CheckingAccountStatus.0.to.200 <dbl>,
#> #   CheckingAccountStatus.gt.200 <dbl>, ...
```


위의 `pivot_longer()`의 결과물로 당장 확인할 수 있는 것은 아래의 세 개 정도가 될 것 같습니다. 

  * `Purpose`와 `Purpose value`  
    - `Purpose`에는 `Purpose.`가 붙어있던 항목들이 열로 저장  
    - `Purpose value`에는 `Purpose.항목`의 값이 열로 저장  
  * `index` 그리고 `Duration`부터 `Age`까지의 값이 같음   
  * 데이터셋의 행과 열의 개수  
    - 행은 1000에서 11000개로, 열은 62개에서 54개로 바뀜  

하나 하나씩 살펴보겠습니다. 먼저 `Purpose`에 정말 `Purpose.`가 붙어있던 항목들이 저장되어 있는지 확인해보겠습니다. 위의 코드에 `count()`만 더해주면 간단하게 확인 가능하겠죠? 



```
#> # A tibble: 11 x 2
#>    Purpose                 n
#>    <chr>               <int>
#>  1 Business             1000
#>  2 DomesticAppliance    1000
#>  3 Education            1000
#>  4 Furniture.Equipment  1000
#>  5 NewCar               1000
#>  6 Other                1000
#>  7 Radio.Television     1000
#>  8 Repairs              1000
#>  9 Retraining           1000
#> 10 UsedCar              1000
#> 11 Vacation             1000
```
`Purpose`에 `Business`부터 `Vacation`까지 처음 데이터셋의 row 개수와 같은 1000개씩 저장되어 있는 것을 확인할 수 있습니다. 



```r
GermanCredit %>% 
  mutate(index = seq(1, 1000, 1)) %>%  
  pivot_longer(cols = matches("Purpose"), 
               names_to = "Purpose", 
               names_prefix = "Purpose.", 
               values_to = "Purpose value") %>% 
  select(index, Purpose, `Purpose value` , everything()) %>% 
  filter(`Purpose value` != 0 & index <= 10) %>% 
  select(index, Purpose, `Purpose value`) -> check # pivot_longer의 결과에서 1:10 rows의 결과만 저장

GermanCredit %>% 
  mutate(index = seq(1, 1000, 1)) %>% 
  select(index, matches("purpose")) %>% 
  inner_join(check, ., by = "index") %>%  # inner_join을 통해 check와 pivot_longer이전의 데이터 결합
  as.data.frame()
#>    index             Purpose Purpose value Purpose.NewCar Purpose.UsedCar
#> 1      1    Radio.Television             1              0               0
#> 2      2    Radio.Television             1              0               0
#> 3      3           Education             1              0               0
#> 4      4 Furniture.Equipment             1              0               0
#> 5      5              NewCar             1              1               0
#> 6      6           Education             1              0               0
#> 7      7 Furniture.Equipment             1              0               0
#> 8      8             UsedCar             1              0               1
#> 9      9    Radio.Television             1              0               0
#> 10    10              NewCar             1              1               0
#>    Purpose.Furniture.Equipment Purpose.Radio.Television
#> 1                            0                        1
#> 2                            0                        1
#> 3                            0                        0
#> 4                            1                        0
#> 5                            0                        0
#> 6                            0                        0
#> 7                            1                        0
#> 8                            0                        0
#> 9                            0                        1
#> 10                           0                        0
#>    Purpose.DomesticAppliance Purpose.Repairs Purpose.Education Purpose.Vacation
#> 1                          0               0                 0                0
#> 2                          0               0                 0                0
#> 3                          0               0                 1                0
#> 4                          0               0                 0                0
#> 5                          0               0                 0                0
#> 6                          0               0                 1                0
#> 7                          0               0                 0                0
#> 8                          0               0                 0                0
#> 9                          0               0                 0                0
#> 10                         0               0                 0                0
#>    Purpose.Retraining Purpose.Business Purpose.Other
#> 1                   0                0             0
#> 2                   0                0             0
#> 3                   0                0             0
#> 4                   0                0             0
#> 5                   0                0             0
#> 6                   0                0             0
#> 7                   0                0             0
#> 8                   0                0             0
#> 9                   0                0             0
#> 10                  0                0             0
```


마찬가지로 `Purpose value`에 `Purpose.`가 붙어있었던 열의 값, 1이 저장되어있음을 볼 수 있습니다. 만약 이 시점에서 데이터로 분석을 할 것이라면 `Purpose value`의 가치는 없어졌기 때문에 `Purpose value`를 삭제할 수 있습니다. 이미 `Purpose`에 **대출의 목적**이 명목형 변수로 저장되어 있기 때문입니다. (물론, `pivot_longer()` 전의 `Purpose.대출목적`이 특별한 값을 갖는다면 지금처럼 `Purpose value`를 삭제해줄 수 없습니다.) 이와 같이 데이터셋 안에 퍼져있는 여러 개의 열과 그 열이 가진 값들을 합쳐 세로로 쌓는 것이 `pivot_longer()`의 효과입니다. 


`pivot_longer()`의 효과를 설명했으니 두 번째와 세 번째는 더욱 쉽게 확인해볼 수 있습니다. `pivot_longer()`를 사용해 열들을 세로로 길게 쌓게 되면 대상이 된 열만큼 나머지 열들의 값이 행에 복사됩니다. 경우의 수를 생각해보면 이해가 쉽습니다. 하나의 obs이 하나의 행을 가진 상태에서 `pivot_longer()`로 생긴 열의 경우의 수 만큼, 하나의 obs가 갖는 행이 늘어나게 되는 것이죠. 



```r
options(tibble.print_max = 11)
GermanCredit %>% 
  mutate(index = seq(1, 1000, 1)) %>%  
  pivot_longer(cols = matches("Purpose"), 
               names_to = "Purpose", 
               names_prefix = "Purpose.", 
               values_to = "Purpose value") %>% 
  select(index, Purpose, `Purpose value` , everything()) %>% 
  filter(index == 1)
#> # A tibble: 11 x 54
#>    index Purpose `Purpose value` Duration Amount InstallmentRate~
#>    <dbl> <chr>             <dbl>    <int>  <int>            <int>
#>  1     1 NewCar                0        6   1169                4
#>  2     1 UsedCar               0        6   1169                4
#>  3     1 Furnit~               0        6   1169                4
#>  4     1 Radio.~               1        6   1169                4
#>  5     1 Domest~               0        6   1169                4
#>  6     1 Repairs               0        6   1169                4
#>  7     1 Educat~               0        6   1169                4
#>  8     1 Vacati~               0        6   1169                4
#>  9     1 Retrai~               0        6   1169                4
#> 10     1 Busine~               0        6   1169                4
#> 11     1 Other                 0        6   1169                4
#> # ... with 48 more variables: ResidenceDuration <int>, Age <int>,
#> #   NumberExistingCredits <int>, NumberPeopleMaintenance <int>,
#> #   Telephone <dbl>, ForeignWorker <dbl>, Class <fct>,
#> #   CheckingAccountStatus.lt.0 <dbl>, CheckingAccountStatus.0.to.200 <dbl>,
#> #   CheckingAccountStatus.gt.200 <dbl>, ...
```


`pivot_longer()`에서 각 obs가 고유한 값을 갖는 `index == 1`만 추출한 결과입니다. `pivot_longer()`을 하기 전엔 index당 하나의 행을 가졌지만 `pivot_longer()`후에는 `Purpose`에 저장된 11개의 값 만큼 증가한 11개의 행을 갖고 있음을 볼 수 있습니다. 더하여, `Purpose`와 `Purpose_value`의 차이만 있을 뿐 나머지 열의 값에는 차이가 없음 또한 볼 수 있습니다.  


`pivot_longer()`후에 행의 개수가 늘어난 것이 바로 이러한 이유 때문입니다. 단순히 `11*1000`해서 증가한 것이죠. 추가된 `Purpose`열로 `Pupose.목적`의 열이 들어갔으니 11개의 열이 줄었고 `Purpose`와 `Purpose_value`가 생겼으니 2개의 열이 늘어서 총 9개의 열이 줄어든 53개의 열을 갖게 되는 것입니다(`index`는 제가 임의로 추가한 것이라 제외했습니다).


예제 데이터를 통해 `pivot_longer()` 효과를 살펴보았습니다. 이제 `pivot_longer()`을 좀 더 잘 쓰기 위해 `pivot_longer()`가 가진 parameter와 효과를 알아보겠습니다. 

## `pivot_longer()`의 parameter



```r
# pivot_longer( 
#   data, # dataset  
#   cols, # 필수, pivot_longer()의 타겟이 되는 열을 지정합니다.   
#   names_to = "name", # 대상이 된 열들이 저장될 열의 이름을 지정합니다.   
#   names_prefix = NULL, # 대상이 된 열들의 이름에서 제외할 문자열을 지정합니다.   
#   names_sep = NULL, # 대상이 된 열들의 이름이 가진 구분자를 정의해줍니다.   
#   names_pattern = NULL, # 대상이 된 열들의 이름이 가진 패턴을 정의해줍니다.   
#   names_ptypes = list(), # 대상이 된 열들이 저장될 열의 prototype을 정의해줍니다.   
#   names_transform = list(), # 대상이 된 열들이 저장될 열의 type을 정의해줍니다.   
#   names_repair = "check_unique", # 대상이 된 열들의 이름이 겹치는 경우, 에러를 방지하기 위한 설정값입니다.   
#   values_to = "value", # 대상이 된 열들이 갖고 있는 값이 저장될 열의 이름을 지정합니다.   
#   values_drop_na = FALSE, # 대상이 된 열들에 NA가 포함된 경우 drop 여부를 결정합니다.   
#   values_ptypes = list(), # 대상이 된 열들이 갖고 있는 값이 저장될 열의 prototype을 정의해줍니다.   
#   values_transform = list(), # 대상이 된 열들이 갖고 있는 값이 저장될 열의 type을 정의해줍니다.   
#     ...  
# )
```


1. data : 본인이 사용하실 데이터셋의 이름을 넣어주시면 됩니다. 만약 파이프라인을 이용하시고 계신다면 생략하셔도 됩니다.  
2. **cols** : `pivot_longer()`의 필수요소입니다. **대상이 되는 열을 지정**해주시면 됩니다. 열을 지정하는 방법은 아래와 같습니다. 다양한 방법이 있으니 상황에 맞게 가장 적절한 방법을 사용하시면 됩니다. 
    - c(열 이름 1, 열 이름 2, ...) 처럼 `c()`를 이용하는 방법  
    - 대상이 되는 열 이름들을 저장한 오브젝트와 `all_of()`, `any_of()`를 사용하는 방법
    - 열 이름 1:열 이름 n 처럼 연속적인 열을 선택하기 위해 `:`를 이용하는 방법
    - [tidyselect](https://tidyselect.r-lib.org/reference/language.html)에서 볼 수 있는 것처럼 `starts_with()`이나 `ends_with()` 또는 `contatins()`나 `matches()`를 사용하는 방법
    - `is.factor`, `is.numeric` 처럼 열이 가진 value들의 type을 이용하는 방법
    - 위의 방법에 `!`, `&`, `|`를 적용, 조건을 이용해 선택하는 방법
3. **names_to**, **values_to** : 문자열(character), `pivot_longer()`의 결과로 추가될 열의 이름을 정합니다. `names_to`는 대상이 되는 열들이 저장될 열의 이름을, `values_to`는 대상이 되는 열들이 갖고 있던 값이 저장될 열의 이름을 문자열로 갖습니다.   
4. names_prefix : 문자열(character), 대상이 되는 열의 이름에서 제외하고자 하는 문자열을 지정해줍니다. 
5. **names_sep** : 문자열(character),`names_to`에 여러 값을 넣었을 때, 값의 분기점이 되는 문자열을 넣어주면 넣어준 값을 기준으로 생성되는 열의 이름이 나뉩니다. 
6. **names_pattern** : 문자열(character), **대상이 되는 열들이 여러 의미를 가진 경우**, 이름들의 패턴을 넣어 `name_to`에서 설정한 열의 이름들에  정의된 열 이름들이 **쪼개어 저장**되도록 합니다. 
    - **names_sep**과 **names_pattern** : 말로는 설명이 참 어렵습니다. [pivot](https://tidyr.tidyverse.org/articles/pivot.html) 페이지의 예제 데이터를 통해 알아보도록 하겠습니다.
    
    ```r
    # a. names_sep의 활용 
    # (a) 일반적인 names_sep을 이용한 열 이름 분할
    data <- tribble(
      ~index,  ~birth_2000,  ~name_2000, ~birth_2010,  ~name_2010,
       1L, "01-15", "A",  "02-09", "F",
       2L, "04-22", "B",  "03-01", "G",
       3L, "08-11", "C",  "04-15", "H",
       4L, "11-09", "D",  "06-21", "I",
       5L, "12-07", "E",  "10-22", "J",
    )
    
    data %>%
      pivot_longer(-index,
               names_to = c("variable", "year"),
               names_sep = "_",
               values_drop_na = TRUE)
    #> # A tibble: 20 x 4
    #>    index variable year  value
    #>    <int> <chr>    <chr> <chr>
    #>  1     1 birth    2000  01-15
    #>  2     1 name     2000  A    
    #>  3     1 birth    2010  02-09
    #>  4     1 name     2010  F    
    #>  5     2 birth    2000  04-22
    #>  6     2 name     2000  B    
    #>  7     2 birth    2010  03-01
    #>  8     2 name     2010  G    
    #>  9     3 birth    2000  08-11
    #> 10     3 name     2000  C    
    #> # ... with 10 more rows
    ```

    
    ```r
    # (b) Special name, ".value"를 이용한 분할
    family <- tribble(
      ~family,  ~dob_USA,  ~dob_Eurupe, ~gender_USA, ~gender_Eurupe,
       1L, "1998-11-26", "2000-01-29",             1L,             2L,
       2L, "1996-06-22",           NA,             2L,             NA,
       3L, "2002-07-11", "2004-04-05",             2L,             2L,
       4L, "2004-10-10", "2009-08-27",             1L,             1L,
       5L, "2000-12-05", "2005-02-28",             2L,             1L,
    )
    
    family %>%
      pivot_longer(-family,
               names_to = c(".value", "Country"), # special name, ".value"
               names_sep = "_",
               values_drop_na = TRUE)
    #> # A tibble: 9 x 4
    #>   family Country dob        gender
    #>    <int> <chr>   <chr>       <int>
    #> 1      1 USA     1998-11-26      1
    #> 2      1 Eurupe  2000-01-29      2
    #> 3      2 USA     1996-06-22      2
    #> 4      3 USA     2002-07-11      2
    #> 5      3 Eurupe  2004-04-05      2
    #> 6      4 USA     2004-10-10      1
    #> 7      4 Eurupe  2009-08-27      1
    #> 8      5 USA     2000-12-05      2
    #> 9      5 Eurupe  2005-02-28      1
    ```

    
    ```r
    # b. names_pattern의 활용 
    names(who)[1:15]
    #>  [1] "country"      "iso2"         "iso3"         "year"         "new_sp_m014" 
    #>  [6] "new_sp_m1524" "new_sp_m2534" "new_sp_m3544" "new_sp_m4554" "new_sp_m5564"
    #> [11] "new_sp_m65"   "new_sp_f014"  "new_sp_f1524" "new_sp_f2534" "new_sp_f3544"
    ```

    
    ```r
    who %>% pivot_longer(cols = new_sp_m014 : newrel_f65,
                     names_to = c("diagnosis", "gender", "age"),
                     names_pattern = "new_?(.*)_(.)(.*)",
                     # _?(.*)_ : _가 있거나 없고 뒤에 _가 붙기 까지의 모든 문자열을 diagnosis로 정의
                     # _(.) : _이후 단 한 글자를 gender로 정의
                     # (.)(.*) : (.) 이후 열 이름의 끝까지를 age로 정의
                     values_to = "count")
    #> # A tibble: 405,440 x 8
    #>    country     iso2  iso3   year diagnosis gender age   count
    #>    <chr>       <chr> <chr> <int> <chr>     <chr>  <chr> <int>
    #>  1 Afghanistan AF    AFG    1980 sp        m      014      NA
    #>  2 Afghanistan AF    AFG    1980 sp        m      1524     NA
    #>  3 Afghanistan AF    AFG    1980 sp        m      2534     NA
    #>  4 Afghanistan AF    AFG    1980 sp        m      3544     NA
    #>  5 Afghanistan AF    AFG    1980 sp        m      4554     NA
    #>  6 Afghanistan AF    AFG    1980 sp        m      5564     NA
    #>  7 Afghanistan AF    AFG    1980 sp        m      65       NA
    #>  8 Afghanistan AF    AFG    1980 sp        f      014      NA
    #>  9 Afghanistan AF    AFG    1980 sp        f      1524     NA
    #> 10 Afghanistan AF    AFG    1980 sp        f      2534     NA
    #> # ... with 405,430 more rows
    ```
    - **a.(a)** 에서는 구분자 _를 이용한 `names_sep`의 기본적인 사용 방법을 볼 수 있습니다. 

    - **a.(b)** 는 special name, **.value**를 이용한 `name_sep`의 응용 활용입니다. **.value**는 `pivot_loger()`의 대상이 되는 열 이름 자체에 **value**가 있을 때 사용할 수 있는 방법입니다. 선언된 구분자와 **.value**의 위치에 따라 무엇을 value로 저장할 것인지 설정됩니다. 위의 경우, **.value_Country**의 형태로 선언되었으므로 Country에 해당하는 USA와 Eurupe을 Country열의 value로 저장하고 나머지 dob와 gender를 각각의 열로 저장된 것입니다. 

    - **b.** 는 `names_pattern`의 예제입니다. `names_pattern`은 `name_sep`보다 더 복잡하게 구성된 열 이름을 구분하기 위해 사용됩니다. `names_pattern`에 선언된 문자열은 [정규표현식](https://cran.r-project.org/web/packages/stringr/vignettes/regular-expressions.html)으로 다른 프로그래밍 언어에서 사용하는 것과 크게 다르지 않습니만... 저는 자주 까먹으므로 따로 정리해보도록 하겠습니다. 다시 돌아와서, 결과 코드에서 보시는 것과 같이 `names_pattern`에 보이는 `(.*)`, `(.)`, `(.*)`에 해당하는 문자열을 차례로 `names_to`에 선언된 `diagnosis`, `gender`, `age`에 분할해서 저장해주는 매우 편리한 parameter입니다. 
    
    
7. names_transform, values_transform : 리스트, `pivot_longe()r`의 결과로 생겨날 열의 타입을 바꾸기 위한 parameter입니다. 리스트 형태로 선언해주어야 하므로 **b.**의 예제에서 `names_transform = list(age = as.integer)`를 추가해주면 age열의 value들이 character에서 integer로 바뀌게 됩니다. 


8. names_repair : `c("minimal", "unique", "universal", "check_unique")` 중 하나, 여러 열을 대상으로 `pivot_longer()`을 시행하는 경우에 발생할 수 있는 열 이름 중복을 막기 위한 parameter입니다. 위의 4가지 세팅이 존재하며 default 세팅은 `"chek_unique"`로 되어 있으며 이는 중복 열이 발생하는 경우에 에러를 리턴하는 옵션입니다. 자세한 내용은 [`vctrs::vec_as_names()`](https://vctrs.r-lib.org/reference/vec_as_names.html)를 참고하시길 바랍니다. 


9. values_drop_na : 논리형, `TRUE` or `FALSE`로 value에 NA가 포함된 경우 해당 행을 drop할지 하지 않을지 설정하는 parameter입니다. 


10. names_ptypes, values_ptypes : ????, 이 부분을 좀 더 공부해보고 싶었지만 [Data Wrangling](https://dcl-wrangle.stanford.edu/pivot_advanced.html)의 예제나 [R for Data Science](https://r4ds.had.co.nz/tidy-data.html#pivoting)의 12.3.3 예제와 똑같은 코드를 써도 ptype을 바꿀 수 없다는 에러가 발생합니다. 다만 [Data Wrangling](https://dcl-wrangle.stanford.edu/pivot_advanced.html)의 결과물을 보면 `names_transform`, `values_transform`와 크게 다르지 않은 것처럼 보입니다. 개인적으로 ptype, 즉 **prototype**은 S4 classes object에서 설정하는 것으로 알고 있는데 `pivot_longer()`에서 어떤 역할을 하는지 좀 더 알아봐야 할 것 같습니다. 


지금까지 여러 열들을 세로로 길게 쌓아 세로형 데이터셋을 만드는 `pivot_longer()`에 대해 알아보았습니다. 이제 데이터셋의 value을 넓게 퍼트려서 가로형 데이터셋을 만드는  `pivot_wider()`에 대해 알아보겠습니다. 




# **Wider**


`pivot_wider()`는 하나의 열에 속한 여러 value를 각각의 열로 펼쳐 넓은 형태의 데이터셋을 만들기 위해 사용합니다. 말은 길지만 앞서 본 **`pivot_longer()`의 역(inverse)**입니다. `pivot_longer()`의 결과로 열의 개수가 줄고 행의 개수가 늘었던 것과 반대로 `pivot_wider()`의 결과로 열의 개수가 늘고 행의 개수가 줄어듭니다. 



## 예제를 통해 알아보는 `pivot_wider()`의 효과

이번에 사용할 데이터셋은 1993년부터 2015년까지의 항공기 사고를 다룬 `AirCrach`입니다. 



```r
library(vcdExtra)
tibble(AirCrash) # 439 x 5 
#> # A tibble: 439 x 5
#>    Phase   Cause    date       Fatalities  Year
#>    <fct>   <fct>    <date>          <int> <int>
#>  1 landing criminal 1993-09-21         27  1993
#>  2 landing criminal 1993-09-22        108  1993
#>  3 landing criminal 1996-11-23        125  1996
#>  4 landing criminal 2002-05-07        112  2002
#>  5 landing unknown  1993-07-01         41  1993
#>  6 landing unknown  1993-07-31         19  1993
#>  7 landing unknown  1994-05-07          8  1994
#>  8 landing unknown  2000-01-13         22  2000
#>  9 landing unknown  2002-05-07         14  2002
#> 10 landing unknown  2004-02-10         43  2004
#> # ... with 429 more rows
```

439건의 사고기록을 5개의 열로 구분해 저장되어 있음을 볼 수 있습니다. 그 중, `Phase`와 `Cause`가 명목형 변수로 저장되어 있으므로 `Cause`를 대상으로 `pivot_wider()`를 사용해보겠습니다. 



```r
AirCrash %>% 
  pivot_wider(names_from = Cause, # 각각의 열로 퍼트릴 대상이 되는 열
              values_from = Year, # names_from에 의해 생성될 열에 value로 들어갈 열
              values_fill = 0 # 새로 생길 열에 NA가 포함된 경우 대체할 값
              )
#> # A tibble: 439 x 8
#>    Phase date       Fatalities criminal unknown mechanical weather `human error`
#>    <fct> <date>          <int>    <int>   <int>      <int>   <int>         <int>
#>  1 land~ 1993-09-21         27     1993       0          0       0             0
#>  2 land~ 1993-09-22        108     1993       0          0       0             0
#>  3 land~ 1996-11-23        125     1996       0          0       0             0
#>  4 land~ 2002-05-07        112     2002       0          0       0             0
#>  5 land~ 1993-07-01         41        0    1993          0       0             0
#>  6 land~ 1993-07-31         19        0    1993          0       0             0
#>  7 land~ 1994-05-07          8        0    1994          0       0             0
#>  8 land~ 2000-01-13         22        0    2000          0       0             0
#>  9 land~ 2002-05-07         14        0    2002          0       0             0
#> 10 land~ 2004-02-10         43        0    2004          0       0             0
#> # ... with 429 more rows
```

데이터셋, `AirCrah`에서 `Cause`는  criminal, unknown, mechanical, weather, human error, 총 5개의 명목형 값를 갖는 열입니다. `pivot_wider()`의 결과로 5개의 명목형 값이 각각의 열이 되어 데이터셋에 추가되었고 열 안에는 발생한 `Year`가 값으로 들어갑니다. 0으로 보이는 값은 `values_fill = 0`의 결과로 `NA를` 0으로 대체한 것입니다. 


`pivot_wider()`를 사용한 결과로 열의 개수는 3개 늘어난 8개로 변했지만 행의 개수는 439개로 이전과 동일함을 볼 수 있는데 이것은 각각의 사고 기록이 unique하기 때문입니다. 


또 다른 사용 방법으로 [Pivoting](https://tidyr.tidyverse.org/articles/pivot.html#aggregation-1) 에서 볼 수 있는 **Aggregation**에 대해 살펴보겠습니다. 


똑같이 `AirCrash`를 사용할 것이며 편의를 위해 `date`와 `Year`를 제외하겠습니다. 


```r
# 사고 원인과 운항 단계를 이용한 평균 사망자 테이블 만들기
AirCrash %>% 
  select(-c(date, Year)) %>%
  pivot_wider(names_from = Cause, 
              values_from = Fatalities, 
              values_fn = mean)
#> # A tibble: 5 x 6
#>   Phase    criminal unknown mechanical weather `human error`
#>   <fct>       <dbl>   <dbl>      <dbl>   <dbl>         <dbl>
#> 1 landing       93     28.9       34.9    31            43.3
#> 2 en route     219.    42.4       55.9    34.2          64.9
#> 3 take-off       2     18.1       27.2    21.7          65.7
#> 4 standing       4     NA          2      NA            NA  
#> 5 unknown       NA     27         NA       1           141
```


`pivot_wider()`를 사용해 `Phase`와 `Cause` 사이의 평균 사망자 테이블을 만든 결과입니다. `names_from`으로 퍼트릴 열을 정하고 `values_from`으로 열에 들어갈 값을 정합니다. 그 후 `values_fn`으로 `values_from`을 사용해 들어간 값에 적용할 함수를 설정하는 것으로 위의 결과가 만들어집니다. 

`values_fn`을 적용하지 않은 상태는 아래와 같으며 tibble 안에 list로 `Fatalities`의 value가 들어가있음을 볼 수 있습니다. 



```r
AirCrash %>% 
  select(-c(date, Year)) %>%
  pivot_wider(names_from = Cause, 
              values_from = Fatalities)
#> # A tibble: 5 x 6
#>   Phase    criminal   unknown    mechanical weather    `human error`
#>   <fct>    <list>     <list>     <list>     <list>     <list>       
#> 1 landing  <int [4]>  <int [18]> <int [19]> <int [55]> <int [114]>  
#> 2 en route <int [16]> <int [25]> <int [29]> <int [24]> <int [63]>   
#> 3 take-off <int [1]>  <int [8]>  <int [24]> <int [3]>  <int [29]>   
#> 4 standing <int [2]>  <NULL>     <int [2]>  <NULL>     <NULL>       
#> 5 unknown  <NULL>     <int [1]>  <NULL>     <int [1]>  <int [1]>
```

두 가지의 방법으로 `pivot_wider()`의 사용 방법을 간단히 보았습니다. 이전에 본 `pivot_longer()`과 비슷한 부분도, 다른 부분도 있으니 `pivot_wider()`의 parameter를 자세히 알아보겠습니다. 


## **`pivot_wider()`의 parameter**



```r
# pivot_wider(
#   data, # 데이터셋 설정
#   id_cols = NULL, # names_와 values_를 적용하지 않을 열을 설정
#   names_from = name, # value를 각각의 열로 퍼트릴 열을 설정
#   names_prefix = "", # 퍼트릴 열의 이름을 수정
#   names_sep = "_", # 퍼트릴 열의 구분자를 설정
#   names_glue = NULL, # 퍼트릴 열들의 이름을 수정하고자 할 때 패턴을 설정
#   names_sort = FALSE, # 퍼트릴 열들을 미리 정렬시키고자 할 때 설정
#   names_repair = "check_unique", # 퍼트릴 열의 중복 확인
#   values_from = value, # 생성될 열들에 들어갈 값을 가진 열을 설정
#   values_fill = NULL, # 생성될 열들에 NA가 value로 들어가 있는 경우 대체할 value를 설정
#   values_fn = NULL, # 생성될 열들에 들어간 value들에 적용할 함수를 설정
#   ...
# )
```


1. data : `pivot_wider()`를 적용할 데이터셋을 입력해줍니다. 파이프라인을 사용하신다면 생략하셔도 됩니다. 


2. **id_cols** : [tidyselect](https://tidyselect.r-lib.org/reference/language.html), 형태를 그대로 유지할 열을 선택합니다. default 설정에 의해 `names_from`과 `values_from`에 선언된 열을 제외한 모든 열은 `id_cols`에 자동으로 선언됩니다. 

    - `pivot_longer()`에서 `cols` parameter가 변환할 모든 열을 선택하는 것과 반대로 `id_cols`는 적용하지 않는 모든 열을 선택합니다. 



3. **names_from**, **values_from** : [tidyselect](https://tidyselect.r-lib.org/reference/language.html), `names_from`은 명목형 값을 이용해 넓게 퍼트릴 열을, `values_from`은 `names_from`으로 생성될 열들에 들어갈 값을 가진 열을 설정해줍니다. 두 parameter 모두 tidyselect을 지원하고 있으므로 여러 열을 한 번에 선택하는 것이 가능합니다. 

    - `pivot_longer()`의 `names_to`, `values_to`와 비슷한 기능을 하는 parameter입니다. 



4. **names_glue** : 문자열(character), 넓게 퍼트려질 열의 이름을 수정하기 위한 parameter로 이름에 대한 패턴을 설정해 사용합니다. 예를 들어, 위의 *사고 원인과 운항 단계를 이용한 평균 사망자 테이블 만들기* 코드에 `Cause_`라는 말머리를 추가해보겠습니다. 

    
    ```r
    # names_glue를 사용해 열 이름 편집하기
    AirCrash %>% 
      select(-c(date, Year)) %>%
      pivot_wider(names_from = Cause, 
              values_from = Fatalities, 
              values_fn = mean, 
              names_glue = "Cause_{Cause}") # {  } 
    #> # A tibble: 5 x 6
    #>   Phase Cause_criminal Cause_unknown Cause_mechanical Cause_weather
    #>   <fct>          <dbl>         <dbl>            <dbl>         <dbl>
    #> 1 land~            93           28.9             34.9          31  
    #> 2 en r~           219.          42.4             55.9          34.2
    #> 3 take~             2           18.1             27.2          21.7
    #> 4 stan~             4           NA                2            NA  
    #> 5 unkn~            NA           27               NA             1  
    #> # ... with 1 more variable: `Cause_human error` <dbl>
    ```

    문자열의 형태로 사용 가능하며 `{...}`에 `names_from`에 사용했던 열 이름을 넣어주시면 해당 열에 속한 모든 명목형 값으로 만들어질 열 이름들에 `names_glue`에서 정의한 패턴이 적용돼 수정됩니다.

    - `pivot_longer()`의 `names_pattern`이 열 이름들의 패턴을 받아 패턴대로 열 이름을 쪼개어 각각의 새로운 열로 저장했다면 `pivot_wider()`의 `names_glue`는 퍼트려질 열 이름들의 패턴을 정의해 수정합니다. 


5. names_sort : 논리형, `pivot_wider()`의 결과로 만들어질 열 이름들을 알파벳 순서대로 정렬할 지에 대한 여부를 설정하는 parameter입니다. 


6. values_fill : 숫자 혹은 문자열, 새로 생겨나는 열의 `NA`를 대체하기 위한 parameter입니다. 

7. **values_fn** : 함수의 이름, summary statistic을 이용한 table을 만들기 위한 parameter입니다. 각 셀의 value에 `values_fn`에서 설정된 함수를 적용합니다. 


8. names_prefix, names_sep, names_repair : `pivot_longer()`에서 설명한 것과 같은 방식으로 사용되는 parameter입니다. 


지금까지 `pivot_wider()`에 대한 기본적인 내용을 예제를 통해 알아보았습니다. 여기까지만 알아도 pivoting에는 문제가 없겠지만 [Pivoting](https://tidyr.tidyverse.org/articles/pivot.html#longer-then-wider-1)를 참고해 `pivot_longer()`와 `pivot_wider()`를 함께 쓰는 방법을 소개해보겠습니다. 



# **Longer, then wider**


`pivot_longer()` 혹은 `pivot_wider()`, 하나만 사용해서는 할 수 없는 작업들이 있습니다. 어찌어지 구현할 수 있더라도 비효율적인 코드로 구성될 것입니다. 그런 경우, 두 함수를 순차적으로 사용해 문제해결을 하는 방법이 있습니다. 이제 예제를 통해 문제 상황과 해결 방법을 보여드리겠습니다. 


## World Bank 데이터셋을 통한 시연
`world_bank_pop`는 World Bank로부터 얻은 *각 나라의 00년부터 18년까지의 인구수*를 정리한 데이터셋입니다. 


```r
world_bank_pop
#> # A tibble: 1,056 x 20
#>    country indicator `2000` `2001` `2002` `2003`  `2004`  `2005`   `2006`
#>    <chr>   <chr>      <dbl>  <dbl>  <dbl>  <dbl>   <dbl>   <dbl>    <dbl>
#>  1 ABW     SP.URB.T~ 4.24e4 4.30e4 4.37e4 4.42e4 4.47e+4 4.49e+4  4.49e+4
#>  2 ABW     SP.URB.G~ 1.18e0 1.41e0 1.43e0 1.31e0 9.51e-1 4.91e-1 -1.78e-2
#>  3 ABW     SP.POP.T~ 9.09e4 9.29e4 9.50e4 9.70e4 9.87e+4 1.00e+5  1.01e+5
#>  4 ABW     SP.POP.G~ 2.06e0 2.23e0 2.23e0 2.11e0 1.76e+0 1.30e+0  7.98e-1
#>  5 AFG     SP.URB.T~ 4.44e6 4.65e6 4.89e6 5.16e6 5.43e+6 5.69e+6  5.93e+6
#>  6 AFG     SP.URB.G~ 3.91e0 4.66e0 5.13e0 5.23e0 5.12e+0 4.77e+0  4.12e+0
#>  7 AFG     SP.POP.T~ 2.01e7 2.10e7 2.20e7 2.31e7 2.41e+7 2.51e+7  2.59e+7
#>  8 AFG     SP.POP.G~ 3.49e0 4.25e0 4.72e0 4.82e0 4.47e+0 3.87e+0  3.23e+0
#>  9 AGO     SP.URB.T~ 8.23e6 8.71e6 9.22e6 9.77e6 1.03e+7 1.09e+7  1.15e+7
#> 10 AGO     SP.URB.G~ 5.44e0 5.59e0 5.70e0 5.76e0 5.75e+0 5.69e+0  4.92e+0
#> # ... with 1,046 more rows, and 11 more variables: `2007` <dbl>, `2008` <dbl>,
#> #   `2009` <dbl>, `2010` <dbl>, `2011` <dbl>, `2012` <dbl>, `2013` <dbl>,
#> #   `2014` <dbl>, `2015` <dbl>, `2016` <dbl>, ...
```


`world_bank_pop`을 이용해 하고 싶은 작업은 1) 2000-2018의 열을 하나의 열로 쌓은 후 2) `indicator`를 의미에 맞게 쪼개어 열로 퍼트리는 것 입니다. 1)에는 `pivot_longer()`를, 2)에는 `pivot_wider()`를 사용하면 되겠군요. 아래의 코드는 [Pivoting](https://tidyr.tidyverse.org/articles/pivot.html#world-bank-1)의 코드입니다. 

```r
# Pivoting의 예제 코드 
world_bank_pop %>% 
  pivot_longer(cols = `2000`:`2017`, 
               names_to = "year", 
               values_to = "value") %>% 
  separate(indicator, c(NA, "area", "variable")) %>% 
  pivot_wider(names_from = variable, values_from = value)
#> # A tibble: 9,504 x 5
#>    country area  year   TOTL    GROW
#>    <chr>   <chr> <chr> <dbl>   <dbl>
#>  1 ABW     URB   2000  42444  1.18  
#>  2 ABW     URB   2001  43048  1.41  
#>  3 ABW     URB   2002  43670  1.43  
#>  4 ABW     URB   2003  44246  1.31  
#>  5 ABW     URB   2004  44669  0.951 
#>  6 ABW     URB   2005  44889  0.491 
#>  7 ABW     URB   2006  44881 -0.0178
#>  8 ABW     URB   2007  44686 -0.435 
#>  9 ABW     URB   2008  44375 -0.698 
#> 10 ABW     URB   2009  44052 -0.731 
#> # ... with 9,494 more rows
```


데이터셋의 문자열값을 가진 열을 설정에 따라 여러 열로 쪼개주는 `separate`를 쓴 예제코드 대신 pivot을 여러번 사용한 아래의 코드로 같은 결과를 만들 수 있습니다. 효율적인 코드는 아니지만 이런 방법도 가능하다는 것을 보여드리고 싶었습니다. 



```r
# 똑같은 결과를 만들기 위한 나의 코드
world_bank_pop %>% 
  pivot_longer(cols = `2000`:`2017`, 
               names_to = "year", 
               values_to = "value") %>% 
  mutate(indicator = str_remove(indicator, "^SP\\.")) %>% 
  pivot_wider(names_from = indicator, values_from = value) %>% 
  pivot_longer(cols = URB.TOTL:POP.GROW,
               names_to = c("area", "variable"),
               names_pattern = "(.*)\\.(.*)") %>%
  pivot_wider(names_from = variable, values_from = value) %>% 
  select(country, area, year, TOTL, GROW) %>% 
  arrange(desc(area))
#> # A tibble: 9,504 x 5
#>    country area  year   TOTL    GROW
#>    <chr>   <chr> <chr> <dbl>   <dbl>
#>  1 ABW     URB   2000  42444  1.18  
#>  2 ABW     URB   2001  43048  1.41  
#>  3 ABW     URB   2002  43670  1.43  
#>  4 ABW     URB   2003  44246  1.31  
#>  5 ABW     URB   2004  44669  0.951 
#>  6 ABW     URB   2005  44889  0.491 
#>  7 ABW     URB   2006  44881 -0.0178
#>  8 ABW     URB   2007  44686 -0.435 
#>  9 ABW     URB   2008  44375 -0.698 
#> 10 ABW     URB   2009  44052 -0.731 
#> # ... with 9,494 more rows
```


## Multi-choice


[Pivoting](https://tidyr.tidyverse.org/articles/pivot.html#multi-choice-1)에 나온 또 다른 예제입니다. 이 예제는 [Maxime Wack이 제시한 이슈](https://github.com/tidyverse/tidyr/issues/384)에 의해 추가된 예제로 `pivot_longer()`과 `pivot_wider()`를 이용한 명목형 변수의 recording을 보여주고 있습니다. 천천히 따라가보겠습니다. 


```r
multi <- tribble(
  ~id, ~choice1, ~choice2, ~choice3,
  1, "A", "B", "C",
  2, "C", "B",  NA,
  3, "D",  NA,  NA,
  4, "B", "D",  NA
)
```

사용할 예제 데이터입니다. 이제 이 데이터를 이용해 1) `pivot_longer()`를 이용해 `NA`를 없앤 후, 2) `pivot_wider()`로 A부터 D까지의 선택 여부를 보여주는 데이터셋으로 만들어보겠습니다. 



```r
multi2 <- multi %>%
  pivot_longer(-id, values_drop_na = TRUE) %>%
  mutate(checked = TRUE)
multi2
#> # A tibble: 8 x 4
#>      id name    value checked
#>   <dbl> <chr>   <chr> <lgl>  
#> 1     1 choice1 A     TRUE   
#> 2     1 choice2 B     TRUE   
#> 3     1 choice3 C     TRUE   
#> 4     2 choice1 C     TRUE   
#> 5     2 choice2 B     TRUE   
#> 6     3 choice1 D     TRUE   
#> 7     4 choice1 B     TRUE   
#> 8     4 choice2 D     TRUE
```
`pivot_longer()`를 사용해 choice1부터 3까지를 name열에 넣어 `NA`를 없앤 후 `mutate()`로 `checked`을 만든 모습입니다. 이제 여기에 `pivot_wider()`를 적용해 A부터 D까지 선택 여부에 따라 `checked`에 TRUE or FALSE가 들어가는 데이터셋으로 바꿔 보겠습니다. 


```r
multi2 %>% 
  pivot_wider(
    id_cols = id,
    names_from = value,
    values_from = checked,
    values_fill = FALSE
  )
#> # A tibble: 4 x 5
#>      id A     B     C     D    
#>   <dbl> <lgl> <lgl> <lgl> <lgl>
#> 1     1 TRUE  TRUE  TRUE  FALSE
#> 2     2 FALSE TRUE  TRUE  FALSE
#> 3     3 FALSE FALSE FALSE TRUE 
#> 4     4 FALSE TRUE  FALSE TRUE
```

이 예제에서 볼 수 있는 것처럼 pivoting으로 데이터셋을 변환할 수 있습니다. 

# **Summary**


**Pivoting**은 **tidy data**를 만들기 위해서도, 분석에서 필요한 여러 요약 통계량을 확인하기 위한 데이터셋으로 만들거나 곧바로 구해볼 수도 있는 매우 유용한 기술입니다. 제대로 사용하면 데이터 분석에서 가장 많은 시간을 차지하는 전처리에서도 효율성을 더할 수 있을 것입니다. 

이번 포스트를 작성하며 두 함수의 parameter까지 찬찬히 뜯어보고 예제에 적용해봄으로써 Pivoting에 대해 많은 것을 배웠습니다. 추후 더 좋은 예제가 있다면 추가해서 공유해보도록 하겠습니다. 감사합니다. 



















