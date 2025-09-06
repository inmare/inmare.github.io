---
title: Unity Rendering 2 셰이더 기초
categories: [Unity]
tags: [Unity, graphics]
---

- [들어가면서](#들어가면서)

Unity 2022 LTS 버전을 기준으로 해서 프로젝트를 진행했다.

# 프로젝트 설정 방법

GameObject > 3D Object > Sphere를 통해서 구체를 하나 생성한다.  
Window > Rendering > Lighting에 들어가서 빛과 관련된 요소들을 없애준다.

- Environment > Skybox Material을 None으로 설정한다.
- Ambient Color를 단색으로 설정. 이는 Skybox를 없애면 자동으로 설정된다.
- Precomputed Realtime GI를 체크 해제 하라고 했는데 설정에는 없어서 스킵했다

Camera의 Inspector에서 Background color도 Skybox를 없애면 자동으로 단색으로 설정된다.  
기본으로 생성된 Directional Light도 삭제한다.

위와 같은 작업을 하면 Scene을 실행했을 때 검정색 구체가 파란색 배경화면에 나나게 된다.
![2025-09-06T180533](2025-09-06T180533.png)

# 렌더링의 과정

Unity에서 렌더링이 일어나는 과정은 대략적으로 아래와 같다.

- Camera의 view frustum내부에 object가 있어야 한다.
- object가 mesh renderer component를 가지고 있어야 한다.
- 행렬을 통해서 위치 이동, 크기, 회전 등이 적용된다.
- 그리고 GPU가 물체를 렌더링 하는데, 이때 Shader를 통해서 어떻게 렌더링 할 지를 결정한다.

# Shader

Assets > Create > Shader > Unlit shader를 통해서 새로운 셰이더를 생성한다.  
Material을 하나 생성하고 Shader를 방금 생성한 셰이더로 설정한다.  
생성한 Material을 구체에 적용한다.

Shader를 더블클릭해서 열면 코드가 나오고, 해당 파일에 아래의 코드를 작성한다.

```c
Shader "Custom/My First Shader" {}
```

```c
Shader "Custom/My First Shader"
{
    // 여러 개의 sub shader를 같이 사용해서 플랫폼마다 다른 shader를 제공할 수 있음
    SubShader
    {
        // Sub shader는 물체가 실질적으로 렌더링 되는 1개 이상의 pass를 무조건 가져야 함
        Pass
        {
            // Shader program의 시작
            CGPROGRAM

            // 컴파일러의 동작 제어를 위한 pragma
            #pragma vertex MyVertexProgram
            #pragma fragment MyFragmentProgram

            // Shader의 작동을 위해서는 vertex와 fragment 프로그램이 필요함
            // Vertex: vertex데이터와 mesh를 렌더링 하기 위해서 필요함
            // Fragment: Mesh의 삼각형을 색칠하기 위해서 필요함
            void MyVertexProgram()
            {

            }

            void MyFragmentProgram()
            {

            }
            ENDCG
        }
    }
}
```

## 컴파일된 셰이더 확인하기

Unity는 플랫폼 별로 셰이더를 컴파일 하고, 해당 결과를 Shader의 Inspector에서 확인할 수 있다.
![2025-09-06T184257](2025-09-06T184257.png)

# 기본적인 셰이더

```c
// Pass 내부

// 렌더링에 필요한 여러 변수, 값들을 포함한 파일 추가
// 크로스 플랫폼 렌더링에 필요한 파일도 포함되어 있음
#include "UnityCG.cginc"

// Vertex program은 float4형태의 vertex좌표를 최종적으로 반환해야 함
// 반환 위치를 SV_POSITION으로 특정함
float4 MyVertexProgram() : SV_POSITION
{
    return 0;
}

// Fragment program도 float4 형태의 색상 좌표를 SV_Target으로 반환해야 됨
float4 MyFragmentProgram() : SV_Target
{
    return 0;
}
```

아래의 코드는 Material에서 설정한 색상을 받아와서 구체에 적용하는 코드이다.

```c
Shader "Custom/My First Shader"
{
    // 변수를 받을 수 있는 부분
    // Material에서 괄호 안의 항목을 선택할 수 있게 됨
    Properties
    {
        _Tint ("Tint", Color) = (1, 1, 1, 1)
    }

    SubShader
    {
        Pass
        {
            CGPROGRAM

            #pragma vertex MyVertexProgram
            #pragma fragment MyFragmentProgram

            // 렌더링에 필요한 여러 변수, 값들을 포함한 파일 추가
            // 크로스 플랫폼 렌더링에 필요한 파일도 포함되어 있음
            #include "UnityCG.cginc"

            // 받아온 값을 저장하는 변수
            float4 _Tint;

            // Shader의 작동을 위해서는 vertex와 fragment 프로그램이 필요함
            // Vertex: vertex데이터와 mesh를 렌더링 하기 위해서 필요함
            // Fragment: Mesh의 삼각형을 색칠하기 위해서 필요함

            // Vertex program은 float4형태의 vertex좌표를 최종적으로 반환해야 함
            // 반환 위치를 SV_POSITION으로 특정함
            // 현재 위치를 가져와서 해당 위치를 반환함
            float4 MyVertexProgram(float4 position: POSITION) : SV_POSITION
            {
                // return mul(UNITY_MATRIX_MVP, position);
                return UnityObjectToClipPos(position);
            }

            // Fragment program도 float4 형태의 색상 좌표를 SV_Target으로 반환해야 됨
            // Vertex Shader의 값을 받아와서 위치를 그대로 반환함
            float4 MyFragmentProgram(float4 position: SV_POSITION) : SV_Target
            {
                return _Tint;
            }
            ENDCG
        }
    }
}
```

# Vertex to Fragment

fragment program에서 gpu는 삼각형을 rasterize하고, 삼각형을 이루는 모든 픽셀은 fragment program을 통해서 보간된 데이터를 받게 된다.  
아래의 코드에서는 localPosition을 받고, 이를 fragment program에 넘겨준다.

```c
float4 MyVertexProgram(
    float4 position: POSITION,
    // localPosition을 TEXCOORD0이라고 하는 위치에 저장함
    out float3 localPosition: TEXCOORD0
    ) : SV_POSITION
{
    // localPosition에 현재 position의 xyz값을 전달함
    localPosition = position.xyz;
    return UnityObjectToClipPos(position);
}

float4 MyFragmentProgram(
    float4 position: SV_POSITION,
    float3 localPosition: TEXCOORD0
    ) : SV_Target
{
    // 전달받은 localPosition값을 색상값으로 전달함
    return float4(localPosition, 1);
}
```

struct를 사용해서 더 단순화한 코드이다.

```c
// 구조체를 반환해서 조금 더 코드를 간략화 시킴
struct Interpolators
{
    float4 position: SV_POSITION;
    float3 localPosition: TEXCOORD0;
};

Interpolators MyVertexProgram(float4 position: POSITION)
{
    Interpolators i;
    i.localPosition = position.xyz;
    i.position = UnityObjectToClipPos(position);
    return i;
}

float4 MyFragmentProgram(Interpolators i) : SV_Target
{
    // 0.5를 더해서 음수 값이 0으로 맞춰지는 걸 방지
    // 그리고 _Tint를 더해서 색상 값을 더해줌
    return float4(i.localPosition + 0.5, 1) * _Tint;
}
```

# UV

```c
// Properties에 MainTex를 추가함
_Tint ("Tint", Color) = (1, 1, 1, 1)
_MainTex ("Texture", 2D) = "White" {}

// Pass 내부
struct Interpolators
{
    float4 position: SV_POSITION;
    float2 uv: TEXCOORD0;
};

// 전달 받는 인수가 2개 이상이라서 struct를 사용함
struct VertexData
{
    float4 position: POSITION;
    float2 uv: TEXCOORD0;
};

float4 _Tint;
sampler2D _MainTex;

Interpolators MyVertexProgram(VertexData v)
{
    Interpolators i;
    i.position = UnityObjectToClipPos(v.position);
    i.uv = v.uv;
    return i;
}

float4 MyFragmentProgram(Interpolators i) : SV_Target
{
    return tex2D(_MainTex, i.uv) * _Tint;
}
```

# WarpMode

텍스처의 설정에서는 다양한 설정을 할 수 있는데, 그 중 하나가 Warp Mode이다.
만약 1, 1을 넘ㅁ는 좌표가 나왔을 때 어떻게 텍스처가 대응 할 것인지 선택할 수 있는데, 만약 Clamp로 선택한다면 기본값인 Repeat와 다르게 1을 넘는 좌표는 무조건 1로 고정된다.

# Mipmap과 Filtering

**Filtering**은 만약 텍스처를 확대한다면 어떻게 표시할지 설정하는 것인데, 기본값은 Bilinear이다.  
이를 어떻게 설정하느냐에 따라서 텍스처가 확대 되었을때 존재하지 않는 픽셀을 어떻게 보간할지 결정된다.

반대로 **Mipmap**은 텍스처를 축소할 때 어떻게 표시할지 설정하는 것인데, 기본값은 Trilinear이다.  
텍스처를 축소하면 한번에 여러 개의 텍스처를 한 픽셀에 표현해야 되는데, 이를 해결하기 위해서 여러 크기의 텍스처를 만들어서 축소된 크기에 맞는 텍스처를 사용한다.

그리고 Advanced setting에서 Fadeout to Gray을 설정하면 Mipmap을 사용한 부분은 회색으로 표시된다.  
이때 Filtering모드를 Bilinear로 설정하면 Mipmap을 사용한 부분의 경계가 날카롭게 표시되고, 이를 해결하려면 두 텍스처 사이의 보간을 사용하는 Trilinear로 설정하면 된다.  
또한 Edit > Project Settings > Quality의 Rendering에서 Anisotropic Filtering을 설정하면 텍스처가 기울어진 각도에 따라서도 보간을 적용할 수 있다.  
이 경우 가장 크게 기울어진 각도에 맞춰서 보간이 적용된다.  
얼마나 깊게 Anisotropic Filtering을 적용할지 Aniso Level로 설정할 수 있다.
