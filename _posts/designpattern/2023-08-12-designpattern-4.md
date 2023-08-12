---
layout: single
title: "7.커맨드 패턴으로 리플레이 시스템 구현하기"
categories: designpattern
toc: true
toc_sticky: true
---
# 7.1 커맨드 패턴 이해하기

![CommandPattern](https://github.com/Kurizzing/Kurizzing.github.io/assets/48445259/b613e10b-a845-4296-be90-7833681c6836)

- 작업 실행 방법을 아는 객체에서 작업을 실행하는 객체를 분리 가능
- InputHandler는 정확히 어떤 액션을 해야하는지 알지 못해도 됨, 커맨드패턴 매커니즘이 처리하도록 함

```csharp
using UnityEngine;

public class InputHandlerEx : MonoBehaviour
{
    [SerializeField] 
    private Controller _characterController;
    private Command _spaceButton;

    private void Start()
    {
        _spaceButton = new JumpCommand();
    }

		// private void Update()
    // {
    //     if (Input.GetKeyDown("space"))
    //         CharacterController.Jump();
    // }

    private void Update()
    {
        if (Input.GetKeyDown("space"))
            _spaceButton.Execute(_characterController);
    }
}
```

- 기본 클래스
    - Invoker(호출자): 명령을 실행하는 방법을 알고 실행한 명령을 즐겨찾기할 수 도 있는 객체
    - Receiver(수신자): 명령을 받아서 수행하는 객체
    - CommandBase: 개별 ConcreteCommand 클래스가 상속하는 추상 클래스, 호출자가 특정 명령을 실행하기 위해 호출하는 Execute 메서드를 노출시킴

## 7.1.2 커맨드 패턴의 장단점

- 장점
    - 분리: 실행 방법을 아는 객체에게서 작업을 호출하는 객체를 분리
    - 시퀀싱: do/undo, 매크로, 명령 큐 구현 용이
- 단점
    - 복잡성: 각 명령이 클래스이므로, 수많은 클래스가 필요함

## 7.2.2 커맨드 패턴을 사용하는 경우

- 실행 취소
- 매크로
- 자동화

# 7.3 리플레이 시스템 설계하기

# 7.4 리플레이 시스템 구현하기

## 7.4.1 리플레이 시스템 구현하기

- Command의 기본 추상 클래스

```csharp
public abstract class Command
{
    public abstract void Execute();
}
```

- Command에서 파생되는 세 가지 구체적인 클래스

```csharp
public class ToggleTurbo : Command
{
    private BikeController _controller;

    public ToggleTurbo(BikeController controller)
    {
        _controller = controller;
    }

    public override void Execute()
    {
        _controller.ToggleTurbo();
    }
}
```

```csharp
public class TurnLeft : Command
{
    private BikeController _controller;

    public TurnLeft(BikeController controller)
    {
        _controller = controller;
    }

    public override void Execute()
    {
        _controller.Turn(BikeController.Direction.Left);
    }
}
```

```csharp
public class TurnRight : Command
{
    private BikeController _controller;

    public TurnRight(BikeController controller)
    {
        _controller = controller;
    }

    public override void Execute()
    {
        _controller.Turn(BikeController.Direction.Right);
    }
}
```

- Invoker

```csharp
public class Invoker : MonoBehaviour
{
    private bool _isRecording;
    private bool _isReplaying;
    private float _replayTime;
    private float _recordingTime;
    private SortedList<float, Command> _recordedCommands = new SortedList<float, Command>();

    public void ExecuteCommand(Command command)
    {
        command.Execute();

        if(_isRecording)
        {
            _recordedCommands.Add(_recordingTime, command);
        }

        Debug.Log("Recorded Time: " + _recordingTime);
        Debug.Log("Recorded Commands: " + command);
    }

    public void Record()
    {
        _recordingTime = 0.0f;
        _isRecording = true;
    }

    public void Replay()
    {
        _replayTime = 0.0f;
        _isReplaying = true;

        if(_recordedCommands.Count <= 0)
            Debug.LogError("No commands to replay!");
        
        _recordedCommands.Reverse();
    }
    
    private void FixedUpdate()
    {
        if (_isRecording)
            _recordingTime += Time.fixedDeltaTime;

        if(_isReplaying)
        {
            _replayTime += Time.deltaTime;

            if(_recordedCommands.Any())
            {
                if(Mathf.Approximately(
                    _replayTime, _recordedCommands.Keys[0]))
                {
                    Debug.Log("Replay Time: " + _replayTime);
                    Debug.Log("Replay Command: " + _recordedCommands.Values[0]);

                    _recordedCommands.Values[0].Execute();
                    _recordedCommands.RemoveAt(0);
                }
            }
        }
        else
        {
            _isReplaying = false;
        }
    }
}
```

## 7.4.2 리플레이 시스템 테스트하기

- Input Handler

```csharp
public class InputHandler : MonoBehaviour
{
    private Invoker _invoker;
    private bool _isReplaying;
    private bool _isRecording;
    private BikeController _bikeController;
    private Command _buttonA, _buttonD, _buttonW;

    private void Start()
    {
        _invoker = gameObject.AddComponent<Invoker>();
        _bikeController = gameObject.GetComponent<BikeController>();

        _buttonA = new TurnLeft(_bikeController);
        _buttonD = new TurnRight(_bikeController);
        _buttonW = new ToggleTurbo(_bikeController);
    }

    private void Update()
    {
        if (!_isReplaying && _isRecording)
        {
            if (Input.GetKeyUp(KeyCode.A))
                _invoker.ExecuteCommand(_buttonA);
            if (Input.GetKeyUp(KeyCode.D))
                _invoker.ExecuteCommand(_buttonD);
            if (Input.GetKeyUp(KeyCode.W))
                _invoker.ExecuteCommand(_buttonW);
        }
    }

    private void OnGUI()
    {
        if (GUILayout.Button("Start Recording"))
        {
            _bikeController.ResetPosition();
            _isReplaying = false;
            _isRecording = true;
            _invoker.Record();

        }

        if (GUILayout.Button("Stop Recording"))
        {
            _bikeController.ResetPosition();
            _isRecording = false;

        }

        if (!_isRecording)
        {
            if (GUILayout.Button("Start Replay"))
            {
                _bikeController.ResetPosition();
                _isRecording = false;
                _isReplaying = true;
                _invoker.Replay();
            }
        }
    }
}
```

- BikeController, 수신자 역할

```csharp
public class BikeController : MonoBehaviour
{
    public enum Direction
    {
        Left = -1,
        Right = 1
    }

    private bool _isTurboOn;
    private float _distance = 1.0f;

    public void ToggleTurbo()
    {
        _isTurboOn = !_isTurboOn;
        Debug.Log("Turbo Active: " + _isTurboOn.ToString());
    }

    public void Turn(Direction direction)
    {
        if (direction == Direction.Left)
            transform.Translate(Vector3.left * _distance);
        if (direction == Direction.Right)
            transform.Translate(Vector3.right * _distance);
    }

    public void ResetPosition()
    {
        transform.position = new Vector3(0, 0, 0);
    }
}
```

## 7.4.3 구현 검토하기

# 7.5 대안 살펴보기

- 메멘토
    - 객체를 이전 상태로 되돌리는 기능을 제공
- 큐/스택
    - InputHandler 클래스의 큐에 모든 입력을 저장