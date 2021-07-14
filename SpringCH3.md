- Spirng
    - 엔터프라이즈 어플리케이션을 개발하기에 적합한 프레임워크
    - 객체 관리를 해주는 빈 컨테이너 프레임워크

# IoC 패턴
- IoC(Inversion of Control) - 제어의 역전
    - 프로그래머가 작성한 프로그램이 재사용 라이브러리의 흐름 제어를 받게 되는 소프트웨어 디자인 패턴
    - 프로그램의 생명주기에 대한 주도권이 웹 애플리케이션 컨테이너에 있다.
- DI(Dependency Injection)
    - IoC의 하나의 형태
    - DIP를 지키려 하지만 자바에서는 인스턴스화를 통해 상위 코듈 또는 하위 모듈의 코드가 수반된다.
        - 즉, 결합도를 완전히 분리할 수 없으므로 인스턴스화 할 수 있는 코드에 대한 의존성을 가지게 된다.
        - 이를 해결해주기 위한 것이 DI이다.
- DIP(Dependency Inversion Principle)
    - 소프트웨어 모듈들을 분리하는 특정 형식을 지칭
    - 두 가지 원칙
        1. 상위 모듈은 하위 모듈에 의존해서는 안된다. 상위 모듈과 하위 모듈 모두 추상화에 의존해야 한다.
        2. 추상화는 세부 사항에 의존해서는 안된다. 세부사항이 추상화에 의존해야 한다.

- 스프링 컨테이너에 클래스를 등록하면 스프링이 클래스의 인스턴스를 관리해준다.
- 스프링 컨테이너에 Bean을 등록하고 설정하는 방법은 두 가지 존재
    - XML
    - @(어노테이션)

## 스프링 XML 설정

### 스프링 라이브러리 추가
```java
"org.springframework:spring-core:${springVersion}",
"org.springframework:spring-context:${springVersion}",
```

- spring core
    - 모듈들이 공통으로 사용하는 라이브러리
- spring context
    - AnnotationConfigContext, XmlApplicationContext 처럼 접미사가 Context로 되어 있는 요소들이 포함되어 있는 라이브러리
    - spring context는 spring core에 기반한 모듈이므로 이에 대한 의존성을 가지고 있따.
        - 이런 경우를 **의존선 전이**라 함
        - 필요한 라이블러기 자동으로 클래스패스에 추가해 주지만, 명시적으로 필요한 라이브러릴 추가하는 것이 좋다.
- build.gradle의 내용이 많은 경우 `apply from`구문을 사용해서 필요한 내용을 나누어 작성할 수 있다.
    - build.gradle
        ```java
        apply plugin: 'java'
        apply plugin: 'application'

        apply plugin: 'idea'
        apply plugin: 'eclipse'

        apply from : 'gradle/lib.gradle'    // lib.gradle 파일에 라이브러리 추가함

        group = 'thecodinglive'
        version = '1.0.0'

        compileJava {
            sourceCompatibility=1.8
            targetCompatibility=1.8
        }

        compileTestJava {
            sourceCompatibility=1.8
            targetCompatibility=1.8
        }

        compileJava.options.encoding = 'UTF-8'
        compileTestJava.options.encoding = 'UTF-8'
        ```
    - lib.gradle
        ```java
        repositories {
            jcenter()
        }

        ext{
            javaVersion = '1.8'
            springVersion = '4.1.6.RELEASE'
            slf4jVersion = '1.7.5'
            logbackVersion = '1.0.13'
        }

        List springLib = [
                "org.springframework:spring-core:${springVersion}",
                "org.springframework:spring-context:${springVersion}",
        ]

        dependencies {
            compile springLib
            compile     'org.slf4j:slf4j-api:1.7.7'
            testCompile 'junit:junit:4.12'
        }
        ```

### 스프링 applicationContext.xml 설정
```xml
<!-- applicationContext 파일 자체를 위한 내용 -->
<?xml version="1.0" encoding="UTF-8" ?> <!-- XML 선언 -->
<!-- 태그들을 사용하기 위한 xsd 파일에 대한 선언, bean태그를 사용하기 위해서는 spring-beans.xsd 파일이 있어야 함 -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="boss" class="basic.Boss" init-method="onCreated" destroy-method="onDestroyed"></bean>
    <bean id="employee" class="basic.Employee" init-method="onCreated" destroy-method="onDestroyed"></bean>

    <bean id="myWorkService" class="basic.WorkService">
        <property name="workManager">
            <ref bean="boss"/>
        </property>
    </bean>

    <bean id="yourWorkService" class="basic.WorkService">
        <property name="workManager">
            <ref bean="employee"/>
        </property>
    </bean>

</beans>
```

