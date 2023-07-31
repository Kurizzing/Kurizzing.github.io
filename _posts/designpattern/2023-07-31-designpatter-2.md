---
layout: single
title: "5.상태 패턴으로 캐릭터 상태 관리하기"
categories: designpattern
toc: true
toc_sticky: true
---
# 5.2 상태 패턴의 개요

![State](https://github.com/Kurizzing/Kurizzing.github.io/assets/48445259/3400e825-40ac-4385-bf8b-2768704909c7)

- 상태 패턴의 핵심 요소
    - Context: 클라이언트가 객체 내부 상태를 변경할 수 있도록 요청하는 인터페이스를 정의, 현재 상태에 대한 포인터 보유
    - IState:  구체적인 상태 클래스로 연결할 수 있는 인터페이스
    - ConcreteState: IState 인터페이스를 구현, Context 오브젝트가 상태의 동작을 트리거하기 위해 호출하는 퍼블릭 메서드 handle()를 노출

# 5.3 캐릭터 상태 정의하기

- 정지, 시작, 회전, 충돌

# 5.4 상태 패턴 구현하기

- 전통적인 접근 방식에서 다소 벗어나는 점에 주의

## 5.4.1 상태 패턴 구현하기

### IBikeState

```csharp
public interface IBikeState
{
    void Handle(BikeController controller);
}
```

### context class

```csharp
public class BikeStateContext
{
    public IBikeState CurrentState
    {
        get; set;
    }

    private readonly BikeController _bikeController;
    
    public BikeStateContext(BikeController bikeController)
    {
        _bikeController = bikeController;
    }

    public void Transition()
    {
        CurrentState.Handle(_bikeController);
    }

    public void Transition(IBikeState state)
    {
        CurrentState = state;
        CurrentState.Handle(_bikeController);
    }
}
```

### BikeController

```csharp
using UnityEngine;

public class BikeController : MonoBehaviour
{
    public float maxSpeed = 2.0f;
    public float trunDistance = 2.0f;
    public float CurrentSpeed { get; set; }

    public Direction CurrentTurnDirection
    {
        get; private set;
    }

    private IBikeState _startState, _stopState, _turnState;

    private BikeStateContext _bikeStateContext;

    /// <summary>
    /// Start is called on the frame when a script is enabled just before
    /// any of the Update methods is called the first time.
    /// </summary>
    private void Start()
    {
        _bikeStateContext = new BikeStateContext(this);

        _startState = gameObject.AddComponent<BikeStartState>();
        _stopState = gameObject.AddComponent<BikeStopState>();
        _turnState = gameObject.AddComponent<BikeTurnState>();

        _bikeStateContext.Transition(_stopState);
    }

    public void StartBike()
    {
        _bikeStateContext.Transition(_startState);
    }

    public void StopBike()
    {
        _bikeStateContext.Transition(_stopState);
    }

    public void Turn(Direction direction)
    {
        CurrentTurnDirection = direction;
        _bikeStateContext.Transition(_turnState);
    }
}
```

### BikeStopState, BikeStartState, BikeTurnState, Direction

```csharp
using UnityEngine;

public class BikeStopState : MonoBehaviour, IBikeState
{
    private BikeController _bikeController;

    public void Handle(BikeController bikeController)
    {
        if(!_bikeController)
            _bikeController = bikeController;
        
        _bikeController.CurrentSpeed = 0;
    }
}

public class BikeStartState : MonoBehaviour, IBikeState
{
    private BikeController _bikeController;

    public void Handle(BikeController bikeController)
    {
        if(!_bikeController)
            _bikeController = bikeController;
        
        _bikeController.CurrentSpeed = _bikeController.maxSpeed;
    }

    /// <summary>
    /// Update is called every frame, if the MonoBehaviour is enabled.
    /// </summary>
    private void Update()
    {
        if(_bikeController)
        {
            if(_bikeController.CurrentSpeed > 0)
            {
                _bikeController.transform.Translate(Vector3.forward 
								* (_bikeController.CurrentSpeed * Time.deltaTime));
            }
        }
    }
}

public class BikeTurnState : MonoBehaviour, IBikeState
{
    private BikeController _bikeController;
    private Vector3 _turnDirection;

    public void Handle(BikeController bikeController)
    {
        if(!_bikeController)
            _bikeController = bikeController;
        
        _turnDirection.x = (float)_bikeController.CurrentTurnDirection;

        if(_bikeController.CurrentSpeed > 0)
        {
            transform.Translate(_turnDirection * _bikeController.trunDistance);
        }
    }
}

public enum Direction
{
    Left = -1, Right = 1
}
```

# 5.5 상태 패턴의 장단점

- 장점
    - 캡슐화: 상태 패턴은 상태가 변할 때 개체에 동적으로 할당할 수 있는 컴포넌트의 집합, 개체의 상태별 행동 구현 가능
    - 유지 및 관리: 긴 조건문 없이 쉽게 새로운 상태 구현 가능
- 단점
    - 애니메이션 캐릭터 관리에 한계
    - 블렌딩: 애니메이션 블렌딩 방법을 제공하지 않음, 캐릭터 애니메이션 상태 간에 시각적 전환이 원활하지 않을 수 있음
    - 전환: 상태 간 관계를 정의하지 않음

# 5.6 대안 살펴보기

- 블랙보드/행동 트리: 복잡한 AI동작 구현시 블랙보드, 행동트리(BT) 고려
- 유한 상태 기계: 상태 패턴은 객체의 상태 종속적인 동작을 캡슐화, 유한 상태 기계는 특정 입력 트리거를 기반으로 하는 유한 상태 간 전환에 깊이 관여
- 메멘토: 개체에 이전 상태로 돌아가는 기능 제공