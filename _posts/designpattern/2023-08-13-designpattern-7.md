---
layout: single
title: "10.방문자 패턴으로 파워업 구현하기"
categories: designpattern
toc: true
toc_sticky: true
---
# 10.1 방문자 패턴 이해하기

![Visitor](https://github.com/Kurizzing/Kurizzing.github.io/assets/48445259/9d40388d-807f-49f5-a4c5-4a85eecee4e5)

- 객체에 방문한 방문자는 객체의 특정 요소를 작업, 객체는 직접 수정하지 않아도 방문자에게서 새로운 기능을 얻음
- IVisitor: 방문자가 될 클래스가 구현해야 할 인터페이스
- Ivisitable: 방문이 가능한 클래스가 될 클래스가 구현해야 하는 인터페이스
    - 진입점을 제공하는 accept()메서드 포함

## 10.1.1 방문자 패턴의 장단점

- 장점
    - 개방/폐쇄: 직접 수정하지 않고도 다른 클래스의 오브젝트와 함께 작동하는 새로운 동작을 추가 가능
    - 단일 책임: 데이터를 보유하는 객체(방문 가능), 특정 행동을 도입하는 책임을 지는 객체(방문자)
- 단점
    - 접근성: 패턴을 사용하기 전보다 더 많은 공개 속성을 노출해야 할 수 있음
    - 복잡성: 구조적으로 복잡
    

# 10.2 파워업 매커니즘 설계하기

- 세분화: 파워업 엔티티는 여러 속성을 동시에 강화 가능
- 시간: 파워업 효과는 일정 시간이 지나도 사라지지 않음

# 10.3 파워업 매커니즘 구현하기

## 10.3.1 파워업 시스템 구현하기

- Visitor

```csharp
public interface IVisitor
{ 
    void Visit(BikeShield bikeShield);
    void Visit(BikeEngine bikeEngine);
    void Visit(BikeWeapon bikeWeapon);
}
```

- 방문 가능 클래스가 구현해야 하는 인터페이스

```csharp
public interface IBikeElement
{ 
    void Accept(IVisitor visitor);
}
```

- 파워업 매커니즘이 작동하도록 하는 메인 클래스

```csharp
[CreateAssetMenu(fileName = "PowerUp", menuName = "PowerUp")]
public class PowerUp : ScriptableObject, IVisitor
{
    public string powerupName;
    public GameObject powerupPrefab;
    public string powerupDescription;
    
    [Tooltip("Fully heal shield")]
    public bool healShield;

    [Range(0.0f, 50f)]
    [Tooltip(
        "Boost turbo settings up to increments of 50/mph")]
    public float turboBoost;

    [Range(0.0f, 25)]
    [Tooltip(
        "Boost weapon range in increments of up to 25 units")]
    public int weaponRange;

    [Range(0.0f, 50f)]
    [Tooltip(
        "Boost weapon strength in increments of up to 50%")]
    public float weaponStrength;

    public void Visit(BikeShield bikeShield) 
    {
        if (healShield) 
            bikeShield.health = 100.0f;
    } 

    public void Visit(BikeWeapon bikeWeapon) 
    {
        int range = bikeWeapon.range += weaponRange;

        if (range >= bikeWeapon.maxRange)
            bikeWeapon.range = bikeWeapon.maxRange;
        else
            bikeWeapon.range = range;
        
        float strength = 
            bikeWeapon.strength += 
                Mathf.Round(
                    bikeWeapon.strength 
                    * weaponStrength / 100);

        if (strength >= bikeWeapon.maxStrength)
            bikeWeapon.strength = bikeWeapon.maxStrength;
        else
            bikeWeapon.strength = strength;
    }

    public void Visit(BikeEngine bikeEngine)
    {
        float boost = bikeEngine.turboBoost += turboBoost;
        
        if (boost < 0.0f)
            bikeEngine.turboBoost = 0.0f;

        if (boost >= bikeEngine.maxTurboBoost)
            bikeEngine.turboBoost = bikeEngine.maxTurboBoost;
    }
}
```

- 오토바이를 제어하는 클래스

```csharp
public class BikeController : MonoBehaviour, IBikeElement
{
    private List<IBikeElement> _bikeElements = 
        new List<IBikeElement>();
    
    void Start()
    {
        _bikeElements.Add(
            gameObject.AddComponent<BikeShield>());
        _bikeElements.Add(
            gameObject.AddComponent<BikeWeapon>());
        _bikeElements.Add(
            gameObject.AddComponent<BikeEngine>());
    }
    
    public void Accept(IVisitor visitor)
    {
        foreach (IBikeElement element in _bikeElements)
        {
            element.Accept(visitor);
        }
    }
}
```

- 개별 방문 가능 요소(나머지 생략)

```csharp
public class BikeWeapon : MonoBehaviour, IBikeElement
{
    [Header("Range")]
    public int range = 5; 
    public int maxRange = 25;
    
    [Header("Strength")]
    public float strength = 25.0f;
    public float maxStrength = 50.0f;
    
    public void Fire()
    {
        Debug.Log("Weapon fired!");
    }

    public void Accept(IVisitor visitor)
    {
        visitor.Visit(this);
    }
    
    void OnGUI() 
    {
        GUI.color = Color.green;
        
        GUI.Label(
            new Rect(125, 40, 200, 20), 
            "Weapon Range: " + range);
        
        GUI.Label(
            new Rect(125, 60, 200, 20), 
            "Weapon Strength: " + strength);
    }
}
```