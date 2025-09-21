---
title: Unity Rendering 3 텍스처 결합하기
categories: [Unity]
tags: [Unity, graphics]
---

이전에 배웠던 shader와 texture를 이용해서 평면을 더 복잡하게 나타내는 방법을 알아본다.

기존의 Texture는 저장할 수 있는 texel의 수에 한계가 있다. 이를 위해서 texture의 크기를 늘리거나 texel의 크기를 줄이는 방법이 있지만 모두 각각의 한계점이 있다.

이를 확인하기 위해서 이전 프로젝트의 fragment shader를 아래처럼 수정하였다.

```c
float4 MyFragmentProgram(Interpolators i) : SV_Target
{
    float4 color = tex2D(_MainTex, i.uv) * _Tint;
    color = tex2D(_MainTex, i.uv * 10);
    return color;
}
```

기존의 코드를 10배 축소해서 반환하는 코드이고, 이를 통해서 텍스처 이미지는 화질이 좋아지지만, 텍스처의 크기가 작아지면서 이미지가 반복되는 현상이 발생하는 것을 확인할 수 있다.

이때 실제로는 `tex2D`를 두번 사용했지만, 컴파일된 shader 코드를 보면 자동으로 최적화가 되어서 텍스처를 한번만 사용한 것을 확인할 수 있다.

그리고 아래처럼 텍스처를 2번 사용하면 컴파일된 shader에서도 텍스처를 2번 사용한 것을 확인할 수 있다.

```c
float4 MyFragmentProgram(Interpolators i) : SV_Target
{
    float4 color = tex2D(_MainTex, i.uv) * _Tint;
    // 기존 텍스처에 10을 곱하는 방법을 사용해서 더 작은 텍스처를 내부에 표현현
    color *= tex2D(_MainTex, i.uv * 10);
    return color;
}
```

다만 위처럼 텍스처를 계속 곱하면 색이 점점 어두워지는 현상이 발생한다.  
이를 해결하기 위해서 색들이 회색(0.5)로 맞춰진 새로운 텍스처를 도입한다.  
이 텍스처를 사용하고 2를 곱해주면 0.5보다 낮은 색은 밝아지고, 0.5보다 높은 색은 어두워지는 효과가 발생한다.

```c
// Properties 내부
_Tint ("Tint", Color) = (1, 1, 1, 1)
_MainTex ("Texture", 2D) = "White" {}
// 새로운 디테일 텍스처 추가
_DetailTex ("Detail Texture", 2D) = "gray" {}

// Pass 내부
float4 _Tint;
// 두번째 텍스처 용 변수 추가
sampler2D _MainTex, _DetailTex;
float4 _MainTex_ST, _DetailTex_ST;

struct Interpolators
{
    float4 position: SV_POSITION;
    float2 uv: TEXCOORD0;
    // struct에 새로운 변수 추가
    float2 uvDetail: TEXCOORD1;
};

Interpolators MyVertexProgram(VertexData v)
{
    Interpolators i;
    i.position = UnityObjectToClipPos(v.position);
    i.uv = v.uv * _MainTex_ST.xy + _MainTex_ST.zw;
    // 새로운 변수에 디테일 텍스처 좌표 계산산
    i.uvDetail = TRANSFORM_TEX(v.uv, _DetailTex);
    return i;
}

float4 MyFragmentProgram(Interpolators i) : SV_Target
{
    float4 color = tex2D(_MainTex, i.uv) * _Tint;
    // 2를 곱해줘서 0.5를 기준으로 밝아지거나 어두워지게 함
    color *= tex2D(_MainTex, i.uvDetail) * 2;
    return color;
}
```

그리고 멀어질수록 기존의 텍스처가 사라지게 하도록 하기 위해서 아래처럼 minmap을 설정하고 fade out 범위를 조절한다. 그러면 멀어질수록 디테일 텍스처가 사라지면서 자연스럽게 보이게 된다.
![2025-09-22T002115](2025-09-22T002115.png)

![2025-09-22T002155](2025-09-22T002155.png)

다만 만약에 CRT 모니터를 위해 만들어진 시스템인 gamma sapce에서 해당 shader를 사용하면 어두워지는 문제가 발생한다.
Gamma space에서는 밝기가 비선형적으로 표시되는데, shader에서는 이를 linear space라 생각하고 자동으로 보정해주지 않기 때문이다.
이를 해결 하기 위해서 unity에서는 `unity_ColorSpaceDouble`이라는 전역변수를 제공하고, 이를 곱해주면된다.

추가로, detail texture의 효과를 확인하고 싶다면 material의 texture 설정에서 detail texture의 tiling 값을 1이상으로 설정하면 된다.

```c
float4 MyFragmentProgram(Interpolators i) : SV_Target
{
    float4 color = tex2D(_MainTex, i.uv) * _Tint;
    // gamma space에서 어두워지는 문제 해결
    color *= unity_ColorSpaceDouble;
    color *= tex2D(_MainTex, i.uvDetail) * unity_ColorSpaceDouble;
    return color;
}
```

