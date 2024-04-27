# dacon_kpi
- 참여 공모전 정보 : https://dacon.io/competitions/official/236248/overview/description
- 1차 : 리뷰 평점과 구매의 상관관계 파악하기(~4/29)


## 1차 데이터 전처리

### 사용 데이터 : reviews, order_items, order

### 1. reviews 데이터 내 Order_id 중복값 처리 과정
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

### 2. reviews 데이터 내 Review_id 중복값 처리 과정
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

##### 🤷‍♀️ Review_id 가 동일하면 Customer_unique_id 같은가에 대해 확인하고자 함
- 2개 이상의 Review_id 인 것을 다시 조회함
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/79de9048-fa6f-4513-af88-414ec73816db)
- Review_id 가 같으나 Customer_unique_id 가 다른 경우를 확인함
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/0682693a-ab10-4420-ac6d-0c1adbedd7ea)

✅ Review_id 는 고려하지 않는 것으로 결정

✅ Customer_unique_id 가 같더라도 다른 오더 아이디기 때문에 그냥 별개로 분류함(또 다른 구매로 분류)

✅ 최종적으로 Order_id 가 일치할 경우 해당되는 리뷰의 평균 점수를 구하는 것으로 결정

### 3. 리뷰 평점 구하기
- reviews 에서 Order_id 와 Review_score 만 따로 추출(re_sc)
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/73f6672a-1e65-42e3-9401-1150e3ccb990)
- Order_id 가 같은 경우 평균을 구하여 re_sc 재정의
- 마침 예시가 동일 Order_id 인데 Review_score 가 다른 리뷰
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/7d735389-c044-4383-94ff-334621b3a0f4)



## 1차 결론

1. Seller_id 와 Product_id 가 동시에 동일할 때 Review_score 와 Total_price(총 판매 금액) 의 상관 관계
- -0.007800808718716167 : 거의 없음
>![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/05d0a5cc-7607-46d7-831e-2832a725e5eb)
2. Seller_id 와 Product_id 가 동시에 동일할 때 Review_score 와 Counts(총 판매 수량) 의 상관 관계
- -0.01025935443501701 : 거의 없음
>![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/4289ea4b-4277-4061-8935-3d5711141568)
3. Product_id 기준으로 Review_score 와 Total_price(총 판매 금액) 의 상관 관계
-
>
4. Product_id 기준으로 Review_score 와 Counts(총 판매 수량) 의 상관 관계
-
>


