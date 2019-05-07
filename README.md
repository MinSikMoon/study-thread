# study-thread
- 스레드 이용방법이 가물가물해져서 다시 기록해본다.
- 공부책 : 신용권 저 '이것이 자바다' 

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

## 10. 동기화 메소드 / 동기화 블록
- 하나의 스레드가 사용중인 공유객체를 다른 스레드가 사용하지 못하게 하려면 잠금, lock 걸어야 한다.
- **Critical Section (임계영역)** : only 하나의 스레드만 실행 가능한 코드 영역
- 임계영역을 지정하기 위해 자바가 제공하는 2가지? = **동기화 메소드 / 동기화 블록**
- 스레드가 객체 내부의 동기화 메소드 or 블록에 들어가면, 즉시 객체에 잠금을 건다.
- 다른 스레드가 크리티컬 섹션을 실행하지 못하게 한다. 
- 어떻게 하면 되나? => **synchronized** 를 메소드 선언에 붙인다. 
- 인스턴스/ 정적 메소드, 어디든 부착 가능!
````java
public synchronized void method(){
    //critical section // only a thread can do something in this area.
}
````
- 잠금해제는 언제? => 스레드가 동기화 메소드 실행 종료하면 잠금해제.
- 메소드 전체가 아닌, 일부 내용만 임계 영역으로 만들고 싶을때
````java
public void method(){
    //여기서는 여러 스레드가 실행 가능한 zone임.
    synchronized(객체){ //동기화블록 //공유객체가 자기자신이라면 this
        //critical section 
    }
    //다시 여러 스레드가 실행가능한 영역
}
````
### 위의 calculator 예제를 동기화 메소드를 사용해 해결

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

	public synchronized void setCalculator(Calculator calculator) {
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
100, 50으로 정상적으로 나온다.
User 1: 100
User 2: 50
*/
````