이때 Gamma space 설정은 Edit > Project Settings > Player > Other Settings > Color Space에서 확인할 수 있다.

# Texture Splatting

위처럼 텍스처를 사용하면 모든 곳에서 detail이 동일하게 적용된다는 단점이 있다.  
만약 지역별로 다른 텍스처를 사용하고 싶다면, 어떤 텍스처를 사용할지 나타내는 splat map을 사용할 수 있다.

이때 splat map을 import 할 때는 개별적인 설정이 필요하다.

- 색상을 가져오는게 아니기에 sRGB (Color Texture) 옵션을 끈다.
- Tiling 하지 않을 것이기에 Warp Mode를 Repeat에서 Clamp로 변경한다.
- Linear space에서 사용하기에 프로젝트 설정의 Color Space를 Linear로 변경한다.

> 기존의 튜토리얼과 버전이 달라져서 AI를 참고해서 설정을 바꿨다.

그런 다음 새로운 shader를 생성하고, splat map의 설정을 받아서 두개의 텍스처 중 하나의 텍스처만을 표현하는 코드를 작성한다.

```c
Shader "Custom/Texture Splatting"
{
    Properties
    {
        // Splat map, 텍스처 2개 받기
        _MainTex ("Splat Map", 2D) = "White" {}
        // 연산 부하를 줄이기 위해 scaling 옵션 삭제
        [NoScaleOffset] _Texture1 ("Texture 1", 2D) = "White" {}
        [NoScaleOffset] _Texture2 ("Texture 2", 2D) = "White" {}
    }

    SubShader
    {
        Pass
        {
            CGPROGRAM

            #pragma vertex MyVertexProgram
            #pragma fragment MyFragmentProgram

            #include "UnityCG.cginc"

            sampler2D _MainTex;
            float4 _MainTex_ST;
            // 텍스처 변수 추가
            sampler2D _Texture1, _Texture2;

            struct Interpolators
            {
                float4 position: SV_POSITION;
                float2 uv: TEXCOORD0;
                // splat map용 uv 추가
                float2 uvSplat: TEXCOORD1;
            };

            struct VertexData
            {
                float4 position: POSITION;
                float2 uv: TEXCOORD0;
            };

            Interpolators MyVertexProgram(VertexData v)
            {
                Interpolators i;
                i.position = UnityObjectToClipPos(v.position);
                i.uv = v.uv * _MainTex_ST.xy + _MainTex_ST.zw;
                // vertex data를 splat용 uv에 적용
                i.uvSplat = v.uv;
                return i;
            }

            float4 MyFragmentProgram(Interpolators i) : SV_Target
            {
                // 두 텍스처중 하나만 표시하게 변경
                float4 splat = tex2D(_MainTex, i.uvSplat);
                return (
                    tex2D(_Texture1, i.uv) * splat.r +
                    tex2D(_Texture2, i.uv) * (1 - splat.r)
                    );
            }
            ENDCG
        }
    }
}
```

위에서 rgb splat map을 추가해서 텍스처 맵의 개수를 늘릴수 있다.  
이때 gamma, linear space의 여부에 따라 맵이 섞이는 부분의 색이 다르게 보일 수 있다.  
이는 gamma space의 경우에는 linear space로 변환되었다가 맵이 섞인다음 다시 gamma space로 변환되기 때문이다.

```c
// Properties 내부
// Splat map, 텍스처 2개 받기
_MainTex ("Splat Map", 2D) = "White" {}
// 연산 부하를 줄이기 위해 scaling 옵션 삭제
[NoScaleOffset] _Texture1 ("Texture 1", 2D) = "White" {}
[NoScaleOffset] _Texture2 ("Texture 2", 2D) = "White" {}
[NoScaleOffset] _Texture3 ("Texture 3", 2D) = "White" {}
[NoScaleOffset] _Texture4 ("Texture 4", 2D) = "White" {}

// Pass 내부
sampler2D _MainTex;
float4 _MainTex_ST;
// 텍스처 변수 추가
sampler2D _Texture1, _Texture2, _Texture3, _Texture4;

struct Interpolators
{
    float4 position: SV_POSITION;
    float2 uv: TEXCOORD0;
    // splat map용 uv 추가
    float2 uvSplat: TEXCOORD1;
};

Interpolators MyVertexProgram(VertexData v)
{
    Interpolators i;
    i.position = UnityObjectToClipPos(v.position);
    i.uv = v.uv * _MainTex_ST.xy + _MainTex_ST.zw;
    // vertex data를 splat용 uv에 적용
    i.uvSplat = v.uv;
    return i;
}

float4 MyFragmentProgram(Interpolators i) : SV_Target
{
    // 네가지의 텍스처를 표현
    float4 splat = tex2D(_MainTex, i.uvSplat);
    return (
        tex2D(_Texture1, i.uv) * splat.r +
        tex2D(_Texture2, i.uv) * splat.g +
        tex2D(_Texture3, i.uv) * splat.b +
        tex2D(_Texture4, i.uv) * (1 - splat.r - splat.g - splat.b)
        );
}
```
