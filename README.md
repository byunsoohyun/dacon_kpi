# dacon_kpi
- 참여 공모전 정보 : https://dacon.io/competitions/official/236248/overview/description
- 1차 : 리뷰 평점과 구매의 상관관계 파악하기(~4/29)


# 1차 데이터 전처리
- 사용 데이터 : reviews, order_items, order

## 1. 리뷰 평점 데이터 추출
### 1.1 reviews 데이터 내 Order_id 중복값 처리 과정
#### 🤷‍♀️ Order_id 컬럼 내 겹치는 항목이 있는 것을 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/7395e242-15f3-44ea-ad58-05591ec8f0fa)
- 겹치는 항목 중 임의의 Order_id 를 조회하여 다른 시기에 작성된 리뷰임을 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/eb58a4c9-3780-4fb3-8f4f-285394c2fe2b)
- Order_id 내 다른 제품을 구매했을 가능성을 고려하여 order_items 에서 구매한 제품을 확인했으나, 1가지의 제품만 구매
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/66362bc1-46d1-412b-a52d-d47bdef87c8b)

✅ 국내 판매 채널에 존재하는 한달 사용 후기와 같은 것으로 취급하여, 동일 Order_id, Product_id 일 경우 평균 값으로 계산하고자 함

✅ 평균값으로 하고자 하는 이유는, 위의 예시와 달리 다른 점수를 부여한 리뷰가 있을 경우 대비(실제로 있었음)

#### 🤷‍♀️ 동일한 Order_id 내에 2가지 이상의 제품을 구매한 것을 확인함
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/2e23168d-147c-4b89-b38e-a0872d2c80db)
- 해당 Order_id 를 조회했을 때 작성된 리뷰는 1개, 어떤 제품에 대한 리뷰인지 확인 불가능
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/414f31a2-2fa2-4414-becf-dc2c53a63358)

✅ 이 경우 동일 Order_id 에 속한 모든 제품에 대한 점수로 가정하고, 동일 제품을 2가지 이상 담은 경우 1개의 리뷰로 취급함

✅ reviews 데이터에 다른 데이터를 합하기보단 order_items 데이터에 reviews 데이터 합하기로 결정

### 1.2 reviews 데이터 내 Review_id 중복값 처리 과정
#### 🤷‍♀️ Review_id 컬럼 내 겹치는 항목이 있는 것을 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/fa8e90bd-6049-4008-bfdc-889461640a02)
- Review_id 가 같으나, Order_id 가 다른 것을 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/a961dd70-4545-4fcc-b83a-7bf245c485c8)

##### order_items 데이터에서 Order_id 마다 제품 구매 내역을 확인해보고
- 서로 다른 제품을 구매했으나, 동일 Review_id 로 분류된 것 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/f4d36157-55be-4f95-ab94-217fc614aecd)

##### order 데이터에서 Customer_id 를 확인하고자 함
- Order_id 를 각각 조회했으나, Customer_id 및 Customer_unique_id 는 서로 다른 데이터만 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/f0e910af-e197-47a1-bfe0-5d0a9cde0065)

##### customers 데이터에서 Customer_id 조회하여 Customer_unique_id 겹치는 가 확인하고자 함
- Customer_unique_id 가 여러 개의 Customer_id 를 갖고 있어 위를 실행함
- 조회 결과 같은 Customer_id 를 가진 것 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/e1a356ea-c2ad-4d71-a4da-bde0e3b62aad)

#### 🤷‍♀️ Review_id 가 동일하면 Customer_unique_id 같은가에 대해 확인하고자 함
- 2개 이상의 Review_id 인 것을 다시 조회함
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/79de9048-fa6f-4513-af88-414ec73816db)
- Review_id 가 같으나 Customer_unique_id 가 다른 경우를 확인함
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/0682693a-ab10-4420-ac6d-0c1adbedd7ea)

✅ Review_id 는 고려하지 않는 것으로 결정

✅ Customer_unique_id 가 같더라도 다른 오더 아이디기 때문에 그냥 별개로 분류함(또 다른 구매로 분류)

✅ 최종적으로 Order_id 가 일치할 경우 해당되는 리뷰의 평균 점수를 구하는 것으로 결정

### 1.3 리뷰 평점 구하기
- reviews 에서 Order_id 와 Review_score 만 따로 추출(re_sc)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/73f6672a-1e65-42e3-9401-1150e3ccb990)
- Order_id 가 같은 경우 평균을 구하여 re_sc 재정의
- 마침 예시가 동일 Order_id 인데 Review_score 가 다른 리뷰
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/7d735389-c044-4383-94ff-334621b3a0f4)

