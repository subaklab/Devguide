# 파일과 코드 설치하기

가장 먼저해야할 일은 Mac app store에서 Xcode를 설치하는 것입니다. 일단 설치해야 새로운 터미널도 열고 command line tool을 설치합니다. :

<div class="host-code"></div>

```bash
xcode-select --install
```

## Homebrew 설치

Mac OS X를 사용하는 경우 [Homebrew package manager](http://mxcl.github.com/homebrew/) 사용을 권장합니다. Homebrew 설치는 [installation instructions](http://mxcl.github.com/homebrew/)를 따라하면 쉽고 간편합니다.

Homebrew를 설치한 후에, 이 명령을 shell에 복사합니다. :

<div class="host-code"></div>

```sh
brew tap PX4/homebrew-px4
brew tap osrf/simulation
brew update
brew install git bash-completion genromfs kconfig-frontends gcc-arm-none-eabi
brew install astyle cmake ninja
# simulation tools
brew install ant graphviz sdformat3 eigen protobuf
brew install homebrew/science/opencv
```

다음으로 필요한 python 패키지를 설치합니다. :

<div class="host-code"></div>

```sh
sudo easy_install pip
sudo pip install pyserial empy pandas jinja2
```

### jMAVSim을 위한 Java

jMAVSim을 사용하고자 한다면, [Java JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)을 설치해야 합니다.

## Snapdragon Flight

Snapdrong Flight에서 개발하는 경우 Ubuntu VM을 사용해야 합니다. Qualcomm은 Ubuntu용으로만 툴을 제공하고 있습니다. PX4 개발팀은 VMWare으로 개발하며 USB 안정성이 있습니다.

## Simulation

OS X는 CLANG이 미리 설치되어 있습니다. 추가적으로 설치하지 않아도 됩니다.

## Editor / IDE

마지막으로 Qt Creator app을 다운로드 받고 설치합니다. :
[Download](http://www.qt.io/download-open-source/#section-6)

자 이제 [첫 빌드](starting-building.md)를 진행할 수 있습니다!
