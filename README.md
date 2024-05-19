# Hilt
[Android] Android DI Hilt 라이브러리
[DI Hilt](https://velog.io/@haanbink/Android-Hilt-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
[Hilt 안드로이드 공식 개발문서](https://developer.android.com/codelabs/android-hilt?hl=ko#0)

 ## DI 개념 & 라이브러리 없이 직접 구현해보기

## Dependency Injection

어떤 클래스는 다른 클래스에 대한 참조가 필요한 경우가 있다.    
예를 들어, Car 클래스는 Engine 클래스 참조가 필요할 것이다.     
이 때 Car 가 Engine 에 의존하고 있다고 말하고, Engine 을 Car 의 종속 항목 (디펜던시) 이라고 한다.     

특정 클래스가 자신이 의존하고 있는 객체를 얻는 방법은 3가지가 있다. (Car 와 Engine 예제 활용)      

1. Car 클래스 안에서 Engine 인스턴스를 생성하여 초기화한다.     
2. 다른 곳에서 객체를 가져온다. Android 로 치면 Context, getSystemService() 등에 해당한다.      
3. 객체를 파라미터로 제공받는다. Car 의 생성자가 Engine 을 파라미터로 받는다.      

세 번째 방법이 바로 Dependency Injection 기법 중 하나이다.     

## 의존관계에 있어 DI 를 사용하지 않을 때
DI 없이 코드에서 자체적으로 Engine 을 생성하는 Car 를 나타낸 모습이다.
```kotlin
class Car {

    private val engine = Engine()

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.start()
}
```
이 코드는 다음과 같은 문제를 갖고 있다.

- Car 의 Engine 에 대한 의존성이 너무 강하다. Car 클래스가 Engine 을 직접 인스턴스화하기    
때문에, Engine 의 서브클래스인 GasEngine, ElectricEngine 등을 사용할 수 없게 된다.      
- 또한, Engine 의 생성자가 변경된 경우 Car 클래스에서도 수정이 이루어져야 한다.        
이러한 강력한 의존관계는 테스트를 어렵게 만든다. Engine 의 실제 인스턴스를 사용하기 때문에 다양한 시나리오를 고려하지 못한다. (Unit Test 에 불리함)     

## 의존관계에 있어 DI 를 사용할 때
DI 를 사용한다면 Car 의 각 인스턴스는 초기화할 때 Engine 객체를 생성자 파라미터로 받게 된다.
```kotlin
class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val engine = Engine()
    val car = Car(engine)
    car.start()
}
```
main() 에서 Engine 인스턴스를 생성하고, 이를 활용하여 Car 인스턴스를 만들게 된다.

이렇게 구현하게 되면 다음과 같은 이점을 챙길 수 있다.

- Car 의 재사용성이 높아진다. 예를 들어 ElectricEngine 과 같은 Engine 의 서브클래스를 넘겨주는 등, Engine 의 다양한 구현을 Car 에 전달할 수 있다.
- Engine 의 생성자 등 구현이 변경되어도, Car 클래스를 수정하지 않아도 된다.
- Car 에 대한 유닛 테스트가 편리해진다. 즉, 다양한 시나리오를 테스트해볼 수 있다. (MockEngine 등)

## 안드로이드에서의 DI 구현 방법
안드로이드에서 DI 를 구현하는 방법은 크게 두 가지가 있다.

- Constructor Injection (생성자 삽입) : 위에서 설명한 방법대로, 생성자 파라미터를 통해 의존성을 주입해주는 것이다.
- Field Injection (필드 삽입) : Activity 나 Fragment 는 시스템이 인스턴스화하기 때문에 생성자 삽입 기법이 불가능하다.
따라서 다음과 같이 필드 삽입을 사용할 수 있다.
```kotlin
class Car {
    lateinit var engine: Engine

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.engine = Engine()
    car.start()
}
```
## DI 의 이점
1. 의존성 분리
클래스가 더이상 디펜던시의 생성 (인스턴스화) 에 관여하지 않기 때문에, 종속 항목이 변경되어도     
(생성자 변경 등) 영향을 받지 않고 유연하게 동작한다. 즉, 리팩토링이 편리해진 것이다.     

2. 클래스 재사용성 증가
의존하는 객체의 구현을 쉽게 갈아끼울 수 있다. 서브 타입 등 다양한 구현을 수용할 수 있고,    
때문에 다양한 곳에서 클래스를 재사용할 수 있다.    

3. 테스트 편의성
의존성이 분리되어, 테스트 시 다양한 구현을 전달하여 여러 시나리오를 검증해볼 수 있다.     
(즉, Mocking 이 쉬워 진다 : Test Double 이 가능해졌다)    

## 직접 DI 구현해보기

안드로이드 개발자들이 주로 사용하는 Dagger2, Hilt 와 같은 라이브러리들이 있지만,      
DI 의 원리를 이해하기 위해서는 우선 직접 구현해보는 편이 낫다.   
예시로 로그인 플로우를 구현함에 있어 DI 를 직접 구현해보자. 디펜던시 그래프는 다음과 같다.        
   
![image](https://github.com/chihyeonwon/Hilt/assets/58906858/93172753-b065-44d0-841c-57da136d7e15)    
이 플로우에 있어 Repository 및 DataSource 클래스는 다음과 같다.      
```kotlin  
class UserRepository(
        private val localDataSource: UserLocalDataSource,
        private val remoteDataSource: UserRemoteDataSource
) { ... }

class UserLocalDataSource { ... }
class UserRemoteDataSource(
    private val loginService: LoginRetrofitService
) { ... }
```
LoginActivity 는 아래와 같다.
```kotlin
class LoginActivity: Activity() {

    private lateinit var loginViewModel: LoginViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // In order to satisfy the dependencies of LoginViewModel, you have to also
        // satisfy the dependencies of all of its dependencies recursively.
        // First, create retrofit which is the dependency of UserRemoteDataSource
        val retrofit = Retrofit.Builder()
            .baseUrl("https://example.com")
            .build()
            .create(LoginService::class.java)

        // Then, satisfy the dependencies of UserRepository
        val remoteDataSource = UserRemoteDataSource(retrofit)
        val localDataSource = UserLocalDataSource()

        // Now you can create an instance of UserRepository that LoginViewModel needs
        val userRepository = UserRepository(localDataSource, remoteDataSource)

        // Lastly, create an instance of LoginViewModel with userRepository
        loginViewModel = LoginViewModel(userRepository)
    }
}
```
위 코드들을 놓고봤을 때, 아래와 같은 문제들을 발견할 수 있다.        

1. 보일러플레이트가 너무 많다. 다른 부분에서 LoginViewModel 의 다른 인스턴스를 만들려면     
중복된 코드가 발생할 수 있다.     

2. 객체를 재사용하기 어렵다. 여러 군데에서 UserRepository 를 재사용하려면 싱글톤 패턴을 따르게 해야    
한다. 그런데 만약 싱글톤으로 구현한다해도, 모든 테스트가 동일한 인스턴스를 공유하므로 다양한 시나리오의 테스트가 어려워지게 된다.     

## Container 로 Dependency 관리
객체 재사용 문제를 해결하려면, 디펜던시를 가져오기 위해 사용할 자체적인 'Dependency Container' 클래스를 만들면 된다.     
이 컨테이너에서 제공하는 인스턴스는 외부로 공개될 수 있다. 지금 예시에서는 UserRepository 인스턴스만 있으면 되므로 얘만 public 상태로 둔다.    

```kotlin
// Container of objects shared across the whole app
class AppContainer {

    // Since you want to expose userRepository out of the container, you need to satisfy
    // its dependencies as you did before
    private val retrofit = Retrofit.Builder()
	                            .baseUrl("https://example.com")
	                            .build()
	                            .create(LoginService::class.java)

    private val remoteDataSource = UserRemoteDataSource(retrofit)
    private val localDataSource = UserLocalDataSource()

    // userRepository is not private; it'll be exposed
    val userRepository = UserRepository(localDataSource, remoteDataSource)
}
```

이러한 디펜던시는 앱 전체에 걸쳐 사용될 수 있으므로 모든 액티비티에서 사용할 수 있는, 즉 Application 클래스에 배치해야 한다.    
그러므로 AppContainer 인스턴스를 갖고 있는 Application 클래스를 만들자.    
```
// Custom Application class that needs to be specified
// in the AndroidManifest.xml file
class MyApplication : Application() {

    // Instance of AppContainer that will be used by all the Activities of the app
    val appContainer = AppContainer()
}
```
이젠 액티비티에서도 해당 클래스를 가지고 AppContainer 인스턴스를 가져와서 UserRepository 인스턴스를 얻을 수 있다.     
```
class LoginActivity: Activity() {

    private lateinit var loginViewModel: LoginViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Gets userRepository from the instance of AppContainer in Application
        val appContainer = (application as MyApplication).appContainer
        loginViewModel = LoginViewModel(appContainer.userRepository)
    }
}
```  
싱글톤으로 구현하지 않고, 모든 액티비티에게 공유되는 AppContainer 를 통해 UserRepository 를 필요로 하는 모든 액티비티에서 인스턴스를 제공할 수 있게 됐다.    

만약 LoginViewModel 도 다른 곳에서 재사용되는 경우, LoginViewModel 인스턴스를 만들어주는 곳 역시    
있으면 좋다. 마찬가지로 이를 컨테이너로 옮기고, 새 LoginViewModel 객체를 생성하는 팩토리를 만들어주자.     
```kotlin
// Definition of a Factory interface with a function to create objects of a type
interface Factory<T> {
    fun create(): T
}

// Factory for LoginViewModel.
// Since LoginViewModel depends on UserRepository, in order to create instances of
// LoginViewModel, you need an instance of UserRepository that you pass as a parameter.
class LoginViewModelFactory(private val userRepository: UserRepository) : Factory {
    override fun create(): LoginViewModel {
        return LoginViewModel(userRepository)
    }
}
```
LoginViewModelFactory 를 AppContainer 로 옮겨주고, 이를 LoginActivity 에서 사용해보자.
```kotlin
// AppContainer can now provide instances of LoginViewModel with LoginViewModelFactory
class AppContainer {
    ...
    val userRepository = UserRepository(localDataSource, remoteDataSource)

    val loginViewModelFactory = LoginViewModelFactory(userRepository)
}
```
```kotlin
class LoginActivity: Activity() {

    private lateinit var loginViewModel: LoginViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Gets LoginViewModelFactory from the application instance of AppContainer
        // to create a new LoginViewModel instance
        val appContainer = (application as MyApplication).appContainer
        loginViewModel = appContainer.loginViewModelFactory.create()
    }
}
```
재사용성을 높였지만, 여전히 다음과 같은 문제들이 남아있다.     

- AppContainer 를 직접 관리하기 때문에 모든 디펜던시의 인스턴스를 수동으로 만들어줘야 한다.      
- 여전히 보일러플레이트 코드가 많다. 객체를 다른 곳에서 재사용할지에 따라     
팩토리, 파라미터 등을 만들어줘야 한다.     

그러나, 앱이 커지면 커질수록 Container, Factory 등 보일러플레이트코드를 많이 작성하게 되고,       
그러한 곳들에서 오류가 발생하기 쉽다. 그리고 컨테이너가 더이상 필요하지 않을 때 메모리에서 삭제하는 등 컨테이너의 수명 주기를 직접 관리해야한다.
만일 이러한 곳에서 실수한다면 자잘한 버그와 메모리 릭이 발생할 수 있다.      

Dagger, Hilt 와 같은 DI 라이브러리들은 이러한 고충을 덜어준다. 지금까지의 과정들을 자동화해준다.     
