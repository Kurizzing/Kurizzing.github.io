---
layout: single
title: "8.오브젝트 풀 패턴으로 최적화하기"
categories: designpattern
toc: true
toc_sticky: true
---
# 8.1 오브젝트 풀 패턴 이해하기

![ObjectPool](https://github.com/Kurizzing/Kurizzing.github.io/assets/48445259/d2c56a7e-5775-459a-97be-cf82c0a56da1)

## 8.2.1 오브젝트 풀 패턴의 장단점

- 장점
    - 예측할 수 있는 메모리 사용: 특정 종류의 객체 인스턴스를 특정 양만큼 유지
    - 성능 향상: 새로운 객체 초기화에 드는 로딩 비용 절감
- 단점
    - 이미 관리되는 메모리에 대한 레이어링: 프로그래밍 언어가 이미 메모리 할당을 최적으로 관리함
    - 예측 불가능한 객체 상태: 잘못 처리시 풀에 되돌아온 객체가 손상되거나 파괴될 수 있음

## 8.2.2 오브젝트 풀 패턴을 사용하는 경우

- 최종 보스같은 엔티티를 오브젝트 풀에 넣는 것은 메모리 낭비
- 오브젝트 풀은 캐시가 아님

# 8.3 오브젝트 풀 패턴 구현하기

- Unity에서 제공하는 오브젝트 풀

[Unity - Scripting API: ObjectPool<T0>](https://docs.unity3d.com/ScriptReference/Pool.ObjectPool_1.html)

## 8.3.1 오브젝트 풀 패턴 구현하기

- Drone

```csharp
public class Drone : MonoBehaviour
{
    public IObjectPool<Drone> Pool { get; set; }
    public float _currentHealth;

    [SerializeField]
    private float maxHealth = 100.0f;
    [SerializeField]
    private float timeToSelfDestruct = 3.0f;

    private void Start()
    {
        _currentHealth = maxHealth;
    }

    private void OnEnable()
    {
        AttackPlayer();
        StartCoroutine(SelfDestruct());
    }

    private void OnDisable()
    {
        ResetDrone();
    }

    IEnumerator SelfDestruct()
    {
        yield return new WaitForSeconds(timeToSelfDestruct);
        TakeDamage(maxHealth);
    }

    private void ReturnToPool()
    {
        Pool.Release(this);
    }

    private void ResetDrone()
    {
        _currentHealth = maxHealth;
    }

    public void AttackPlayer()
    {
        Debug.Log("Attack player!");
    }

    public void TakeDamage(float amount)
    {
        _currentHealth -= amount;

        if(_currentHealth <= 0.0f)
            ReturnToPool();
    }
}
```

- ObjectPool: 드론 인스턴스의 풀을 관리

```csharp
public class DroneObjectPool : MonoBehaviour
{
    public int maxPoolSize = 10;   
    public int stackDefaultCapacity = 10;

    public IObjectPool<Drone> Pool
    {
        get
        {
            if (_pool == null)
                _pool = new ObjectPool<Drone>(
                    CreatedPooledItem,
                    OnTakeFromPool,
                    OnReturnedToPool,
                    OndestroyPoolObject,
                    true,
                    stackDefaultCapacity,
                    maxPoolSize);
            return _pool;
        }
    }

    private IObjectPool<Drone> _pool;

    private Drone CreatedPooledItem()
    {
        var go = GameObject.CreatePrimitive(PrimitiveType.Cube);

        Drone drone = go.AddComponent<Drone>();
        go.name = "Drone";
        drone.Pool = Pool;

        return drone;
    }

    private void OnReturnedToPool(Drone drone)
    {
        drone.gameObject.SetActive(false);
    }

    private void OnTakeFromPool(Drone drone)
    {
        drone.gameObject.SetActive(true);
    }
		// 풀에 더 이상 공간이 없을 때 호출됨
    private void OndestroyPoolObject(Drone drone)
    {
        Destroy(drone.gameObject);
    }

    public void Spawn()
    {
        var amount = Random.Range(1, 10);

        for (int i = 0; i < amount; i++)
        {
            var drone = Pool.Get();
            drone.transform.position = Random.insideUnitSphere * 10;
        }
    }
}
```

## 8.3.2 오브젝트 풀 구현 테스트하기

## 8.3.3 오브젝트 풀 구현 살펴보기

# 8.4 대안 살펴보기

- 프로토타입 패턴: 복제 매커니즘 사용, 새로운 객체 생성시 비용이 들지 않음, 새 객체 초기화 대신 프로토타입이라는 참조 객체에서 얕은 복사를 진행