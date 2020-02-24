JAVA IO
==================

# 스트림이란?
* 데이터를 운반하는데 사용되는 연결통로
* 연속적인 데이터의 흐름
* 단방향성, FIFO


![stream](./image/stream.png)


# 바이트 단위 입출력 스트림

* InputStream / OutputStream
* 바이트 단위로 데이터 전송
* **File**InputStream, **File**OutputStream
* **ByteArray**InputStream, **ByteArray**OutputStream

# 문자 단위 입출력 스트림

* Reader / Writer
* 문자 단위로 데이터 전송
* **File**Reader, **File**Writer
* **Buffered**Reader, **Buffered** Writer

# 자바 입출력 상속 관계도

![tree](./image/IO_tree.PNG)


# 바이트 단위 입출력 예제(1)

<pre>
<code>
package exIO;

import java.io.*;

public class Ex1 {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		FileInputStream fis = null; 
        FileOutputStream fos = null;        
        try {
            fis = new FileInputStream("input.txt"); // 파일로부터 읽어오기 위한 객체
            fos = new FileOutputStream("output.txt"); // 파일에 쓸 수 있게 해주는 객체

            int readData = -1; // read() 메소드가 끝을 나타낼 때 -1을 return하므로 int형으로 선언
            while((readData = fis.read())!= -1){ // 1 byte씩 읽고
                fos.write(readData); // 1 byte씩 쓴다
            }           
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }finally{
            try {
                fos.close(); // OutputStream 닫기
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            try {
                fis.close(); // InputStream 닫기
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
	}

}

</code>
</pre>

![result1](./image/ex1.PNG)

# 바이트 단위 입출력 예제(2)

<pre>
<code>
package exIO;

import java.io.*;

public class Ex1 {

	public static void main(String[] args) {
		
		long startTime = System.currentTimeMillis();        
        FileInputStream fis = null; 
        FileOutputStream fos = null;        
        try {
            fis = new FileInputStream("input.txt");
            fos = new FileOutputStream("output.txt");

            int readCount = -1; 
            byte[] buffer = new byte[512]; //512 byte 크기의 바이트형 배열 생성
            while((readCount = fis.read(buffer))!= -1){
                fos.write(buffer,0,readCount);
            }
            
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }finally{
            try {
                fos.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            try {
                fis.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    //메소드가 끝났을때 시간을 구하기 위함. 
    long endTime = System.currentTimeMillis();
    //메소드를 수행하는데 걸린 시간을 구할 수 있음. 
    System.out.println(endTime-startTime);
	}

}
</code>
</pre>

# 문자 단위 입출력 예제(1)
<pre>
<code>
package exIO;

import java.io.*;

public class Ex1 {

	public static void main(String[] args) {
		
		BufferedReader br = null;
		PrintWriter pw = null;
		
		
		
		try {
			br = new BufferedReader(new FileReader("input.txt"));
			//System.in은 InputStream 타입이므로 BufferedReader의 생성자에 바로 들어갈 수 없다.
			pw = new PrintWriter(new FileWriter("output.txt"));
			
			String line = null;
			
			while((line = br.readLine()) != null) {
				pw.println(line);
			}
		} catch(Exception e) {
			e.printStackTrace();
		} finally {
			pw.close();
			try {
				br.close();
			} catch(IOException e) {
				e.printStackTrace();
			}
		}
		
	}
}
</code>
</pre>

![result2](./image/ex2.PNG)


# 문자 단위 입출력 예제(2)
<pre>
<code>
package exIO;

import java.io.*;

public class Ex1 {

	public static void main(String[] args) throws IOException {
		
		InputStreamReader isr = null;
		OutputStreamWriter osw = null;
		
		isr = new InputStreamReader(System.in);
		osw = new OutputStreamWriter(System.out);
		
		try {
			int i = 0;
			
			while((i = isr.read()) != -1)
			{
				osw.write(i);
				osw.flush();
			}
		} catch(Exception e){
			e.printStackTrace();
		}
		
		finally {
			try {
				osw.close();
				isr.close();
			} catch(Exception e) {
				e.printStackTrace();
			}
		}
		
	}
}
</code>
</pre>

![result3](./image/ex3.PNG)

JAVA NIO
===============

## 자바는 어디가, 왜 느릴까?

자바는 C/C++과 달리 포인터로 **_1. 직접 메모리를 관리_** 하고 운영체제 수준의 **_2. 시스템 콜을 직접 사용_** 할 수 없다.
(JVM이라는 프로세스 위에서 동작하는 추상화된 비교적 고수준의 언어이기 때문)

NIO : 위의 두 가지 포인트를 개선한 I/O 클래스.

자바의 I/O 속도를 개선하는 방법 : **메모리를 직접 접근**하는 듯, **시스템 콜을 직접 콜하는듯** 하게 사용하는 방법.(+ 동기/비동기 제어)

![NIO1](./image/NIO1.png)

유저 영역 - 일반적인 프로세스들(실행중인 프로그램)이 존재하는 권한이 제한된 영역(하드웨어 직접 접근 불가)
커널 영역 - 운영체제에 존재하는 영역으로 다른 프로세스를 제어할 수 있는 권한 가짐 (하드웨어 직접 접근 가능)

### 자바 I/O 프로세스

1. 프로세스가 커널에 파일 읽기 명령을 내림
2. 커널은 시스템콜(read())를 사용해 디스크 컨트롤러가 물리적 디스크로부터 읽어온 파일 데이터를 커널 영역안의 버퍼에 씀.
3. 모든 파일 데이터가 버퍼에 복사되면 다시 프로세스안의 버퍼로 복사.
4. 프로세스 안의 버퍼의 내용으로 프로그래밍.

여기서 커널 영역의 버퍼를 **직접**접근할 수 있다면 3번의 비효율적인 복사과정을 건너뛸 수 있음.
(2번 과정은 DMA(Direct Memory Access) 기능으로 CPU의 도움이 필요 없지만, 3번 과정은 CPU의 도움이 필요해 CPU 낭비가 발생)

또한, 위의 IO 프로세스를 거치는 동안 작업을 요청한 쓰레드는 블록킹된다는 문제점이 존재.
NIO는 이러한 문제도 해결할 수 있음.

### NIO의 키워드 3가지 (버퍼, 채널, 셀렉터)
1. 자바의 포인터 버퍼(NIO에서 제공하는 Buffer 클래스)
 - 커널에 의해 관리되는 시스템 메모리를 직접 사용할 수 있는 Buffer 클래스
2. 채널(channel)
 - 스트림 : 단방향식으로 읽기와 쓰기 중 하나만 사용 가능.
 - 채널 : 양방향식으로 읽기와 쓰기 모두 가능. 또한, 네이티브 IO, Scatter/gather 구현으로 효율적인 IO처리(시스템 콜의 수 줄이기, 모아서 처리하기)
3. 셀렉터(Selector)
 - 네트워크 프로그래밍의 효율을 높이기 위한 것.
 - 클라이언트 하나당 쓰레드 하나를 생성해서 처리하기 때문에 쓰레드가 많이 생성될 수록 성능이 급격히 저하되는 단점을 개선하는 Reactor 패턴의 구현체.
 
### NIO에서 사용하는 I/O 향상을 위한 운영체제 수준의 기술
1. 버퍼
- 데이터를 한개씩 여러번 반복적으로 전달하는 것 보다 중간에 버퍼를 두고 모아서 한 번에 전달하는 것이 훨씬 효율적.

2. Scatter/Gather
- JAVA 안에서 여러 개의 버퍼를 만들어 사용할 때 동시에 각각 버퍼에 데이터를 쓰거나 읽는 경우 시스템 콜을 여러번 불러서 읽기 쓰기를 수행.
- 시스템 콜을 호출하는 것은 가벼운 작업이 아니므로 비효율적.
- Scatter와 Gather는 프로세스에서 사용하는 버퍼 목록을 한 번에 넘겨줌.
- 운영체제에서는 최적화된 로직에 따라 버퍼들로부터 순차적으로 데이터를 읽고 쓸수 있음.

3. 가상 메모리
- 프로그램이 사용할 수 있는 주소 공간을 늘리기 위해 운영체제에서 지원하는 기술.
- 실제 프로그램이 실행되는 데 지금 필요한 페이지의 가상 주소만 물리 메모리에 넣어 놓는 것.
- 실제 메모리 크기보다 큰 가상 메모리 공간을 사용할 수 있음.
- 여러 개의 가상 주소가 하나의 물리적 메모리를 참조함으로써 메모리를 효율적으로 사용 가능.

![NIO2](./image/NIO2.png)
