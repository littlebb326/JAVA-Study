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

NIO는 위의 두 가지 포인트를 개선한 I/O 클래스다.

자바의 I/O 속도를 개선하는 방법은 **메모리를 직접 접근**하는 듯, **시스템 콜을 직접 콜하는듯** 하게 사용하는 방법이다.(+ 동기/비동기 제어)

유저 영역 - 일반적인 프로세스들(실행중인 프로그램)이 존재하는 권한이 제한된 영역(하드웨어 직접 접근 불가)
커널 영역 - 운영체제에 존재하는 영역으로 다른 프로세스를 제어할 수 있는 권한 가짐 (하드웨어 직접 접근 가능)
