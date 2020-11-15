# Choleor
<strong>기존 안무 영상을 다양한 조합으로 교차편집 할 수 있는 기능을 제공해 안부 창작을 돕는 서비스(졸업 프로젝트)</strong><br>
3인 팀으로 1년간 수행한 프로젝트로 저는 포즈수치화 / 영상 교차편집 / 프론트엔드 / DB 구축을 담당했습니다.

## 1. 포즈 수치화
PoseNumericalization.ipynb
### 포즈 수치화의 필요성
![스크린샷(18)](https://user-images.githubusercontent.com/50199997/99183057-870aec00-277c-11eb-9519-d56ec22ac4c9.png)<br><br>
서로 다른 두 영상을 부드럽게 이어붙이기위해서는 영상이 전환되는 부분에서 사람의 위치, 크기, 포즈가 유사해야 합니다. 이 중 포즈의 유사도를 측정하기 위해서 포즈를 수치화 하는 방법을 고안해냈습니다.<br>
포즈 수치화를 위해 필요한 각 관절의 좌표 정보는 openpose를 사용해 분석했습니다.

### Idea 1. Labanotation(무용기보법)
![labanotation](https://user-images.githubusercontent.com/50199997/99185037-7feada80-278a-11eb-86dc-dbe78cae0b73.PNG)<br><br>
labanotation은 무용기보법의 일종으로 왼팔, 오른팔, 왼다리, 오른다리 그리고 몸통이 어느 방향으로 움직이는지를 표기하는 방식으로 구성됩니다. 이런 표기법에서 사지의 위치정보를 각각 표현하는 방법을 착안했고, 몸의 방향, 몸의 높이(서있는지, 앉아있는지, 누워있는지), 손발의 높이, 손발의 좌우 위치를 각각 2bit로 표기하고 이것을 쭉 연결해 하나의 int값으로 표기하는 방법을 만들었습니다.

### Idea 2. Karnaugh Map(카르노 맵)
![카르노맵](https://user-images.githubusercontent.com/50199997/99184994-4dd97880-278a-11eb-9571-ab750c91b6ae.PNG)<br><br>
카르노맵은 보통 논리 회로 설계를 단순화 하기 위해 사용되는 표 입니다. 4x4 카르노맵의 각 행과 열에는 인접한 칸과 1bit씩만 차이가 나도록 2bit 숫자가 할당되어있습니다. 따라서 인접한 칸끼리 XOR연산을 수행하면 결과는 1이 나오고 두 칸 사이의 거리가 멀어질수록 XOR 연산의 결과는 커지게 됩니다. 이런 원리를 포즈 수치화 과정에 반영하면 마찬가지로 XOR연산으로 두 포즈 사이에 얼마나 큰 차이가 있는지를 수치화 할 수 있습니다.

### 포즈 수치화의 예시
![스크린샷(19)](https://user-images.githubusercontent.com/50199997/99183984-4662a100-2783-11eb-9b7c-132d996a247a.png)<br><br>
위 이미지는 왼손과 오른발의 위치를 수치화 한 예시입니다.<br>
손의 경우 어깨를 기준으로 상하/좌우 각각 세 구역으로 나누어 손이 위치해 있는 구역의 값을 할당합니다. 위 이미지의 경우에는 좌측 중앙, 즉 1000의 값을 할당할 수 있습니다.<br>
발의 경우 골만을 기준으로 상하/좌우 각각 세 구역으로 나누어 발이 위치해 있는 구역의 값을 할당합니다. 위 이미지의 경우에는 중앙 하단, 즉 0001의 값을 할당할 수 있습니다.<br><br>

이 외에도 이미지에 나온 포즈 전체를 수치화하면 아래와 같습니다.<br>
방향/높이/ 왼손 /오른손/ 왼발 / 오른발<br>
00 / 00 / 1000 / 1000 / 1001 / 0001<br>
(00001000100010010001)<sub>2</sub> = (34961)<sub>10</sub>

### 포즈 수치화의 한계점
1. 포즈의 깊이를 판단할 수 없습니다. 포즈 수치화를 위한 각 관절의 정보는 openpose를 통해 받아오게 됩니다. 그런데 사람의 몸은 3차원 공간에서 움직이는 것과 다르게 openpose로 얻은 정보는 2차원 정보이기 때문에 발을 앞으로 뻗거나 뒤로 뻗은 동작은 구분하는게 불가능합니다.<br>
2. 몸의 방향을 구별하는데 어려움이 있습니다. openpose에서 앞, 뒤를 구별해 좌표를 출력하기 때문에 정면주시/후면주시는 구별할 수 있지만 좌/우측주시의 경우에는 몸의 좌우중 어느쪽이 카메라와 가까운 곳인지 알 수 없기 때문에 구별할 수 없습니다.<br>
3. 몸의 높이의 경우도 일단은 어깨-골반, 골반-((무릎+발)/2) 이 두 거리의 비율을 기준으로 구별하고 있지만 손/발 측정에 어깨/골반이 기준으로 쓰이는 것 만큼 명확한 기준이 되지는 못하기 때문에 정확도가 많이 떨어지는 결과를 보여줍니다.<br>
4. openpose 자체의 기능이 떨어집니다. 여러 사람이 나오는 영상은 두 사람을 합쳐서 한 사람으로 인식하기도 하고, 학습 데이터에 춤과 같은 다이나믹한 포즈가 없었는지 정적인 포즈에 적용했을 때에 비해서 정확도가 떨어지는 것을 확인할 수 있었습니다.<br><br>

이러한 문제점을 해결하기 위해 결국 자동화된 코드를 통해 수집한 데이터를 수작업으로 검수하는 과정을 거쳐야 했습니다. 이런 검수 과정을 조금이라도 줄이기 위해 다양한 코드를 작성해 사용했습니다. (Tools.ipynb)


## 2. 영상 교차편집
VideoEditor.ipynb<br><br>
![edit1](https://user-images.githubusercontent.com/50199997/99198470-172c4e00-27dc-11eb-9a85-bfea7907fcb5.PNG)<br><br>
서로 다른 두 영상을 부드럽게 이어붙이기위해서는 영상이 전환되는 부분에서 사람의 위치, 크기, 포즈가 유사해야 합니다. 포즈는 앞의 과정을 통해 유사한 포즈를 구했으니 이번에는 위치와 크기를 일치시키도록 하겠습니다.<br>
위 이미지 중 왼쪽 이미지가 먼저 올 영상의 마지막 포즈이고, 오른쪽 이미지가 뒤에 올 영상의 첫번째 포즈입니다. 영상 교차편집은 항상 뒤에 올 영상의 크기가 앞의 영상에 맞춰서 수정됩니다.

### Step 1. Resize
가장 먼저 크기를 동일하게 맞춰주도록 합니다. 인물의 크기는 키를 기준으로 판별하는데 위 이미지를 보면 오른쪽 인물의 키가 왼쪽 인물의 키보다 큰 것을 볼 수 있습니다. 이 두 키의 비율을 이미지의 높이와 폭에 곱해줘서 인물의 크기를 일치시킬 수 있습니다.<br>

<pre>
<code>
//y_max = [앞의 y_max, 뒤의 y_max]
//y_min = [앞의 y_min, 뒤의 y_min]
resize_ratio = (y_max[0] - y_min[0]) / (y_max[1] - y_min[1])
resized_x = (int)(1920 * resize_ratio)
resized_y = (int)(1080 * resize_ratio)
resize = cv2.resize(image, dsize=(resized_xC, resized_yC), interpolation=cv2.INTER_AREA)
</code>
</pre>

<strong>Result</strong><br>
![edit2](https://user-images.githubusercontent.com/50199997/99198660-658e1c80-27dd-11eb-91c1-101dd362a01e.PNG)

### Step 2. Crop & Margin
두번째로는 인물의 위치를 맞춰주도록 합니다. DB에는 인물의 y_min, y_max, x_mean 값이 들어있습니다. 두 영상에서 이 값들이 동일한 위치에 있도록 이미지를 자르고, 붙여주는 작업을 합니다.<br>
[이미지의 상단 - y_min], [하단 - y_min], [우측 - x_mean], [좌측 - x_mean] 사이의 거리를 두 이미지 각각에서 구하고 앞의 이미지 값이 더 크면 crop, 앞의 이미지 값이 더 작으면 margin을 뒤의 이미지에 적용해서 인물의 위치를 조절합니다.

<pre>
<code>
// C = crop, 이미지의 상하좌우를 얼마나 자를지
// M = margin, 이미지의 상하좌우에 얼마나 이미지를 붙일지
Cx1 = Cy1 = 0
Cx2 = resized_x
Cy2 = resized_y
Mx1 = Mx2 = My1 = My2 = 0
rx_mean = (int)(x_mean[1] * resize_ratio)
ry_max = (int)(y_max[1] * resize_ratio)
ry_min = (int)(y_min[1] * resize_ratio)

// 인물 왼쪽 사이즈 조절
if x_mean[0] > rx_mean: // 뒤의 이미지가 앞의 이미지보다 왼쪽에 있으면
  Mx1 = x_mean[0] - rx_mean
else:
  Cx1 = rx_mean - x_mean[0]

// 인물 오른쪽 사이즈 조절
if 1920 - x_mean[0] > resized_x - rx_mean: // 뒤의 이미지가 앞의 이미지보다 오른쪽에 있으면
  Mx2 = (1920 - x_mean[0]) - (resized_x - rx_mean)
else:
  Cx2 = resized_x - ((resized_x - rx_mean) - (1920 - x_mean[0]))

// 인물 상단 사이즈 조절
if y_max[0] > ry_max: // 뒤의 이미지가 앞의 이미지보다 위에 있으면
  My1 = y_max[0] - ry_max
else:
  Cy1 = ry_max - y_max[0]

// 인물 하단 사이즈 조절
if 1080 - y_min[0] > resized_y - ry_min: // 뒤의 이미지가 앞의 이미지보다 아래에 있으면
  My2 = (1080 - y_min[0]) - (resized_y - ry_min)
else:
  Cy2 = resized_y - ((resized_y - ry_min) - (1080 - y_min[0]))
  
crop = resize[Cy1C:Cy2C - Cy1C, Cx1C:Cx2C - Cx1C]
margin = cv2.copyMakeBorder(crop, My1C, My2C, Mx1C, Mx2C, cv2.BORDER_CONSTANT, value=[0, 0, 0]) // margin은 검은색으로 둘러준다.
final = cv2.resize(margin, dsize=(1920, 1080), interpolation=cv2.INTER_AREA) // 계산 결과가 1920x1080에서 조금 벗어날 경우를 대비

</code>
</pre>

<strong>Crop Result(노란 선)</strong><br>
![edit3](https://user-images.githubusercontent.com/50199997/99199085-fa921500-27df-11eb-8e44-ff217efa3ad5.PNG)<br>

<strong>Margin Result(초록 선)</strong><br>
![edit4](https://user-images.githubusercontent.com/50199997/99199088-fbc34200-27df-11eb-9b48-969b8aa2a860.PNG)

### Step 3. Restore
앞의 단계들에서는 인물의 위치를 맞춰주기 위해서 영상의 크기, 위치에 조작을 가했습니다. 하지만 만약 연달아 나오는 영상에서 인물들이 자꾸 오른쪽으로 가거나 뒤로 가면 어떻게 될까요? 결국에는 영상에 가하는 조작에 중첩되어서 인물이 아주 작아지거나 영상 밖으로 나가버릴지도 모릅니다. 이를 막아주기 위해 resize_ratio, M, C값은 영상이 재생되면서 다시 조작 이전의 값으로 돌아가야 합니다.

<pre>
<code>
change_value = 0
count = 0
curr = 0
while (vidcap.isOpened()):
  ret, image = vidcap.read() // 영상의 이미지를 하나씩 불러옴
  curr += 1 // 이미지 번호
  if curr > 20 and curr < (frame_num - 20): // 영상의 첫 20장과 마지막 20장을 제외하고(영상이 바뀌는 부분)
    count += 1
    change_value = count / (frame_num - 40) // count가 증가함에 따라 0에서 1까지 값이 조금씩 증가한다.

  // resize, crop, margin 값을 조금씩 없던걸로 돌린다.
  resized_xC = resized_x + (int)((1920 - resized_x) * change_value)
  resized_yC = resized_y + (int)((1080 - resized_y) * change_value)

  resize = cv2.resize(image, dsize=(resized_xC, resized_yC), interpolation=cv2.INTER_AREA)

  Cx1C = Cx1 + (int)((0 - Cx1) * change_value)
  Cx2C = Cx2 + (int)((1920 - Cx2) * change_value)
  Cy1C = Cy1 + (int)((0 - Cy1) * change_value)
  Cy2C = Cy2 + (int)((1080 - Cy2) * change_value)

  Mx1C = Mx1 + (int)((0 - Mx1) * change_value)
  Mx2C = Mx2 + (int)((0 - Mx2) * change_value)
  My1C = My1 + (int)((0 - My1) * change_value)
  My2C = My2 + (int)((0 - My2) * change_value)
</code>
</pre>

### Step 4. Fade in & out
이제 영상을 이어붙이기만 하면 됩니다. 하지만 갑작스럽게 영상이 휙 바뀐다면 아무리 포즈나 크기 위치를 최대한 맞춰줬더라도 완벽히 일치하는게 아니기 때문에 사용자가 영상의 변화를 따라가기 힘들 것입니다. 따라서 영상이 변화는 과정을 부드럽게 만들어주기 위해 fade in & fade out을 적용하도록 합니다.

<pre>
<code>
if count < end - 20: // 앞의 영상만 나오는 부분
  ret1, image1 = vidcap1.read()
  
  if count <= 20: // 앞의 20장은 이미 교차하는 부분에서 처리되었으니 뛰어넘음
    continue
    
  out.write(image1)

elif count < end: // 두 영상이 교차하는 부분
  ret1, image1 = vidcap1.read()
  ret2, image2 = vidcap2.read()

  //image1은 count가 커질수록 작아지고, image2은 count가 작아질수록 커진다.
  blending = cv2.addWeighted(image1, ((end - count) / 20), image2, 1 - ((end - count) / 20), 0)
  out.write(blending)

else: // 뒤의 영상만 나오는 부분
  ret2, image2 = vidcap2.read()
  out.write(image2)
</code>
</pre>

### Result
지금까지 예시로 사용한 두 영상을 실제 코드를 사용해 교차편집한 결과입니다.<br>
https://www.youtube.com/watch?v=lhp3c4nscZc

### 영상 & 이미지 출처
https://www.youtube.com/watch?v=QDqlB8M25DQ<br>
https://www.youtube.com/watch?v=Pg-SswwMc9w

