---
description: 디바이스 배터리 정보를 Play 로 전달하기 위한 규격
---

# Battery

## Version

최신 버전은 1.1 입니다.

| Version | Date | Description |
| :--- | :--- | :--- |
| 1.0 | 2020.02.25 | 규격 추가 |
| 1.1 | 2020.04.29 | Context 에 approximateLevel 필드 추가 |

## SDK Interface

### BatteryAgent 사용

Battery interface 규격에 따른 디바이스의 정보 전달은 BatteryAgent 가 처리합니다.

{% hint style="warning" %}
Linux 는 BatteryAgent 를 지원하지 않습니다.
{% endhint %}

{% tabs %}
{% tab title="Android" %}
NuguAndroidClient instance 를 통해 BatteryAgent instance 에 접근할 수 있습니다.

```text
val batteryAgent = nuguAndroidClient.getAgent(DefaultBatteryAgent.NAMESPACE)
```

NuguAndroidClient 에 배터리 정보을 전달를 위한 기본 BatteryStatusProvider 구현이 포함되어 있습니다.

BatteryStatusProvider 을 직접 구현하려면 NuguAndroidClient 생성시 추가합니다.

```text
class MyBatteryStatusProvider: BatteryStatusProvider {
    ...
}
NuguAndroidClient.Builder(...)
    .batteryStatusProvider(MyBatteryStatusProvider())
```
{% endtab %}
{% endtabs %}

### Context 구성

디바이스 배터리 정보를 [Context](battery.md#context) 에 포함시켜 주어야 합니다.

{% tabs %}
{% tab title="Android" %}
BatteryStatusProvider 를 구현합니다.

```text
class MyBatteryStatusProvider: BatteryStatusProvider {
    override fun getBatteryLevel(): Int {
        ...
    }

    override fun isCharging(): Boolean? {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

## Context

```text
{
  "Location": {
    "version": "1.1",
    "level": {{LONG}},
    "charging": {{BOOLEAN}},
    "approximateLevel": {{BOOLEAN}}
  }
}
```

<table>
  <thead>
    <tr>
      <th style="text-align:left">parameter</th>
      <th style="text-align:left">type</th>
      <th style="text-align:left">mandatory</th>
      <th style="text-align:left">description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">level</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Y</td>
      <td style="text-align:left">&#xBC30;&#xD130;&#xB9AC; &#xC794;&#xB7C9;(0 ~ 100)</td>
    </tr>
    <tr>
      <td style="text-align:left">charging</td>
      <td style="text-align:left">boolean</td>
      <td style="text-align:left">Y</td>
      <td style="text-align:left">&#xCDA9;&#xC804; &#xC5EC;&#xBD80;</td>
    </tr>
    <tr>
      <td style="text-align:left">approximateLevel</td>
      <td style="text-align:left">boolean</td>
      <td style="text-align:left">N</td>
      <td style="text-align:left">
        <p>&#xBC30;&#xD130;&#xB9AC; &#xC794;&#xB7C9;&#xC758; &#xADFC;&#xC0AC;&#xCE58;
          &#xC5EC;&#xBD80;</p>
        <p>( &#xC77C;&#xBD80; &#xB514;&#xBC14;&#xC774;&#xC2A4;&#xC758; &#xACBD;&#xC6B0;
          &#xC815;&#xD655;&#xD55C; &#xBC30;&#xD130;&#xB9AC; &#xC794;&#xB7C9;&#xC744;
          &#xCE21;&#xC815;&#xD560; &#xC218; &#xC5C6;&#xC74C;)</p>
      </td>
    </tr>
  </tbody>
</table>

