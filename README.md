# macOS에서 Data Science를 위한 Native 개발환경 설정

(https://comafire.github.io/pages/platform-native-env-for-data-science-on-macos/)

## Home brew
OSX 에서 편리하게 linux 개발 툴을 사용하기 위해 필수적으로 필요한 패키지 관리자입니다. (http://brew.sh/index_ko.html)

설치는 간단합니다. 아래와 같이 xcode 개발 툴 설치 후, brew 설치 스크립트를 수행해 주면 됩니다.

```bash
> xcode-select --install
> /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

이제 필요한 linux 개발 툴들이 있으면 brew install 명령을 통해 손쉽게 설치할수 있습니다. 

## Git

소스의 버전을 관리하기 위해 brew 를 통해 버전 관리 툴인 git을 설치합니다.

```bash
> brew update
> brew install git
> brew install git-flow-avh
```

Git Manual
* Git 간편안내서: https://rogerdudler.github.io/git-guide/index.ko.html
* Git Book: https://git-scm.com/book/ko/v2
* Git Flow(Git 개발 브렌치 전략): https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html

## pyenv, pyenv-virtualenv, direnv

프로젝트를 진행할때, 파이썬의 특성상 파이썬 자체의 버전과 패키지 버전들에 대한 의존성이 생기게 됩니다. 프로젝트 별로 편리하게 의존성을 관리하기 위해 3가지 툴을 조합하여 사용합니다.

* pyenv: 로컬에 멀티 버전의 파이썬을 설치/사용, 파이썬 버전에 대한 의존성을 해결
* virtualenv: 로컬에 멀티 파이썬 환경을 설치/사용, 프로젝트별 패키지에 대한 의존성을 해결
* direnv: 프로젝트 디렉토리에 들어갈때 마다 자동 환경 셋팅, .bash_profile 과 비슷한 역활

3가지 툴을 설치하고 기본 셋팅합니다.

```bash
> brew update
> brew install pyenv
> brew install pyenv-virtualenv
> brew install direnv
> echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc 
```

pyenv를 이용해 파이썬 여러 버전을 설치합니다. 주로 사용하는 2.x 대와 3.x 대의 최신 버전을 설치해 봅니다. 2.x 의 지원종료가 다가오기에 3.x 사용을 추천합니다. apache spark 과의 호완성을 위해 여기서는 3.7.6 버전을 사용합니다. (https://stackoverflow.com/questions/58700384/how-to-fix-typeerror-an-integer-is-required-got-type-bytes-error-when-tryin)

```bash
> pyenv install 2.7.16
> pyenv install 3.7.6
> pyenv versions
* system (set by /Users/comafire/.pyenv/version)
  2.7.16
  3.7.6
```

pyenv-virtualenv를 이용해 각 버전의 파이썬 가상환경을 만들어 봅니다.

```bash
> pyenv virtualenv 2.7.16 pyenv-2.7.16
> pyenv virtualenv 3.7.6 macos-jupyter
> pyenv virtualenvs
  ...
  pyenv-2.7.16 (created from /Users/comafire/.pyenv/versions/2.7.16)
  macos-jupyter (created from /Users/comafire/.pyenv/versions/3.7.6)
```

프로젝트를 생성하고 프로젝트의 기본 환경을 자동 셋팅할 수 있도록 direnv를 사용해 봅니다. direnv를 이용하면 프로젝트 폴더에 들어갈때 마다 자동으로 프로젝트에 맞는 특정 파이썬 가상환경을 활성화할 수 있습니다. 방법은 간단히 프로젝트의 최상위 디렉토리 아래 .envrc 파일을 만들고 아래 필요한 환경설정 내용을 적어주면 됩니다.

```bash
> vi .envrc

eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
export PYENV_VIRTUALENV_DISABLE_PROMPT=1

pyenv activate macos-jupyter

```

저장 후, 해당 환경을 활성화 하기 위해 아래 명령을 수행해 줍니다.

```bash
> direnv allow .
```

이후 부터는 .envrc 가 있는 프로젝트 디렉토리에 들어가면 자동으로 .envrc 내에 있는 환경이 활성화되고, 디렉토리를 벗어나면 자동으로 환경이 해제 됩니다.

```bash
> cd macos-jupyter
direnv: loading ~/macos-jupyter/.envrc
direnv: export +PYENV_ACTIVATE_SHELL +PYENV_SHELL +PYENV_VERSION +PYENV_VIRTUALENV_DISABLE_PROMPT +PYENV_VIRTUAL_ENV +VIRTUAL_ENV ~PATH

> cd ../
direnv: unloading
```

## Spark & Jupyter & Python Data Science Library

macOS 에서 Spark 을 설치하여 사용하는 방법은 여러가지가 있지만, 버전 의존성 및 설정의 편리성을 위해 github 저장소에 Spark 및 자주사용하는 Data Science Library 그리고 Jupyter를 띄울수 있도록 템플릿 환경을 저장하여 놓고 사용합니다.

```bash
> git clone https://github.com/comafire/macos-jupyter.git
```

Spark 을 위한 소스를 다운받아 프로젝트 내부의 usr/local 디렉토리내에 압축을 풀어 줍니다.

* openjdk-13.0.2_osx-x64_bin.tar.gz 
* scala-2.12.10.tgz
* spark-2.4.5-bin-hadoop2.7.tgz

```bash
> mkdir -p usr/local
> wget https://download.java.net/java/GA/jdk13.0.2/d4173c853231432d94f001e99d882ca7/8/GPL/openjdk-13.0.2_osx-x64_bin.tar.gz
> wget https://downloads.lightbend.com/scala/2.12.10/scala-2.12.10.tgz
> wget http://mirror.navercorp.com/apache/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz
> tar -zxvf openjdk-13.0.2_osx-x64_bin.tar.gz
> tar -zvxf scala-2.12.10.tgz
> tar -zxvf spark-2.4.5-bin-hadoop2.7.tgz
> ln -s jdk-13.0.2.jdk openjdk
> ln -s scala-2.12.10 scala
> ln -s spark-2.4.5-bin-hadoop2.7 spark
```

macos-jupyter 내부 디렉토리 안에 미리 .envrc 안에 기본 설정이 되어 있으며, 추가적으로 필요한 부분은 수정해서 사용할 수 있습니다.

```bash
> vi .envrc

# BASE
export MACOS_JUPYTER=$PWD

# Java
export JAVA_HOME="$MACOS_JUPYTER/usr/local/openjdk/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
export CPPFLAGS="-I$JAVA_HOME/include"

# Scala
export SCALA_HOME="$MACOS_JUPYTER/usr/local/scala"
export PATH="$SCALA_HOME/bin:$PATH"

# Python
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
export PYENV_VIRTUALENV_DISABLE_PROMPT=1

pyenv activate macos-jupyter

# Spark 
export SPARK_HOME="$MACOS_JUPYTER/usr/local/spark"
export PYSPARK_PYTHON="$(which python3)"
export PYSPARK_DRIVER_PYTHON="$PYSPARK_PYTHON"
export PY4J_VERSION=0.10.7
export PATH="${SPARK_HOME}/bin:$PATH"
export PYTHONPATH="$SPARK_HOME/python/:$PYTHONPATH"
export PYTHONPATH="$SPARK_HOME/python/lib/py4j-$PY4J_VERSION-src.zip:$PYTHONPATH"

# Jupyter
export LOCALE="ko_KR.UTF-8"

# Jupyter
export JUPYTER_PORT="7010" # Your Jupyter Port
export JUPYTER_PASSWORD="notebooks" # Your Jupyter Password
export JUPYTER_BASEURL="jupyter" # Your Jupyter BaseURL, ex) http://localhost:8010/jupyter

```

이제 필요한 Data Science 라이브러리를 설치합니다. requirements.txt 파일내에 기본 설치 라이브러리가 있으며, 필요시 추가해서 사용하면 됩니다.

```bash
> brew install cmake
> pip3 install --upgrade --user pip
> pip3 install -r etc/pip/requirements.txt
> python3 -m spylon_kernel install --user
```

이제 아래 명령으로 jupyter 노트북을 띄우게 되면 macOS 상에서 독립적인 가상 환경내에서 프로젝트 진행이 가능하게 됩니다.

```bash
> ./bin/run_jupyter.sh

[I 16:11:31.013 LabApp] Loading IPython parallel extension
[I 16:11:31.248 LabApp] JupyterLab extension loaded from /Users/comafire/.pyenv/versions/3.7.6/envs/macos-jupyter/lib/python3.7/site-packages/jupyterlab
[I 16:11:31.249 LabApp] JupyterLab application directory is /Users/comafire/.pyenv/versions/3.7.6/envs/macos-jupyter/share/jupyter/lab
[I 16:11:31.251 LabApp] Serving notebooks from local directory: /Users/comafire/LocalDrive/macos-jupyter
[I 16:11:31.251 LabApp] The Jupyter Notebook is running at:
[I 16:11:31.252 LabApp] http://localhost:7010/jupyter/
```