- bean
    - bean 태그 class - 실제 클래스 파일 경로 입력
    - id - 클래스 명의 소문자 형태로 입력
    - 이를 통해서 new 연산자를 이용해서 인스턴스를 생성했던 작업을 스프링에 위임할 수 있다.
- workservice 처럼 다른 클래스 또는 인터페이스를 멤버 변수로 가지고 있는 경우
    - property 태그를 사용해서 명시할 수 있다.
    - workManager처럼 구현 타입이 구체화되지 않은 인터페이스인 경우 ref 태그를 통해 구현체 클래스를 명시할 수 있다.
- 사용
    ```java
    public static void main(String ar[]){
        GenericXmlApplicationContext context = new GenericXmlApplicationContext(
                "classpath:applicationContext.xml"
        );

        WorkService myWorkService = context.getBean("myWorkService", WorkService.class);
        myWorkService.askWork();

        WorkService yourWorkService = context.getBean("yourWorkService", WorkService.class);
        yourWorkService.askWork();

        context.close();
    }
    ```
    - getBean은 XML에 설정한 id값과 클래스명을 입력한다.

### XML 설정 시 빈 생명주기 제어
- init-method 속성
    - 초기 인스턴스가 생성될 때 실행되는 메소드를 기입한다.
- destroy-method 속성
    - 인스턴스가 소멸할 때 실행되는 메소드를 기입한다.

## 스프링 JavaConfig 설정
- 자바 5 이상부터 XML을 사용하지 아놓고 자바 코드만으로 스프링 컨테이너 설정을 할 수 있다.

### @Configuration을 이용한 설정
- Configuration 어노테이션을 클래스 상단에 추가한다.
    - 이 클래스가 빈 설정 정보가 포함된 클래스임을 명시
    - 기존 XML에 사용했던 `<bean>`태그는 @Bean 어노테이션으로 대체할 수 있다.

```java
@Configuration
public class BeanConfig {
    @Bean
    public WorkManager employee() {
        return new Employee();
    }

    @Bean
    public WorkManager boss() {
        return new Boss();
    }

    @Bean
    public WorkService yourWorkService() {
        WorkService workService = new WorkService();
        workService.setWorkManager(employee());
        return workService;
    }

    @Bean
    public WorkService myWorkService() {
        WorkService workService = new WorkService();
        workService.setWorkManager(boss());
        return workService;
    }
}
```

- 기존에 bean id 태그로 등록한 부분은 @Bean어노테이션과 메서드를 이용해서 대체할 수 있다.
- 사용은 기존과 같음

### @Import 어노테이션 사용
- 설정 내용을 파일별로 분리하고 Import를 통해 적용 가능함
- `@Import(CompanyConfig.class)`
- 사용 
    ```java
    @Configuration
    @Import(CompanyConfig.class)
    public class BeanConfig {
        @Bean
        public WorkManager employee() {
            return new Employee();
        }

        @Bean
        public WorkManager boss() {
            return new Boss();
        }

        @Bean
        public WorkService yourWorkService() {
            WorkService workService = new WorkService();
            workService.setWorkManager(employee());
            return workService;
        }

        @Bean
        public WorkService myWorkService() {
            WorkService workService = new WorkService();
            workService.setWorkManager(boss());
            return workService;
        }
    }
    ```

### 어노테이션 설정 시 생명주기 제어
- @PostConstruct
    - 빈이 초기화 될 때 호출
    - 별도로 설정과 클래스를 매핑하지 않고도 사용할 수 있다.
- @PreDestroy
    - 빈이 소멸 시 호출

```java
public class Employee implements WorkManager {
    @Override
    public String doIt() {
        return "do work";
    }

    @PostConstruct
    public void onCreated() {
        System.out.println("employee 초기화");
    }

    @PreDestroy
    public void onDestroyed() {
        System.out.println("employee 소멸");
    }
}
```

