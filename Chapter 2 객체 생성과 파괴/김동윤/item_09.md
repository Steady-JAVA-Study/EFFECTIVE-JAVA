
# try-finally 보다는 try-with-resources를 사용하라

- close()를 통해서 직접 닫아줘야 하는 자원 多 (InputStream, OutputStream, java.sql.Connection 등)
- 자원을 제대로 닫힘을 보장하는 try-finally 방식 주로 사용

### try-finally의 문제
- 자원이 1개
```java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine(); //readLine 메서드가 예외던지면
	} finally {
		br.close(); //close도 실패
	}
}
```
- 자원이 2개인 경우
```java

static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
	} finally {
		in.close();
	}
}
```
- 자원이 둘 이상이면 코드 복잡
- 기기의 물리적인 문제가 발생했다면 이로 인해서 readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패 => 두번째 예외가 첫 예외를 먹어버린 것 
- 스택추적 내역에서 첫 번째 예외의 정보는 남지 않게 되어 실제 시스템에서의 디버깅을 어렵


## try-with-resources 사용합시다!
- try-with-resources 구조를 사용하기 위해선 AutoCloseable 인터페이스를 구현
- 꼭 회수해야 하는 자원 다룰 때는 try-with-resources를 사용합시다.
- 문제 진단도 easy하다. 
- try-with-resoureces와 AutoCloseable 을 구현해놓은 클래스를 사용하면 try 문이 종료될 때 자동으로 close를 호출

1) 자원이 하나
```java
static String firstLineOfFile(String path) throws Exception {
	try (BufferedReader br = new BufferedReader (
		new FileReader(path))) {
			return br.readLine();
	}
}
```

2) 자원이 두개 
```java
static void copy(String src, String det) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n == in.read(buf)) > = 0)
			out.write(buf, 0, n);
}
```