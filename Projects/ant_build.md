# ant로 build하기
- ant는 기본적으로 eclipse에 탑재가 되어 있음  
  만약 없다면 ant를 다운로드 받아 설치

## build.xml file 확인
- `build.xml` 파일을 작성하기 전, `build.properties` file에 원하는 property 정보를 다음과 같이 입력할 수 있습니다.
~~~
#Directories Infomation
src.dir = src
lib.dir = /blog/ojava/webapp/WEB-INF/lib
build.dir = build
classes.dir = "${build.dir}/classes
~~~
- ant의 설정파일인 build.xml 의 구조를 간단하게 살펴보도록 하겠습니다
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="agilejava" default="build" basedir=".">
</project>
~~~
- 위의 xml 코드는 기본이 되는 형태입니다  
  default 값이 `build`로 설정되었다는 의미는 build target을 지정함을 의미합니다.  
- `basedir = "."` 로 지정하면 build.xml file과 동일한 위치로 base directory를 지정하겠다는 의미입니다.
- 다음으로는 property 정의에 대해서 살펴보겠습니다.  
  다음과 같은 선언문으로 각각의 property 들을 선언할 수 있습니다.
~~~xml
<property name="project.name" value="AntTask"/>
<property name="PROJECT" value="D:/workspace/${project.name}" />
<property name="build.dir" value="${PROJECT}/build"/>
<property name="dist.dir" value="${PROJECT}/dist"/>
<property name="src.dir" value="${PROJECT}/src"/>
~~~
- 다음으로 필요한 라이브러리들의 클래스 패스를 잡습니다.  
~~~xml
<path id="project.classpath">
    <fileset dir="${PROJECT}/web/WEB-INF/lib" includes="**/*.jar" />
</path>
~~~
- 이번에는 컴파일을 수행해 보겠습니다. `target` 속성에 `depends` 라는 녀석이 보이고 `init` 이라는 값을 가지고 있습니다
- build 타겟을 실행할 때, init 타겟을 먼저 수행하라는 의미입니다
- `javac`는 컴파일을 수행하라는 의미를 가집니다  
  src ==> 소스 디렉토리,  destdir ==> 컴파일된 클래스 파일이 들어갈 디렉토리
- `includes`는 컴파일할 대상을 말합니다. 나머지 옵션은 에러 추적용입니다
~~~xml
<target name="build" depends="init">
    <javac srcdir="${src.dir}" destdir="${build.dir}" includes="**/*.java" debug="true" failonerror="true">
        <classpath refid="project.classpath" />
    </javac>
</target>
~~~
- `init` 타겟은 별 내용이 없습니다. 시작시와 끝날시에 메세지 보여주고 컴파일 시에 디렉토리를 생성시킵니다
~~~xml
<target name="init">
    <echo message="init... start" />
        <mkdir dir="${build.dir}"/>
        <mkdir dir="${dist.dir}"/>
    <echo message="init... end" />
</target>
~~~
- 이제 실행을 해보면 아래와 같은 메세지를 확인하실 수 있습니다
~~~
>ant build

Buildfile: build.xml

init:
     [echo] init... start
     [echo] init... end

build:
    [javac] Compiling 1 source file to D:\workspace\AntTask\build

BUILD SUCCESSFUL
Total time: 4 seconds
~~~
- 다음은 파일을 복사해 보겠습니다  
  빌드된 파일을 특정한 디렉토리로 복사합니다. 예를 들면 소스 컴파일이 성공했을 경우  
  WAS에 해당 클래스를 복사해서 넣는 것입니다
~~~xml
<target name="deploy" depends="build">
    <copy todir="${deploy.dir}" overwrite="true">
        <fileset dir="${build.dir}" >
            <include name="**/*.class" /> 
        </fileset>
    </copy>
</target>
~~~
- 다음과 같이 jar file을 만들 수도 있습니다
~~~xml
<target name="deployjar" depends="build">
    <echo message="create jar file : ${dist.dir}/${deploy.name}.jar" />
        <jar jarfile="${dist.dir}/${deploy.name}.jar" update="true">
            <fileset dir="${build.dir}">
                <include name="**/*.*"/>
            </fileset>
        </jar>
</target>
~~~
