# ITEM 77. 예외를 무시하지 말라 

Catch 블럭에서는 반드시 필요한 일을 하자.

만약 예외를 무시하기로 했다면 catch 블록 안에 명확한 이유를 주석으로 남기고, 예외 변수의 이름도 ignored로 바꿔라

```java
public class FileInputStreamEx {

    private static final File defaultFile = new File("defaultFilePath");

    public static void main(String[] args) {
        FileInputStreamEx fx = new FileInputStreamEx();

        FileInputStream fileInputStream = fx.openFile();
        close(fileInputStream);
    }

    public FileInputStream openFile() {
        String filePath = (new Scanner(System.in)).nextLine();
        File file = new File(filePath);

        try {
            return new FileInputStream(file);
        } catch (FileNotFoundException e) {
            return openFile();
        }
    }

    public static void close(FileInputStream fileInputStream) {
        try {
            fileInputStream.close();
        } catch (IOException ignored) { 
            // 아래의 이유로 예외를 무시한다.
            ignored.printStackTrace(); // 예외가 주기적으로 발생한다면 조사하기 좋게 로그를 남긴다.
        }
    }
}


```

FileInputStream 은 파일을 읽는 스트림이므로, 파일의 상태를 변경하지 않기에 복구할 것이 없다.

close() 한다는 것은 정보는 다 읽었다는 뜻이므로 작업을 중단할 이유도 없다.