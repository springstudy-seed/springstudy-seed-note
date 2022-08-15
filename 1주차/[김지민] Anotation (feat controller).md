# Anotation (feat. controller)

## 1. Anotation이란

- Anotation이란?
    - JAVA에서 사용되는 기능으로, 사전상으로는 주석이라는 의미이지만 사실 그 이상의 의미. 단순 **주석의 기능을 넘어 특별한 기능을 사용**할 수 있음
    - spring에서 해당 클래스의 역할을 정의하기도 하고, Bean을 주입하기도 하며, 자동으로 getter & setter을 생성하기도 함.
- Anotation의 장점
    - 코드량의 감소, 유지보수 용이, 생산성의 증가

## 2. Spring에서의 Anotation

Spring에서는 여러 가지 Annotation을 사용하지만, Bean을 등록하기 위해서는 @Component Annotation을 사용

- **@Component**
    
    ```java
    @Component(value="myman")
    public class Man{
    	public Man(){
    		System.out.println("hi);
    	}
    }
    ```
    
    개발자가 생성한 Class를 Spring의 Bean으로 등록할 때 사용하는 Anotation
    
- **@ComponentScan**
    
    Spring은 @Component, @Service, @Repository, @Controller, @Configuration 중 1개라도 등록된 클래스를 찾으면 Context에 Bean으로 등록 → @ComponentScan Anotation이 있는 클래스의 하위 Bean을 등록될 클래스를 스캔하여 Bean으로 등록해줌
    

- **@Bean**
    
    @Bean Anotation은 개발자가 제어가 불가능한 외부 라이브러리를 Bean으로 만들 때 사용
    

- **@Controller**
    
    Spring에게 해당 Class가 Controller의 역할을 한다고 명시하기 위해 사용하는 Annotation
    ( → 스프링 프레임워크에서는 MVC(Model - View - Controller)패턴을 사용.)
    

- **@Autowired**
    
    Spring에서 Bean 객체를 주입받기 위한 방법 크게 3가지 중 하나로 Bean을 주입받기 위하여 @Autowired 사용. Spring이 Class를 보고 Type에 맞게(Type을 먼저 확인 후, 없으면 Name 확인) Bean을 주입.
    
    <aside>
    💡 Bean 객체를 주입받는 방법
    
    1. @Autowired
    2. 생성자 (@AllArgsConstructor 사용)
    3. setter
    </aside>
    
    ### **@GetMapping**
    
    RequestMapping(Method=RequestMethod.GET)과 같은 역할
    
    ```java
    @Controller                   // 이 Class는 Controller 역할을 합니다
    @RequestMapping("/user")      // 이 Class는 /user로 들어오는 요청을 모두 처리합니다.
    public class UserController {
        @GetMapping("/")
        public String getUser(Model model) {
            //  GET method, /user 요청을 처리
        }
    
        // 위와 아래 메소드는 동일하게 동작
    
        @RequestMapping(method = RequestMethod.GET)
        public String getUser(Model model) {
            //  GET method, /user 요청을 처리
        }
    }
    ```
    
    ### **@Test**
    
    JUnit에서 테스트 할 대상을 표시