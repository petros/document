# 시작하기

## Step 1: 최소 요구사항 확인하기

* Ubuntu xenial\(16.04\)

## Step 2: NUGU SDK 설치하기

NUGU SDK for Linux는 설치에 필요한 패키지\(`*.deb`\) 파일들을 쉽게 다운로드 받을 수 있도록 Ubuntu에서 제공하는 [PPA](https://launchpad.net/~nugulinux/+archive/ubuntu/sdk)\([https://launchpad.net](https://launchpad.net)\)를 통해 배포하고 있습니다.

### PPA 추가하기

아래 명령을 통해 시스템에 PPA를 추가할 수 있습니다.

{% tabs %}
{% tab title="Ubuntu" %}
```bash
sudo add-apt-repository ppa:nugulinux/sdk
sudo apt-get update
```
{% endtab %}

{% tab title="Debian" %}
사용하고 있는 Debian 버전에 따라 NUGU SDK의 Bionic 또는 Xenial PPA 주소를 추가해야 합니다.

```bash
# Debian 버전이 Buster일 경우
$ sudo vi /etc/apt/sources.list.d/nugu.list
deb http://ppa.launchpad.net/nugulinux/sdk/ubuntu bionic main

# Debian 버전이 Stretch일 경우
$ sudo vi /etc/apt/sources.list.d/nugu.list
deb http://ppa.launchpad.net/nugulinux/sdk/ubuntu xenial main
```

이제 NUGU SDK PPA에 대한 인증키를 설치해야 합니다.

```bash
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key 5DE933034EEA59C4
sudo apt-get update
```
{% endtab %}
{% endtabs %}

### 패키지 설치하기

NUGU SDK for Linux는 아래의 패키지들로 구성되어 있습니다.

| 패키지 | 설명 |
| :--- | :--- |
| `libnugu` | NUGU SDK 동작에 필요한 패키지 - shared library\(`*.so.{version}` files\) |
| `libnugu-plugins-default` | 기본 제공 플러그인 모음 패키지 - `gstreamer.so`, `opus.so`, `portaudio.so` 등 |
| `libnugu-dev` | 빌드에 필요한 패키지 - header files, pkg-config\(`nugu.pc`\) 및 `libnugu.so` |
| `libnugu-examples` | 예제 프로그램 패키지 - 콘솔 기반의 예제 프로그램, OAuth2 클라이언트 예제 |

아래 명령을 통해 시스템에 설치할 수 있습니다.

```bash
sudo apt-get install libnugu libnugu-plugins-default libnugu-dev libnugu-examples
```

## Step 3: 프로젝트 설정하기

### PoC 정보 입력하기

{% hint style="warning" %}
NUGU PoC를 생성하기 위해서는 NUGU Developers를 통해 제휴가 필요합니다.  
더 자세한 내용은 [NUGU SDK 소개](https://developers.nugu.co.kr/#/sdk/nuguSdkInfo)에서 확인이 가능합니다.
{% endhint %}

발급받은 PoC 정보를 확인하기 위해서 [NUGU SDK PoC목록](https://developers.nugu.co.kr/#/sdk/pocList)으로 이동해서 Client ID, Client Secret, Redirect URI 정보를 확인하세요.

### 음성인식 모델 파일 설정하기

#### 다운로드 받기

[NUGU SDK PoC목록](https://developers.nugu.co.kr/#/sdk/pocList)에서 음성인식 모델 파일을 다운로드 받습니다.

#### 설정하기

다운로드가 완료되면, Linux 디바이스에 임의의 디렉토리를 생성한 후, 아래와 같은 파일명으로 복사해 주시면 됩니다.

* `nugu_model_wakeup_net.raw`  - keyword detection에 사용되는 model 파일 \(1/2\)
* `nugu_model_wakeup_search.raw`  - keyword detection에 사용되는 model 파일 \(2/2\)
* `nugu_model_epd.raw` - VAD\(Voice Activity Detection\) 에 사용되는 model 파일

### OAuth2 client 구현하기

NUGU SDK for Linux는 아래의 이유로 인해 iOS, Android와 달리 인증을 위한 기능을 제공하고 있지 않습니다. 

* Linux 기반의 제품은 대부분 디스플레이가 없어 별도의 연동 어플리케이션을 통한 사용자 인증을 필요로 합니다.
* 다른 플랫폼과 달리 GUI framework이 다양하기 때문에 표준화된 인증 UI를 SDK에서 제공해 주기 어렵습니다.

하지만, 인증을 쉽게 테스트할 수 있도록 아래의 NUGU SDK for Linux Github에 Python으로 작성된 별도의 웹기반 OAuth2 client 예제를 제공하고 있습니다.

{% embed url="https://github.com/nugu-developers/nugu-linux/wiki/Sample-OAuth2" caption="NUGU SDK for Linux OAuth2 client python sample" %}

## Step 4: NUGU 사용하기

### GMainLoop 사용하기

NUGU SDK for Linux는 event-driven 방식으로 어플리케이션을 개발할 수 있도록 [glib](https://wiki.gnome.org/Projects/GLib)에서 제공하는 [event loop](https://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html)을 사용하고 있습니다.  따라서 `main()` 코드는 아래 구조로 작성되어야 합니다.

```c
#include <glib.h>

int main(int argc, char *argv[])
{
  GMainLoop *loop;

  /* event loop 생성 */
  loop = g_main_loop_new (NULL, FALSE);

  /* 사용자의 초기화 코드 등록(nugu sdk 코드 등) */

  /* event loop 시작 */
  g_main_loop_run (loop);

  /* event loop 해제 */
  g_main_loop_unref (loop);

  return 0;
}
```

### NUGU 음성인식 사용하기

음성인식을 요청하기 위해서는 아래와 같은 코드를 작성해야 합니다.

1. 헤더 파일\(nugu\_client.hh, capability\_factory.hh\)을 include에 포함시키고, `NuguClientKit, NuguCapability` namespace를 사용하도록 설정합니다.

   ```cpp
   #include <clientkit/nugu_client.hh>
   #include <capability/capability_factory.hh>

   using namespace NuguClientKit;
   using namespace NuguCapability;
   ```

2. `IASRHandler` 객체를 생성하고, 음성인식 모델 파일을 설정합니다.

   ```cpp
   auto my_asr_listener(std::make_shared<MyASR>());
   auto asr_handler(std::shared_ptr<IASRHandler>(
       CapabilityFactory::makeCapability<ASRAgent, IASRHandler>(my_asr_listener.get())));
   asr_handler->setAttribute(ASRAttribute { "/var/lib/nugu/model", "CLIENT", "PARTIAL" });
   ```

3. `NuguClient` 객체를 생성하고, `ASR Capability` 추가 후, SDK를 초기화 합니다.

   ```cpp
   NuguClient* nugu_client = new NuguClient());
   nugu_client->getCapabilityBuilder()
           ->add(CapabilityType::ASR, my_asr_listener.get())
           ->construct();
   nugu_client->initialize();
   ```

4. 음성인식 기능을 사용하기 위해 OAuth2 access-token 설정 후, NUGU 서비스 연결을 요청합니다.

   ```cpp
   auto network_manager(nugu_client->getNetworkManager());
   network_manager->setToken("...");
   network_manager->connect();
   ```

전체 코드는 Github의 아래 wiki에서 확인할 수 있습니다.

{% embed url="https://github.com/nugu-developers/nugu-linux/wiki/Create-your-first-application" caption="NUGU SDK for Linux Wiki - Create your first application" %}

### Build

NUGU SDK for Linux는 빌드 설정을 쉽게 구성할 수 있도록 `pkg-config` 파일을 제공합니다. 따라서, 아래와 같이 `nugu.pc`를 사용해 빌드 명령을 수행할 수 있습니다.

```bash
$ g++ -std=c++11 hello.cc `pkg-config --cflags --libs nugu` -o hello
```

## 더 알아보기

### Download the SDK source code

아래 Github 주소를 통해 NUGU SDK for Linux 전체 소스 코드를 다운로드 받을 수 있습니다.

{% embed url="https://github.com/nugu-developers/nugu-linux" caption="NUGU SDK for Linux Github repository" %}