## 2. order_items 데이터에 리뷰 평점 컬럼 붙이기
### 2.1 order_items 데이터 내 Order_id 중복값 처리 과정
#### 🤷‍♀️ Order_id 컬럼 내 겹치는 항목이 있는 것을 확인
- order_items 데이터를 사용하는 이유는 Product_id , Seller_id , Price 정보가 있기 때문
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/e47bfdfe-ce46-4fd6-bbf0-d9fb5f065e00)
- 일치하는 Order_id 조회 결과 2가지 이상의 제품을 구매한 것 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/e27af4d9-cd86-4cca-b4b6-3b507350d1fa)
#### 🤷‍♀️ order_items 데이터에서 Order_id , Product_id가 일치하는 경우 평균을 구하도록 했으나, 오류가 생김
- 동일 Product_id 가 여러 Seller_id 에서 판매되는 것 확인되어 여기서 2개의 갈래로 나누었음
- Counts 컬럼 무시 요망 
> (1) Seller_id 와 Product_id 한 세트로 분석(해당 판매자의 제품에 대한 점수) (order_items_ys)
 > ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/b18ca162-729d-4d04-9893-5a8b5b145620)

> (2) 판매자와 상관 없이 Product_id 자체만으로 분석(order_items_ns)
 > ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/85d86348-a386-40f0-92ca-5740c831e727)

### 2.2 Seller_id 와 Product_id 세트 분석
- order_items_ys 와 re_sc 합치기
- order_items_ys & re_sc -> rs_ys
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/dc722b80-a8f5-47ed-9f50-88b6a03d5234)

#### 2.2.1 rs_ys 데이터 정리
- 필요한 컬럼만 추출하여 rs_ys3 로 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/88167b08-76ea-4516-9789-98ead47c2b5a)
- Product_id , Seller_id 가 일치한 행들의 평균 Review_score 로 rs_ys4 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/1be533ea-77c9-4fd5-91e3-92ba7207b251)

✅ Seller_id 와 Product_id 한 세트로 본 평균 리뷰 점수 삽입 완료 (rs_ys4)

#### 2.2.2 Seller_id 와 Product_id 세트일 경우 가격
- 판매 가격을 보기 위해 필요한 컬럼만 추출하여 pr_ys 로 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/07f1fc09-ca27-4dab-aab7-5cd9c0f39387)
- 행 합치기 위해 생성한 임의의 컬럼 Row_Number 생성(무시할 컬럼)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/d5250c4a-c051-442d-b025-6cfade9e5593)
- 기존 4가지 컬럼이 동일한 것의 행 수를 추가한 Counts
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/f965b04f-c4b9-412f-b98e-aee13d677be8)
- 기존 4가지 컬럼 + Counts 컬럼이 이 동일한 경우 합치기
- 하나라도 일치하지 않으면 컬럼이 합쳐지지 않아서 Order_id 도 동일한 것으로 취급함..
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/6f5ea78f-ec6d-4bf3-8a56-ac910580d5b4)
- Row_Number , Order_id 컬럼 삭제
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/5beb8692-d73d-43db-ae20-922882c5943a)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/e34b2534-d68b-47a3-88ed-03f0933d583d)
- Product_id , Seller_id , Price 같은 경우 Counts 합산
- Order_id 컬럼을 진작에 삭제해야 했는데 늦게 해서 합산해도 무방함
- 앞의 두 컬럼이 일치하는데 가격이 다른 것이 있어서 3가지가 같은 경우를 합치기로 함
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/f2343cc8-e581-4777-8935-219376785e5b)
- 판매액을 구하기 위해 Price * Counts 를 한 Total 컬럼 추가
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/ebefd4b5-c259-421d-ad81-7c4cf78f1f2b)
- Product_id 와 Seller_id 이 동일한 행끼리 합치기
- 이제 Price는 무시해도 괜찮음(개별 가격이기 때문에 추후 삭제함)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/d278d08e-74e2-41b5-b82d-4c3f9b849f31)
- corr 함수 사용하기 위해 평점과 합침 (score_price_ys)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/4d4074e9-e3e4-44c2-9155-c3e937272124)

### 2.3 Product_id 자체 분석
- order_items_ns 와 re_sc 합치기
- order_items_ns & re_sc -> rs_ns
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/889bbd8f-520f-404b-a686-283868ad1a22)
#### 2.3.1 rs_ns 데이터 정리
- 필요한 컬럼만 추출하여 rs_ns2 로 정의
- 가격 달라도 동일한 제품으로 취급
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/74739e74-91fa-4e7f-b458-bc65977edabd)
- Product_id 동일한 것들 리뷰 평점 구하기 (rs_ns3)
- ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/767e6a49-4c13-4b19-b165-49ae80746df5)


✅ Product_id 의 평균 리뷰 점수 삽입 완료 (rs_ns3)

