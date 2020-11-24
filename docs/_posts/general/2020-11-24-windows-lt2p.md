---
layout: post
title: Windows 10 Home 에서 LT2P (VPN) 연결하기
author: kade
categories: general
comments: true
---

안녕하세요 두다지의 인턴 kade입니다 :)  
오늘의 포스트 주제는 윈도우10 Home 환경에서 LT2P VPN 접속 방법입니다.

최근 코로나 확진자가 증가함에 따라 재택근무를 하게 되었습니다. 
Mac OS 의 경우 비교적 손쉽게 LT2P 연결을 통해 회사의 서버에 접속할 수 있지만
윈도우10, 특히 Home 버전의 경우 VPN 연결방법이 꽤나 복잡하여 정리해보았습니다.

* 간단한 순서는 
1. 로컬 보안 정책 활성화
2. 로컬 보안 정책 수정
3. 레지스트리 수정
4. VPN 접속 시도

* 레지스트리 편집과, .bat 파일을 바로 실행할 수 있도록 파일 첨부합니다 

    [windows10-lt2p.zip](https://github.com/dudaji/blog/files/5581972/windows10-lt2p.zip)

    압축 파일을 풀어주시면 `.bat` 파일한개와 `.reg` 파일 두개가 있습니다.  
    `.bat` 파일은 관리자권한 실행시 로컬 보안 정책을 활성화합니다.  
    `.reg` 파일 두가지는 3번 내용으로, 실행 시 레지스트리를 편집해줍니다.  


### 1. window10 home 에 로컬 보안 정책 (secpol.msc) 활성화

윈도우10 home 에는 로컬 보안 정책이 기본적으로 지원되지 않습니다.  
따라서 첨부한 bat 파일을 이용하여 따로 설치해주시거나   
하단 내용에 따라 직접 배치파일을 만드셔서 관리자권한으로 실행하셔도 됩니다.

* 메모장을 열어 하단 코드를 붙여넣고 bat 파일로 저장합니다. ( Windows 배치 파일 )

```bash
@echo off 
pushd "%~dp0"
 
dir /b %SystemRoot%\\servicing\\Packages\\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt 
dir /b %SystemRoot%\\servicing\\Packages\\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt
 
for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\\servicing\\Packages\\%%i" 
pause
```

* 다른이름으로 저장 → 모든 파일 → .bat (확장자 명을 .bat으로 저장합니다. )

![makebat](/assets/windows10-lt2p/makebat.png)

![makebat2](/assets/windows10-lt2p/makebat2.png)

* 만든 bat 파일 관리자권한으로 실행합니다.

![run_admin](/assets/windows10-lt2p/run_admin.png)

* 아래와 같이 작업이 완료됩니다.

![bat-reuslt](/assets/windows10-lt2p/bat_result.png)


### 2. 로컬 보안 정책 수정


* 이제 윈도우 검색창에 로컬 , 또는 secpol 검색하면 로컬 보안 정책을 실행할 수 있습니다.

![search_secpol](/assets/windows10-lt2p/search_secpol.png)



* 로컬 정책 → 보안 옵션 → 네트워크 보안 (우클릭) -> 속성 -> LAN MANAGER 인증 수준   
→ LM 및 NTLM 보내기 선택 → 적용



![set_secpol1](/assets/windows10-lt2p/set_secpol1.png)




* 네트워크 보안 : NTLM SSP 기반 (보안 RPC 포함) 서버에 대한 최소 세션 보안  
 → 속성 → NTMLv2 세션 보안 필요 박스에 체크가 되어 있는지 확인합니다.


* 네트워크 보안 : NTLM SSP 기반 (보안 RPC 포함) 클라이언트에 대한 최소 세션 보안  
 → 속성 → NTMLv2 세션 보안 필요에 체크해주고 , 128비트 암호화 필요 체크 해제해줍니다.



![set_secpol2](/assets/windows10-lt2p/set_secpol2.png)




### 3. 레지스트리 편집


윈도우 검색 → regedit → 하단 이미지와 코드를 보고 경로 따라 수정또는 추가해주시면 됩니다.

또는, 파일을 첨부할테니 .reg 파일을 실행시켜주세요


![regedit1](/assets/windows10-lt2p/regedit1.png)

![regedit2](/assets/windows10-lt2p/regedit2.png)


* 첨부파일 (edit_policyAgent_registry.reg) 은 하단과 같습니다.

```bash
[HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\PolicyAgent]
"AssumeUDPEncapsulationContextOnSendRule"=dword:00000002
```

* 첨부파일 (edit_lsa_registry.reg) 은 하단과 같습니다.

```bash
[HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\Lsa]
"LmCompatibilityLevel"=dword:00000003
```



### 4. VPN 연결 확인

( iptime 에서 기존에 설정이 되어 있어야 합니다. )

![vpn_settings_window](/assets/windows10-lt2p/vpn_settings_window.png)

![vpn_settings_iptime](/assets/windows10-lt2p/vpn_settings_iptime.png)




### 5. 마치며

윈도우에서 VPN L2TP 연결을 여러번 시도하다가 포기하였는데 이 방법을 통해 해결하였습니다.

첨부한 파일의 경우 레지스트리를 건드리기 때문에 위험할 수 있습니다. 
다운받아서 마우스 우클릭 편집을 눌러서 첨부한 코드와 동일한지 확인해보시고, 현재 컴퓨터에 이 레지스트리를 적용했을 때 문제가 안생기는지 잘 찾아보시고 적용하시길 바랍니다.


### 참고자료

https://blog.naver.com/linuxni/221774779895 

[Connect L2TP VPN to Windows 10](https://www.youtube.com/watch?v=OIPbhb-j0a4) 



​           