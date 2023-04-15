러스트 레거시 핵 개발 프로젝트
==================

* 러스트 레거시
  * ![1](https://user-images.githubusercontent.com/49302859/232206551-d8127566-be3d-4ae4-9098-4629dacf36f6.PNG)
  * 2014년도에 발매된 rust 라는 게임.
    * 알파에서 베타, 그리고 현재는 러스트로 유명함.
    * 러스트 레거시라는 이름도 원래는 러스트 알파로 유명했는데.. 시대가 흐르면서 러스트 레거시로 바뀜
    * ![2](https://user-images.githubusercontent.com/49302859/232206718-d91c9df2-80fb-4f76-aa2c-20e23625f1d2.PNG)
----------------------------------------    
* 개발 이유
  * 2014 ~ 2016년까지 재밌게했던 게임이고, 실제 서비스 중단되어 과거에 배포된 서버팩으로 유저끼리만 알음알음 하는 게임이라..
 ----------------------------------------    
* 개발 환경
  * 윈도우 10
----------------------------------------    
* 개발 도구
  * CheatEngine, x64Dbg, dnSpy, die, HxD, Enigma Virtual Box, EnigmaVBUnpacker
  * 실직적으로 dnspy, die, HxD, EnigmaVBUnpacker 를 중점적으로 사용
----------------------------------------    
* 클라이언트
  * Rust.Client.v25.02.2014
----------------------------------------    
* 서버
  * 구하지 못했다. 너무 옛날 게임이라 자료가 남아있지 않음.
  * 해외 서버 몇 개를 간신히 찾아서 거기 접속해서 핵 개발 착수
----------------------------------------      
* 초기에 많은 뻘짓을 했다.
  * 과거의 게임 대부분이 유니티 il2Cpp 를 지원하지 않기 때문에 mono 방식
  * mono 방식의 경우 Assembly CSharp.dll 만 디스어셈블하면 손쉽게 해킹 가능
  * 러스트의 경우, exe 와 dll 이 패킹되어 있음
    * die 로 확인
    * EnigmaVBUnpacker 로 언패킹
----------------------------------------    
* 언패킹 이후에 손쉽게 해킹이 가능할 줄 알았는데...
  * Rust 클라이언트 동봉된 asset 을 하나를 HxD 로 열어보면 해당 유니티 버전을 알 수 있음
  * 2014 년도에 발매했기 때문에 Unity 4.3.4 를 사용함.
  * 아무리 코드를 열어봤다고는 하지만 지원되는 기능도 너무 부족하고.. 자료도 부족한 상태였음
----------------------------------------   
* 방식
  * ESP 출력 (사각형 박스)
    * 외부 프로그램을 이용해서 게임 윈도우 Screen 에 덧댈 것이 아니라면 직접 게임 내에서 출력해줘야한다.
    * 외부 프로그램으로 처리 시, 치트 엔진을 이용한 메모리 포인터 및 frida 의 후킹을 사용해야함. ( 후킹의 경우 직접 구현해도됨.)
    * 결론적으로 dnSpy 로 코드 전체를 볼 수 있는데, 굳이 외부에서 진행할 필요가 없다고 판단.
---------------------------------------- 
* 코드 구현
  * EasyCheat class 구현
    * 인게임 채팅 및 콘솔창 출력 기능
  * 플레이어 및 동물 오브젝트 위치 파악
    * Character class > origin
      * 해당 프로퍼티에서 Update 돌면서 유저 및 동물 오브젝트의 위치 정보 리프레쉬해주는 것 파악
      * 프로퍼티 내에 ESP 코드 삽입
  * EspObject class 구현
    * ESP 대상 주변에 사각형 박스를 그려주는 역할
  * 사각형 박스 랜더링을 위한 카메라 생성
    * ESP 핵은 모든 오브젝트를 뚫고 ESP 대상 주변에 사각형 박스를 출력해야함.
    * 그냥 그리게 되면 나무나 산 같은 기타 오브젝트에 가려서 보이지 않게된다.
    * 카메라 생성 코드
    ```C#
    public static void CreateCamera()
	   {
		   EspObject.uiCamera = new GameObject
		   {
			   transform = 
			   {
				   position = new Vector3(500f, 500f, -10f)
			   }
		   }.AddComponent<Camera>();
		   EspObject.uiCamera.clearFlags = CameraClearFlags.Depth;
		   EspObject.uiCamera.cullingMask = 1024;
		   EspObject.uiCamera.orthographic = true;
		   EspObject.uiCamera.orthographicSize = 5f;
		   EspObject.uiCamera.nearClipPlane = 0.3f;
		   EspObject.uiCamera.farClipPlane = 1000f;
		   EspObject.uiCamera.rect = new Rect(0f, 0f, 1f, 1f);
		   EspObject.uiCamera.depth = 0f;
	   }
    ```
    * 카메라 생성 후, LineRenderer 를 통해 생성된 오브젝트의 레이어를 cullingMask 와 동일하게 맞춰주면 완료.
---------------------------------------- 

* 동영상 유튜브에 올리긴 좀 그래서.. 간단하게 gif 로 만들어서 올림. 동영상 편집 프로그램이 없어서.. 반디컷으로 
![bandicam 2023-04-15 19-54-55-719 (2)](https://user-images.githubusercontent.com/49302859/232217668-0345f3c2-39ff-4fce-8ea4-d16143bef0e9.gif)
