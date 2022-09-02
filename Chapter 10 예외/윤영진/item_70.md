# ITEM 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

## Exception의 상속 구조

![image](https://user-images.githubusercontent.com/83503188/187611300-cf2a347c-8e73-436c-bee6-59183c985fb0.png)

 ## Checked / Unchecked Exception

|                      | Checked Exception     | Unchecked Exception  |
|----------------------|-----------------------|:--------------------:|
| 처리 여부                | 명시적인 예외 처리 필요         |  명시적인 처리를 강제하진 않는다.  |
| Transaction Rollback | Yes                   |          No          |
| Ex                   | FileNotFoundException | NullPointerException |


## Checked Exception

호출하는 쪽에서 복구하리라 여겨질 때 사용한다.

```java
public static void main(String[] args) {
    List<String> hello = new ArrayList<>();
    try {
        hello.add("hello");
        
        }catch (NeedRetryException e){
         // retry
        }
    
        }
public List<String> add (String name) throws NeedRetryException {
    throw new NeedRetryException();
        }

class NeedRetryException extends Exception {}
```

## Runtime Exception

```java
@Retryable
@Transactional
public Book saveBook(BookForm bookForm) {
    if(bookCategoryRepository.findById(bookForm.getBookCategoryId()) == null) {
        bookCategoryRepository.save(BookCategory.from(bookForm));
        }
    Book book = Book.from(bookForm);
    return bookRepository.save(book);
}
```

![image](https://user-images.githubusercontent.com/83503188/187614540-5497beb4-eebf-447f-b43d-7cfe8f7df83a.png)

위 코드 중 `bookRepository.save(book);` 에서 문제가 생긴다면 위 스냅샷에서 보면 알 수 있듯이 `IllegalArgumentExcpetion(=RuntimeException)` 이 발생하는 것을 볼 수 있다.

이전에 Book Category가 save가 되고 아래 Book을 save하는 과정에서 문제가 생긴다면 쓸모없는 Book Category 데이터가 생길 수 있으므로 Transaction 단위로 묶어서 오류가 발생한다면 한번에 Rollback해야한다.
