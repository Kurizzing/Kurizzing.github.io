---
layout: single
title: "9.옵저버 패턴으로 컴포넌트 분리하기"
categories: designpattern
toc: true
toc_sticky: true
---
# 9.1 옵저버 패턴 이해하기
![Observer](https://github.com/Kurizzing/Kurizzing.github.io/assets/48445259/3b6d9763-8d94-4b03-a3cc-2d3427d105a2)

- 한 객체가 주체 역할을 하고 다른 객체가 관찰자 역할을 맡는 객체 간의 일대다 관계를 설정
- 주체 역할을 하는 객체는 내부에서 변경이 일어났을 때 관찰자에게 알리는 책임을 짐
- 주체는 무언가 바뀌었다는 것만 알리고 재량껏 반응하도록 함

## 9.1.1 옵저버 패턴의 장단점

- 장점
    - 역동성: 주체에 필요한 만큼의 객체를 관찰자로 추가 가능, 동적으로 제거 가능
    - 일대다: 일대다 관계가 있는 객체 간 이벤트 처리 시스템의 구현 문제를 우아하게 해결
- 단점
    - 무질서: 관찰자가 알림받는 순서를 보장하지 않음
    - 누수: 주체는 관찰자에 대한 강한 참조를 가져 메모리 누수 발생 가능

## 9.1.2 옵저버 패턴을 사용하는 경우

- 객체 간의 일대다 관계와 관련된 문제를 해결

# 9.2 옵저버 패턴으로 핵심 컴포넌트 분리하기

- 오토바이 경주 게임이므로 오토바이가 주요 주체, HUD, 카메라가 오토바이를 관찰

# 9.3 옵저버 패턴 구현하기

- Subject

```csharp
public abstract class Subject : MonoBehaviour
{
    private readonly ArrayList _observers = new ArrayList();

    public void Attach(Observer observer)
    {
        _observers.Add(observer);
    }

    public void Detach(Observer observer)
    {
        _observers.Remove(observer);
    }

    public void NotifyObservers()
    {
        foreach (Observer observer in _observers)
        {
            observer.Notify(this);
        }
    }
}
```

- Observer

```csharp
public abstract class Observer : MonoBehaviour
{
    public abstract void Notify(Subject subject);
}
```

- BikeController: subject
    - 직접 HUDController나 CameraController를 호출하지 않고, 바뀐 것이 있다는 것만 알림

```csharp
public class BikeController : Subject
{
    public bool IsTurboOn
    {
        get; private set;
    }

    public float CurrentHealth
    {
        get { return health; }
    }

    private bool _isEngineOn;
    private HUDController _hudController;
    private CameraController _cameraController;

    [SerializeField]
    private float health = 100.0f;

    private void Awake()
    {
        _hudController = gameObject.AddComponent<HUDController>();
        _cameraController = (CameraController) FindObjectOfType(typeof(CameraController));
    }

    private void Start()
    {
        StartEngine();
    }

    private void OnEnable()
    {
        if (_hudController)
            Attach(_hudController);
        
        if (_cameraController)
            Attach(_cameraController);
    }

    private void OnDisable()
    {
        if (_hudController)
            Detach(_hudController);
        
        if (_cameraController)
            Detach(_cameraController);
    }

    private void StartEngine()
    {
        _isEngineOn = true;
        NotifyObservers();
    }

    public void ToggleTurbo()
    {
        if (_isEngineOn)
            IsTurboOn = !IsTurboOn;
        
        NotifyObservers();
    }

    public void TakeDamage(float amount)
    {
        health -= amount;
        IsTurboOn = false;

        NotifyObservers();

        if (health < 0)
            Destroy(gameObject);
    }
}
```

- Observer를 상속하는 객체들

```csharp
public class HUDController : Observer
{
    private bool _isTurboOn;
    private float _currentHealth;
    private BikeController _bikeController;

    private void OnGUI()
    {
        GUILayout.BeginArea(new Rect(50, 50, 100, 200));
        GUILayout.BeginHorizontal("box");
        GUILayout.Label("Helath: " + _currentHealth);
        GUILayout.EndHorizontal();

        if (_isTurboOn)
        {
            GUILayout.BeginHorizontal("box");
            GUILayout.Label("Turbo Activated");
            GUILayout.EndHorizontal();
        }

        if (_currentHealth <= 50.0f)
        {
            GUILayout.BeginHorizontal("box");
            GUILayout.Label("WARNING: Low Health");
            GUILayout.EndHorizontal();
        }

        GUILayout.EndArea();
    }

    public override void Notify(Subject subject)
    {
        if (!_bikeController)
        {
            _bikeController = subject.GetComponent<BikeController>();
        }

        if(_bikeController)
        {
            _isTurboOn = _bikeController.IsTurboOn;
            _currentHealth = _bikeController.CurrentHealth;
        }
    }
}
```

```csharp
public class CameraController : Observer
{
    private bool _isTurboOn;
    private Vector3 _initialPosition;
    private float _shakeMagnitude = 0.1f;
    private BikeController _bikeController;

    private void OnEnable()
    {
        _initialPosition = gameObject.transform.position;
    }

    private void Update()
    {
        if (_isTurboOn)
        {
            gameObject.transform.position = _initialPosition + (Random.insideUnitSphere * _shakeMagnitude);
        }
        else
        {
            gameObject.transform.localPosition = _initialPosition;
        }
    }

    public override void Notify(Subject subject)
    {
        if (!_bikeController)
            _bikeController = subject.GetComponent<BikeController>();
        
        if (_bikeController)
            _isTurboOn = _bikeController.IsTurboOn;
    }
}
```

# 9.4 대안 살펴보기

- C#의 기본 이벤트 시스템