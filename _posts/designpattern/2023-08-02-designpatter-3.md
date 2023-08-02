---
layout: single
title: "6.이벤트 버스로 게임 이벤트 관리하기"
categories: designpattern
toc: true
toc_sticky: true
---
# 6.1 이벤트 버스 패턴 이해하기

![EventBus](https://github.com/Kurizzing/Kurizzing.github.io/assets/48445259/be4c8759-0e41-41ee-a17a-99ec2211481a)

- 이벤트 버스: 발행/구독 모델을 사용하여 오브젝트를 연결하는 방법
- 주요 구성 요소
    - Publisher: 이벤트 버스에서 선언한 특정 종류의 이벤트를 구독자에게 publish 할 수 있음
    - Event bus: 구독자와 게시자 사이의 이벤트 전송을 조정하는 역할
    - Subscriber: 이벤트 버스를 통해 특정 이벤트의 구독자로 자신을 등록

## 6.2.1 이벤트 버스 패턴의 장단점

- 장점
    - 분리: 오브젝트는 직접 서로를 참조하는 대신 이벤트로 통신할 수 있다
    - 단순성: 이벤트의 구독 혹은 게시 매커니즘을 추상화하여 단순성을 제공
- 단점
    - 성능: 이벤트 시스템 사용시 약간의 성능 비용 발생
    - 전역: 여기서는 static 메서드와 변수로 이벤트 버스를 구현하여 쉽게 접근할 수 있도록 함. 전역적 접근은 디버깅과 유닛 테스트를 어렵게 해 프로젝트 관리를 어렵게 만듬

## 6.2.2 이벤트 버스를 사용하는 시기

- 빠른 프로토타이핑
    - 분리 상태를 유지하면서 이벤트로 각자 다른 동작을 트리거하는 컴포넌트를 쉽게 만들 수 있음
    - 이벤트 버스로 쉽게 구독자 및 게시자로 오브젝트를 추가하거나 제거 가능
- 프로덕션 코드
    - 복잡한 이벤트 타입이나 구조체를 다루지 않아도 되는 경우에 유용

## 6.2.3 전역 레이스 이벤트 관리하기

- 레이싱 게임 단계: 카운트 다운, 레이스 시작, 레이스 끝, 레이스 일시중지, 레이스 나가기, 레이스 중지
- 구성 요소: HUD, RaceTimer, BikeController, InputRecorder

# 6.3 레이스 이벤트 버스 구현하기

- RaceEventType

```csharp
public enum RaceEventType
{
    COUNTDOWN, START, RESTART, PAUSE, STOP, FINISH, QUIT
}
```

- RaceEventBus

```csharp
public class RaceEventBus
{
    private static readonly IDictionary<RaceEventType, UnityEvent>
    Events = new Dictionary<RaceEventType, UnityEvent>();

    public static void Subscribe(RaceEventType eventType, UnityAction listener)
    {
        UnityEvent thisEvent;

        if (Events.TryGetValue(eventType, out thisEvent))
        {
            thisEvent.AddListener(listener);
        }
        else
        {
            thisEvent = new UnityEvent();
            thisEvent.AddListener(listener);
            Events.Add(eventType, thisEvent);
        }
    }

    public static void Unsubscribe(RaceEventType type, UnityAction listener)
    {
        UnityEvent thisEvent;

        if (Events.TryGetValue(type, out thisEvent))
        {
            thisEvent.RemoveListener(listener);
        }
    }

    public static void Publish(RaceEventType type)
    {
        UnityEvent thisEvent;

        if (Events.TryGetValue(type, out thisEvent))
        {
            thisEvent.Invoke();
        }
    }
}
```

## 6.3.1 레이스 이벤트 버스 테스트하기

- CountdownTimer

```csharp
public class CountdownTimer : MonoBehaviour
{
    private float _currentTime;
    private float duration = 3.0f;

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.COUNTDOWN, StartTimer);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.COUNTDOWN, StartTimer);
    }

    private void StartTimer()
    {
        StartCoroutine(Countdown());
    }

    private IEnumerator Countdown()
    {
        _currentTime = duration;

        while (_currentTime > 0)
        {
            yield return new WaitForSeconds(1f);
            _currentTime--;
        }
        RaceEventBus.Publish(RaceEventType.START);
    }

    private void OnGUI()
    {
        GUI.color = Color.blue;
        GUI.Label(new Rect(125, 0, 100, 20), "COUNTDOWN: " + _currentTime);
    }
}
```

- BikeController

```csharp
public class BikeController : MonoBehaviour
{
    private string _status;

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.START, StartBike);
        RaceEventBus.Subscribe(RaceEventType.STOP, StopBike);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.START, StartBike);
        RaceEventBus.Unsubscribe(RaceEventType.STOP, StopBike);
    }

    private void StartBike()
    {
        _status = "Started";
    }

    private void StopBike()
    {
        _status = "Stopped";
    }

    private void OnGUI()
    {
        GUI.color = Color.green;
        GUI.Label(new Rect(10, 60, 200, 20), "BIKE STATUS: " + _status);
    }
}
```

- HUDController

```csharp
public class HUDController : MonoBehaviour
{
    private bool _isDisplayOn;

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.START, DisplayHUD);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.START, DisplayHUD);
    }

    private void DisplayHUD()
    {
        _isDisplayOn = true;
    }

    private void OnGUI()
    {
        if(_isDisplayOn)
        {
            if (GUILayout.Button("Stop Race"))
            {
                _isDisplayOn = false;
                RaceEventBus.Publish(RaceEventType.STOP);
            }
        }
    }
}
```

- ClientEventBus

```csharp
public class ClientEventBus : MonoBehaviour
{
    private bool _isButtonEnabled;

    private void Start()
    {
        gameObject.AddComponent<HUDController>();
        gameObject.AddComponent<CountdownTimer>();
        gameObject.AddComponent<BikeController>();

        _isButtonEnabled = true;
    }

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.STOP, Restart);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.START, Restart);
    }

    private void Restart()
    {
        _isButtonEnabled = true;
    }
    
    private void OnGUI()
    {
        if (_isButtonEnabled)
        {
            if (GUILayout.Button("Start Countdown"))
            {
                _isButtonEnabled = false;
                RaceEventBus.Publish(RaceEventType.COUNTDOWN);
            }
        }
    }

}
```

# 6.4 대안 살펴보기

- 옵저버
    - 서브젝트가 옵저버 목록을 유지 및 관리하고 내부 상태 변경을 알리는 패턴
    - 엔티티 그룹 간의 일대다 관계를 설정할 때 고려
- 이벤트 큐
    - 게시자가 생성한 이벤트를 큐에 저장하고 편한 시간에 구독자에게 전달 가능
    - 게시자와 구독자 간 시각적 관계를 분리
- ScriptableObject
    - SO로 이벤트 시스템 구현 가능
    - 새로운 커스텀 게임 이벤트 쉽게 생성 가능