#### Product_id 자체일 경우 가격
- 판매 가격을 보기 위해 필요한 컬럼만 추출하여 pr_ns 로 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/305fd1ca-5624-456a-922e-9e10623ae41b)
- 행 합치기 위해 생성한 임의의 컬럼 Row_Number 생성(무시할 컬럼)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/03e5dddc-ed71-4158-a2a0-eb274f7a4837)
- 기존 2가지 컬럼이 동일한 것의 행 수를 추가한 Counts
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/9b38a292-c25d-4b05-a283-644a3357cd91)
- 기존 2가지 컬럼 + Counts 컬럼이 동일한 경우 합치기
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/61d050a2-6ba1-4651-bf80-99cc709ab9da)
- Row_Number 컬럼 버리고
- Price * Counts 한 Total 컬럼 추가 (pr_ns2)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/bb8c7e4b-7b9c-4c1f-8ae2-2d7e451751b6)
- Product_id 동일할 경우 sum (pr_ns3)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/76a1d6c2-f176-4e33-a8f0-f6c1c33f60e2)
- corr 함수 사용하기 위해 rs_ns3 와 pr_ns3 합침
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/bd2204f0-5e6d-4fd8-9b9a-3edc77620557)






# 1차 결론

## 1. Seller_id 와 Product_id 가 동시에 동일할 때
### 1.1 Review_score 와 Total_price(총 판매 금액) 의 상관 관계
- 0.007800808718716167 : 거의 없음
- Total -> Total_price 로 컬럼명 변경
>![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/bf503760-f626-47c8-ac9d-58b54ba56965)


### 1.2 Review_score 와 Counts(총 판매 수량) 의 상관 관계
- -0.01025935443501701 : 거의 없음
- Total -> Total_price 로 컬럼명 변경
>![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/3ffaf0f2-676e-4334-85a0-3cc8ae67ab70)


## 2. Product_id 만을 기준으로
### 2.1 Review_score 와 Total_price(총 판매 금액) 의 상관 관계
- -0.007445772259951883 : 거의 없음
- Total -> Total_price 로 컬럼명 변경
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/f938691b-169e-4d66-a059-c069ef99f6f8)

### 2.2 Review_score 와 Counts(총 판매 수량) 의 상관 관계
- -0.009771342784221454 : 거의 없음
- Total -> Total_price 로 컬럼명 변경
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/10f4c8d6-47ca-4317-8608-9371e3b17898)


## 3. 다른 방식 분석
### 제품당 주문 횟수가 2번 이상인 것들을 뽑아서 리뷰와 판매 상관계수 보기
- 구매가 1번 뿐이면 다음 구매에 영향 미쳤는가는 아무도 모름(다음 구매가 없으니까)
### 3.1 제품별 판매 횟수_Seller_id 있는 버전
- Order_items 에서 필요한 컬럼만 불러서 order_counts_ys 로 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/76dce55e-ffc4-476a-91d9-40fa60fdb94f)
- 동일 주문 내 동일 셀러/상품을 묶어서 중복이 없도록 하는 order_counts_ys2 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/bffe204d-5c21-4bf0-b697-37647137e4ca)
- 구매 횟수 확인을 위한 Order_counts 컬럼 추가
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/17a72dd3-64d6-48f4-bac6-ad1774276d04)
> 확인해보니 제대로 들어갔어요...
- Product_id , Seller_id , Order_counts 모두 일치하는 경우 하나의 행으로 만드는 작업 실행 후 order_counts_ys3 로 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/5077c8fd-9309-47d5-b222-30941aebcc12)
- 필요없는 컬럼 삭제
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/14e23f85-7558-4703-a89f-2b0f9f2464aa)
- 주문이 2번 이상 있었던 제품만 ... order_counts_ys4
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/2af876d4-61d0-4c97-8352-ec1f2db2aece)
- 2회 이상 구매가 있던 것에 결론에 사용한 df 추가 score_price_ys3
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/5306eeee-940f-4f9d-9cbc-0cd9af472855)
- 상관계수 확인 : 순서대로 판매 횟수, 판매 수량, 판매 금액
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/40f86fa0-6db3-4be1-ae9b-1936884d746b)


### 3.2 제품별 판매 횟수_Seller_id 없는 버전
- Order_items 에서 필요한 컬럼만 불러서 order_counts_ns 로 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/79aca716-97fc-4747-96e0-677988ea8085)
- 동일 주문 내 동일 상품을 묶어서 중복이 없도록 하는 order_counts_ns2 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/ecd1cb09-d4a8-4943-a412-6a980c5dc97b)
- 구매 횟수 확인을 위한 Order_counts 컬럼 추가
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/3683321f-452d-4557-bd51-2c2753f0ff60)
> 확인해보니 제대로 들어갔어요...
- Product_id , Order_counts 모두 일치하는 경우 하나의 행으로 만드는 작업 실행 후 order_counts_ns3 로 정의
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/e97e312d-1811-44a6-a9a9-b345790ba8a2)
- 필요없는 컬럼 삭제
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/5b47a0b6-bf03-4829-aa70-3c370ce855fa)
- 주문이 2번 이상 있었던 제품만 ... order_counts_ns4
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/6798fe65-95e4-465e-b0c1-55564d7ee794)
- 2회 이상 구매가 있던 것에 결론에 사용한 df 추가 score_price_ys4
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/78cb9559-2ae4-4630-af32-fc8a78c10f0c)
- 상관계수 확인 : 순서대로 판매 횟수, 판매 수량, 판매 금액
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/9b540a81-ab37-4f3c-a93a-ef1ba9ebcb74)
