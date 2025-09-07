---
title: DirectX11 튜토리얼 2 프로젝트 세팅
categories: [Graphics]
tags: [graphics, DirectX]
---

DirectX를 렌더링 하기 위해서는 이를 창에 표현하기 위한 framework가 필요하다.  
여기에서는 최소한의 기능만을 구현한 framework를 만든다.

대략적인 framework의 구조는 아래와 같다.

```
WinMain
    - SystemClass
        - InputClass
        - ApplicationClass
```

- `WinMain`: `SystemClass`를 포함한 전체 application이 돌아가는 곳
- `InputClass`: 사용자 입력을 다루는 클래스
- `ApplicationClass`: DirectX 그래픽 코드를 다루는 클래스

# WinMain

```cpp
// main.cpp
#include "systemclass.h";

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PSTR pScmdline, int iCmdshow)
{
	SystemClass* System;
	bool result;

	// 새로운 system object 생성
	System = new SystemClass;

	// object를 초기화하고 작동시킴
	result = System->Initialize();
	if (result)
	{
        // Run에서 모든 애니메이션 관련 코드 실행
		System->Run();
	}

	// system을 shutdown하고 release함
	System->Shutdown();
	delete System;
	System = 0;

	return 0;
}
```

# SystemClass 헤더

```cpp
#pragma once

#ifndef _SYSTEMCLASS_H_
#define _SYSTEMCLASS_H_

// 불필요한 Win32 헤더파일 제거용
#define WIN32_LEAN_AND_MEAN

// 윈도우 생성, 파괴 및 다른 함수 사용 용도
#include <windows.h>

// 직접 생성할 클래스들
#include "inputclass.h"
#include "applicationclass.h"

// SystemClass 및 main.cpp에서 사용하는 Initialize, Shutdown, Run등 정의
class SystemClass
{
public:
	SystemClass();
	SystemClass(const SystemClass&);
	~SystemClass();

	bool Initialize();
	void Shutdown();
	void Run();

	// 윈도우 시스템 메세지를 읽을 함수 정의
	LRESULT CALLBACK MessageHandler(HWND, UINT, WPARAM, LPARAM);
private:
	bool Frame();
	void InitializeWindows(int&, int&);
	void ShutdownWindows();
private:
	LPCWSTR m_applicationName;
	HINSTANCE m_hinstance;
	HWND m_hwnd;

	// input, application을 관리할 포인터
	InputClass* m_Input;
	ApplicationClass* m_Application;
};

// 윈도우 시스템 메세지를 클래스 내부로 보낼 함수, 변수
static LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);
static SystemClass* ApplicationHandle = 0;

#endif // _SYSTEMCLASS_H_
```
