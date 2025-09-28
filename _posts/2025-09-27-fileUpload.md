---
title: 파일업로드
date: 2025-09-27 10:00:00 +09:00
categories: [Java, Servlet]
tags: [ Java, Servlet ]
---

## 1. 파일업로드 - http

### 1. 폼(form)과 HTTP
 - HTML <form>은 사용자 입력 데이터를 서버로 전송하는 표준 방법
 - 전송 방식(method)
    - GET → URL 쿼리 스트링으로 전송, 짧은 데이터/검색용
    - POST → 요청 본문(body)에 데이터 전송, 대용량/보안 데이터 업로드에 적합

### 2. http 예시
 ```
 // 파일 업로드는 반드시 enctype="multipart/form-data"로 지정해야 함.
 // Post로만 사용
 <form action="upload.do" method="post" enctype="multipart/form-data">
     이름: <input type="text" name="username"><br>
     파일: <input type="file" name="uploadFile"><br> //타입을 파일로 설정
     <input type="submit" value="업로드">
 </form>
 ```

## 2. COS.jar 방식
### 1. COS.jar
 - cos.jar의 정확한 명칭은 "com.oreilly.servlet"로, 이는 O'Reilly Servlet Package의 일부
 - COS는 JSP(Java Server Pages) 환경에서 파일 업로드 기능을 구현하기 위해 사용되는 자바 라이브러리
 - 이 라이브러리는 multipart/form-data 형식으로 전송된 파일 데이터를 처리하는 데 특화되어 있음.

### 2. 사용
```
    String saveDir = "C:/upload"; // 업로드 저장 경로
    int maxSize = 10 * 1024 * 1024; // 크기 10MB 
    String encoding = "UTF-8";

    MultipartRequest multi = new MultipartRequest(
        request,                // 원래 HttpServletRequest
        saveDir,                // 저장 디렉토리
        maxSize,                // 업로드 제한 크기
        encoding,               // 인코딩
        new DefaultFileRenamePolicy() // 중복 파일명 처리 정책
    );

    // 파일명 추출
    String fileName = multi.getFilesystemName("uploadFile");
    String originalName = multi.getOriginalFileName("uploadFile");

}
```

## 3. Servlet 3.0 이후버전
### 1. @MultipartConfig 방식
 - Java EE 6 (2009)에 포함되면서 표준 API로 파일 업로드 지원
 - 특징
    - @MultipartConfig 애노테이션으로 서블릿에 설정 가능
    - request.getPart("file") 또는 request.getParts()로 파일/폼 데이터를 쉽게 가져올 수 있음
    - 임시 디렉터리, 최대 파일 크기 등 업로드 제약 조건도 설정 가능
    - 별도의 외부 라이브러리 없이 서블릿 자체로 multipart 처리 가능

### 2. 사용
```
@WebServlet("/upload")
@MultipartConfig( // 기존에 
    fileSizeThreshold = 1024 * 1024,  // 1MB
    maxFileSize = 1024 * 1024 * 10,   // 10MB
    maxRequestSize = 1024 * 1024 * 50 // 50MB
)
public class FileUploadServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        Part filePart = request.getPart("file");
        String fileName = Path.of(filePart.getSubmittedFileName()).getFileName().toString();
        filePart.write("/upload/" + fileName);
        response.getWriter().println("Upload complete: " + fileName);
    }
}
```

## 4. COS.jar -> selrvet 3.0
### 1. 변화
 - 변수가 코드에서 직접 지정 → 애노테이션으로 선언만 하면 Servlet이 알아서 처리로 바뀐 것
 
| 구분     | cos.jar           | Servlet 3.0 이후 |
| ------ |-------------------| -------------- |
| 설정 위치  | 코드 안 / web.xml    | 애노테이션          |
| 라이브러리  | 외부 필요 (cos.jar 등) | 표준 API 내장      |
| 코드 복잡도 | 상대적으로 길고 반복적      | 간결, 자동 처리      |
| 제어 수준  | 세밀하게 조정 가능        | 애노테이션 속성 범위 내  |

### 2. 장단점

| 구분            | cos.jar                                       | Servlet 3.0 이후 (@MultipartConfig)          |
| ------------- | --------------------------------------------- | ------------------------------------------ |
| 설정 방식     | 코드 안에서 MultipartRequest 객체 생성, 파일 경로·크기 직접 지정 | @MultipartConfig 애노테이션으로 서블릿 위에서 설정, 자동 처리 |
| 라이브러리 필요성 | 외부 라이브러리 필요                                   | 서블릿 표준 API 내장, 별도 라이브러리 불필요                |
| 코드 복잡도    | 상대적으로 길고 반복적                                  | 간결, request.getPart()로 자동 파싱               |
| 파일 처리 제어  | 세밀하게 가능 (파일 이름, 경로, 스트림 직접 처리)                | 애노테이션 속성 범위 내에서만 제어 가능                     |
| 호환성       | Servlet 3.0 이전 환경에서도 사용 가능                    | Servlet 3.0 이상 필요                          |
| 장점        | - 구형 서버에서도 사용 가능<br>- 세밀한 커스터마이징 가능           | - 코드 간결<br>- 외부 라이브러리 불필요<br>- 표준 API로 안정적 |
| 단점        | - 코드가 장황<br>- 예외 처리 및 스트림 관리 복잡<br>- 유지보수 어려움 | - 세밀한 커스터마이징은 제한적<br>- 구형 서버에서는 사용 불가      |

