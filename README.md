# study-thread
- 스레드 이용방법이 가물가물해져서 다시 기록해본다.
- 신용권 저 '이것이 자바다' 참고

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
- start()하면 파라미터로 받은 runnable의 run()을 대신 프록시 처럼 실행해준다.
````java
thread.start();
````

## 5. Basic 야호 thread 예제
````java

public class BasicMultiThreadTest {

	public static void main(String[] args) {
		Thread task = new Thread(new Runnable() {
			@Override
			public void run() {
				for(int i=0; i<5; i++) {
					System.out.println("호");
					try {
						Thread.sleep(500);
					} catch (Exception e) {}
				}
			}
		});
		
		//task thread : '호'를 외친다.
		task.start();
		
		// main thread : '야' 를 외친다.
		for(int i=0; i<5; i++) {
			System.out.println("야");
			try {
				Thread.sleep(500);
			} catch (Exception e) {}
		}
	}

}

/**
야
호
호
야
호
야
호
야
호
야
/**
````

## 6. Thread 이름
````java
thread.setName('1번');
thread.getName();
Thread curThread = Thread.currentThread();
````

## 7. 자식 클래스로 thread 만들기
````java
public class BasicMultiThreadTest {
	public static void main(String[] args) {
		//main thread 얻기
		Thread mainThread = Thread.currentThread();
		System.out.println("main thread name is " + mainThread.getName());
		//Thread2 준비
		Thread task1 = new Thread(new Runnable() {
			@Override
			public void run() {
				for(int i=0; i<5; i++) {
					System.out.println("호");
					try {
						Thread.sleep(500);
					} catch (Exception e) {}
				}
			}
		});
		
		//task thread : '호'를 외친다.
		task1.start();
		
		//그냥 thread 생성자로 만든거
		ThreadB threadb = new ThreadB();
		System.out.println(threadb.getName() + "start");
		threadb.start();
		
		// main thread : '야' 를 외친다.
		for(int i=0; i<5; i++) {
			System.out.println("야");
			try {
				Thread.sleep(500);
			} catch (Exception e) {}
		}
	}
}

class ThreadB extends Thread{
	public ThreadB() {
		setName("this thread's name is ThreadB");
	}
	
	public void run() {
		for(int i=0; i<5; i++) {
			System.out.println("in " + getName());
			try {
				Thread.sleep(500);
			} catch (Exception e) {}
		}
	}
}

/*
main thread name is main
호
this thread's name is ThreadBstart
야
in this thread's name is ThreadB
호
야
in this thread's name is ThreadB
호
야
in this thread's name is ThreadB
호
야
in this thread's name is ThreadB
호
야
in this thread's name is ThreadB
*/
````

## 8. 스레드 우선순위 
- priority 넣을 수 있는 방식 : 제어가능
- round robin 방식 : jvm이 결정해서 우리가 어찌하지 못함.

## 9. 공유 객체 문제 예제
````java
package ttt;

public class SharingTest {

	public static void main(String[] args) {
		Calculator calculator = new Calculator();

		User1 user1 = new User1();
		user1.setCalculator(calculator);
		user1.start();

		User2 user2 = new User2();
		user2.setCalculator(calculator);
		user2.start();

	}

}

class Calculator {
	private int memory;

	public int getMemory() {
		return memory;
	}

	public void setMemory(int memory) {
		try {
			this.memory = memory;
			Thread.sleep(200);
		} catch (InterruptedException e) {
		} finally {
			System.out.println(Thread.currentThread().getName() + ": " + this.memory);
		}
	}
}

class User1 extends Thread {
	private Calculator calculator;

	public void setCalculator(Calculator calculator) {
		this.setName("User 1");
		this.calculator = calculator;
	}

	public void run() {
		calculator.setMemory(100);
	}
}

class User2 extends Thread {
	private Calculator calculator;

	public void setCalculator(Calculator calculator) {
		this.setName("User 2");
		this.calculator = calculator;
	}

	public void run() {
		calculator.setMemory(50);
	}
}

/*
100, 50이 아니라 최종값이 50이 나와버림
User 1: 50
User 2: 50
*/
````