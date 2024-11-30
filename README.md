## AI Study와 더불어 실습 내용을 업로드 하는 공간입니다.
### CNN_Dataprocess_Source.ipynb
컨벌루젼 연산을 케라스를 통해 진행하고, Categorical CrossEntropy + softmax를 통해 결과 도출 

# 2018년~2023년 사유별 이혼 건수 예측 코드 설명

이 문서는 2018년부터 2023년까지의 사유별 이혼 건수를 예측하는 Python 코드에 대한 설명입니다.  
예측은 2017년 데이터를 기반으로 **사유별 비율 계산**을 활용하여 진행되었습니다.

---

## 코드 설명

### 1. 연도별로 반복
```python
for year in range(2018, 2024):
```
- 2018년부터 2023년까지의 연도를 순회하면서 예측 데이터를 생성합니다.

---

### 2. 해당 연도의 실제 총 이혼 건수 가져오기
```python
total_cases = actual_data.loc[year, 'Total'] if year in actual_data.index else None
```
- `actual_data`에서 `year`(2018~2023)의 **총 이혼 건수**를 가져옵니다.
- 예: `2023년 Total = 12397`.
- 만약 데이터가 없다면 `None`으로 처리하여 건너뜁니다.

---

### 3. 새로운 행 데이터 생성
```python
new_row = {'Year': year, '소계': total_cases}
```
- `new_row`라는 사전(dictionary)을 생성:
  - `Year`: 현재 연도 (2018~2023).
  - `소계`: 해당 연도의 총 이혼 건수.

---

### 4. 사유별 이혼 건수 계산
```python
for column in df.columns[2:]:
    new_row[column] = (total_cases * df[column].iloc[-1] / df['Total'].iloc[-1]).round().astype(int)
```
- **2017년 데이터**를 기준으로 각 사유별 이혼 건수를 예측.
- 계산 공식:
  \[
  	ext{예측 사유별 건수} = 	ext{미래 총 이혼 건수} 	imes rac{	ext{사유별 건수 (2017년)}}{	ext{총 이혼 건수 (2017년)}}
  \]

#### 예제:
- `2017년 배우자 부정 = 1082`
- `2017년 총 이혼 건수 = 17083`
- `2023년 총 이혼 건수 = 12397`

\[
	ext{2023년 배우자 부정} = 12397 	imes rac{1082}{17083} pprox 785
\]

---

### 5. 새로운 데이터 추가
```python
predicted_actual = pd.concat([predicted_actual, pd.DataFrame([new_row])], ignore_index=True)
```
- 계산된 데이터(`new_row`)를 `predicted_actual` 데이터프레임에 추가.

---

### 6. 불필요한 열 제거
```python
predicted_actual = predicted_actual.drop(columns=['Total'], errors='ignore')
```
- `Total` 열은 계산 과정에서만 사용되므로 최종 데이터에서 제거합니다.

---

### 7. 결과 확인 및 저장
```python
print(predicted_actual)
predicted_actual.to_csv('/content/2023년_사유별_이혼건수_최종.csv', index=False, encoding='utf-8-sig')
```
- 결과를 출력하고, 예측 데이터를 CSV 파일로 저장합니다.

---

## 데이터 예제

### 입력 데이터:
#### 2017년 데이터:
- 총 이혼 건수(`소계`): 17083
- `배우자 부정`: 1082 (비율 ≈ 0.0634)

#### 2023년 실제 총 이혼 건수:
- 총 이혼 건수(`Total`): 12397

---

### 계산 예제:
#### `2023년 배우자 부정` 예측:
1. 2017년 비율 계산:
   \[
   	ext{비율} = rac{1082}{17083} pprox 0.0634
   \]
2. 2023년 예측:
   \[
   	ext{2023년 배우자 부정} = 12397 	imes 0.0634 pprox 785
   \]

---

## 요약

이 코드는 **2017년 데이터를 기반으로 사유별 비율을 계산**하고, **미래 연도(2018~2023)의 실제 총 이혼 건수**에 따라 **사유별 이혼 건수**를 예측하여 기존 데이터프레임에 추가합니다. 

이 방식은 간단하고 빠르게 예측이 가능하지만, 2017년의 비율이 앞으로도 유지된다는 가정을 기반으로 한다는 점에 유의해야 합니다.
