# dacon_kpi
- 참여 공모전 정보 : https://dacon.io/competitions/official/236248/overview/description
- 1차 : 리뷰 평점과 구매의 상관관계 파악하기(~4/29)


## 1차 과정

### 사용 데이터 : reviews, order_items

1. reviews 데이터 내 Order_id 중복값 처리 과정
- Order_id 컬럼 내 겹치는 항목이 있는 것을 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/7395e242-15f3-44ea-ad58-05591ec8f0fa)
- 겹치는 항목 중 임의의 Order_id 를 조회하여 다른 시기에 작성된 리뷰임을 확인
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/eb58a4c9-3780-4fb3-8f4f-285394c2fe2b)
- Order_id 내 다른 제품을 구매했을 가능성을 고려하여 order_items 에서 구매한 제품을 확인했으나, 1가지의 제품만 구매
> ![image](https://github.com/byunsoohyun/dacon_kpi/assets/167173701/66362bc1-46d1-412b-a52d-d47bdef87c8b)

✅ 국내 판매 채널에 존재하는 한달 사용 후기와 같은 것으로 취급하여, 동일 Order_id, Product_id 일 경우 평균 값으로 계산하고자 함

✅ 평균값으로 하고자 하는 이유는, 위의 예시와 달리 다른 점수를 부여한 리뷰가 있을 경우 대비(실제로 있었음)


2. reviews 데이터 내 Review_id 중복값 처리 과정
