# [Fragment Lifecycle과 LiveData](http://pluu.github.io/blog/android/2020/01/25/android-fragment-lifecycle/)

   - Fragment의 생명주기는 Attach되는 Activity 생명주기에 영향을 받는다.
   - Fragment는 Activity와 다르게 ``onDestroy``가 호출되지 않은 상태에서 ``onCreateView``가 여러 번 호출될 수 있다. 
        - Fragment의 Lifecycle이 Destory되지 않은 상황에서 LiveData에 새로운 Observer가 등록되어 복수의 Observer가 호출되는 현상이 발생할 가능성이 있다.
- Fragment에는 2개의 LifeCycle이 존재
  - Fragment Lifcecycle : 생명 주기에 해당하는 내용
  - Fragment View Lifecycle : 위 Issue에 사용 패턴을 개선하기 위해 도입된 Lifecycle
- ``Fragment Lifecycle`` **>** ``Fragment View Lifecycle``
- UI 업데이트는 Fragment View Lifecycle이 적절하다.
- ``onCreateView()`` ``onViewCreated()`` ``onActivityCreate()`` 에서 LiveData사용 시 ``ViewLifecycleOwner``를 사용하도록 권장

![img](https://miro.medium.com/max/1596/1*hK_YRdty1GoafABfug-r4g.png)