# Background 동작 처리

Connection-oriented 방식을 사용하는 PoC 의 경우 application 이 background 로 전활될 경우 필요하다면 연결을 끊어주어야 합니다.

{% hint style="info" %}
Type1 PoC 에서 OAuth token 발급 결과 scope 에 "device:S.I.D" 가 포함된 경우에만 connection-oriented 방식을 사용합니다.
{% endhint %}

### Background 연결 종료

```text
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        ProcessLifecycleOwner.get().lifecycle
            .addObserver(MyLifecycleObserver())
    }
}

class MyLifecycleObserver : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun onForeground() {
        // 재연결
        ClientManager.getClient().connect()
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onBackground() {
        // 연결 종료
        ClientManager.getClient().disconnect()
    }
}
```

