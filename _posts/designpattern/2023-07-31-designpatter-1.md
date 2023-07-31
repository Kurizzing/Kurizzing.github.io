---
layout: single
title: "4.싱글턴으로 게임 매니저 구현하기"
categories: designpattern
toc: true
toc_sticky: true
---

# 4.2 싱글턴 패턴 이해하기

![Singleton](https://github.com/Kurizzing/Kurizzing.github.io/assets/48445259/f38e79cc-7510-4488-9e63-60788b141d94)

- 유일성 보장: 런타임 동안 메모리에 오직 하나의 인스턴스만 존재

## 4.2.1 싱글턴 패턴의 장단점

- 장점
    - 전역 접근 가능: 리소스나 서비스의 전역 접근점 제공
    - 동시성 제어: 공유 자원에 동시 접근을 제한하고자 사용 가능
- 단점
    - 유닛 테스트: 유닛 단위의 테스트가 어려워짐, 싱글턴 오브젝트가 다른 싱글턴에 종속될 수 있음
    - 잘못된 습관: 싱글턴으로 모든것에 쉽게 접근하기 때문에 정교한 코드 작성에 귀찮음을 느낌
    

# 4.3 게임 매니저 디자인하기

- 게임 세션을 관리하는 단일 관리자
- 백엔드 서비스와의 통신, 전역 설정 초기화, 로깅, 플레이어의 진행 상황 저장 등
- 게임의 전체 수명 동안 살아있어야 함

# 4.4 게임 매니저 구현하기

```csharp
public class Singleton<T>: MonoBehaviour where T: Component
{
    private static T _instance;

    public static T Instance
    {
        get
        {
            if(_instance == null)
            {
                _instance = FindObjectOfType<T>();

                if(_instance == null)
                {
                    GameObject obj = new GameObject();
                    obj.name = typeof(T).Name;
                    _instance = obj.AddComponent<T>();
                }
            }
            return _instance;
        }
    }

    public virtual void Awake()
    {
        if(_instance == null)
        {
            _instance = this as T;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }
}
```