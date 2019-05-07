# study-thread
- 스레드 이용방법이 가물가물해져서 다시 기록해본다.

## 1. 작업스레드
- 메인스레드가 있고, 진짜 작업을 할 작업스레드가 있다.

## 2. Runnable 
- run() 메소드 하나만 들어있다. 
- 인터페이스다. 그래서 구현객체를 만들어 대입해준다. 
## 3. Thread 생성
- runnable 인터페이스를 구현한 객체를 파라미터로 넣고 생성자 호출한다. 

````java
Thread thread = new Thread(Runnable somthingRunnable);
````
## 

````java
class Task implements Runnable {
    public void run(){
        //스레드에서 실행할 코드
    }
}
````

````java
Runnable task = new Task();

Thread thread = new Thread(task);
````
````java
// 근데 보통 익명객체로 많이 쓰지.
Thread thread = new Thread(
    new Runnable(){
        public void run(){
            //somthing your code for work;
        }
    }
);
````

## 4. Thread 실행
- Thread는 그냥 실행되지 않는다!
- start() 메소드 써야한다.
````java
thread.start();
````
