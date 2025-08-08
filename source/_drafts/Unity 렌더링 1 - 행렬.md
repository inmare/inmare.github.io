---
title: Unity 렌더링 1 - 행렬
tags:
---

- [3D 변환 행렬](#3d-변환-행렬)
- [Orthogrpahic 카메라](#orthogrpahic-카메라)
- [Perspective 카메라](#perspective-카메라)
  - [Focal length](#focal-length)

# 3D 변환 행렬

기존의 3\*3 행렬은 물체의 회전을 표현할 수 있다.  
여기에서 물체의 위치를 표현하려면 offset를 추가해야 하는데, 그러기 위해서는 아래처럼 열을 하나 더 추가해 표현하는 방법이 있다.

$$
\begin{bmatrix}
1 & 0 & 0 & a \\
0 & 1 & 0 & b \\
0 & 0 & 1 & c \\
\end{bmatrix}
\begin{bmatrix}
x \\ y \\ z
\end{bmatrix}=
\begin{bmatrix}
x + a \\
y + b \\
z + c
\end{bmatrix}
$$

여기에서 그치지 않고 행을 하나 더 추가하고, $w$값(4번째 값)으로 1을 넣어서 4\*4 행렬로 표현한다.

$$
\begin{bmatrix}
1 & 0 & 0 & a \\
0 & 1 & 0 & b \\
0 & 0 & 1 & c \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
\begin{bmatrix}
x \\ y \\ z \\ 1
\end{bmatrix}
$$

$w$의 값은 1이어야 하는데, 그 이유는 행렬 연산을 했을 때 offset값(a, b, c)위치의 값에 곱해져서 이동 연산의 결과 값이 반영되게 하기 때문이다.

그리고 w의 값은 0이 아니라면 다른 상수여도 상관없는데, 결과적으로는 $x, y, z$의 $w$배한 값이 결과값이 되어 기존의 값을 바꾸지 않기 때문이다.  
다만 결과값에 변화를 주지 않기 위해 1로 설정하는 것 뿐이다.

# Orthogrpahic 카메라

Orthographic 카메라는 3D 공간을 2D 평면으로 투영할 때, 원근 효과 없이 평행하게 투영하는 방식이다.

![Orthographic projection](2025-08-08T180913.png)

이를 행렬로 구현하려면 한 차원의 값을 0으로 설정하면 된다.  
예를 들어 아래의 행렬에서는 z의 값을 0으로 설정해 xy평면에 물체를 투영한다.  
이 행렬을 기존의 3d 변환 행렬에 곱하면 해당 결과를 확인 할 수 있다.

$$
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$

# Perspective 카메라

위의 방식은 현실처럼 멀리있는 물체를 더 작게 보주지 못하기 때문에 Perspective 카메라를 사용한다.

![2025-08-08T181603](2025-08-08T181603.png)

이를 구현하려면 위의 행렬에서 w의 값을 z가 되도록 행렬을 변경하면 된다.

$$
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
\begin{bmatrix}
x \\ y \\ z \\ 1
\end{bmatrix}=
\begin{bmatrix}
x \\ y \\ 0 \\ z
\end{bmatrix}
$$

위처럼 결과로 나온 벡터는 이전에 설명한 것처럼 $z$를 나누면 $w$값이 1인 벡터로 바꿀 수 있다.  
그렇게 되면 $x, y$값이 $z$값에 비례해서 작아지게 된다.  
이 행렬을 기존의 3d 변환 행렬에 곱하면 Perspective 카메라의 효과를 확인 할 수 있다.

$$
\begin{bmatrix}
x \\ y \\ 0 \\ z
\end{bmatrix} \div z =
\begin{bmatrix}
\frac{x}{z} \\ \frac{y}{z} \\ 0 \\ 1
\end{bmatrix}
$$

## Focal length

그리고 projection을 하는 평면이 어디에 있느냐에 따라서 최종적으로 표시되는 크기도 조절하는데, 이를 focal length라고 한다.  
이 값을 바꾸면 화면에 표시되는 물체의 크기가 달라진다.  
이를 구현하려면 행렬에서 $x, y$의 값을 원하는 focal length의 값으로 바꾸면 된다.

$$
\begin{bmatrix}
fl & 0 & 0 & 0 \\
0 & fl & 0 & 0 \\
0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
\begin{bmatrix}
x \\ y \\ z \\ 1
\end{bmatrix}=
\begin{bmatrix}
xfl \\ yfl \\ 0 \\ z
\end{bmatrix}=
\begin{bmatrix}
\frac{xfl}{z} \\ \frac{yfl}{z} \\ 0 \\ 1
\end{bmatrix}
$$
