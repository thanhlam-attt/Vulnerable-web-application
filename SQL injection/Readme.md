# **SQL Injection**

## **SQL Injection là gì?**
***SQL Injection là kỹ thuật lợi edụng lỗ hổng, chèn thêm một đoạn sql để làm sai lệch đi câu truy vấn ban đầu và từ đấy có thể khai thác được dữ liệu trong database***

**Ví dụ:**

- Ta có câu truy vấn như sau:
    - Select * from table where id = ‘$user_id’;
- Khi ta đặt giá trị $user_id = ‘ or 1=1 ⇒ Select * from table where id = ‘’ or 1=1; vì 1=1 luôn đúng ⇒ Câu truy vấn này sẽ trả về hết các giá trị trong bảng table

**Tại sao SQL Injection lại nguy hiểm?:**

- Cơ bản nhất, SQL Injection có thể khai thác dữ liệu trong DB, xem sửa và xóa dữ liệu trong DB
- SQL Injection có thể đọc file trong hệ thống, ghi file đó
- Thậm chí SQL Injection có thể dẫn tới RCE


## **Các loại SQL Injection:**

- **Error SQL:** hacker sẽ thực hiện các hành động làm DB thông báo lỗi, từ thông báo lỗi này để thu thập thông tin về cấu trúc của DB
- **Union SQL:** Sử dụng toán tử UNION SQL để lấy thông tin về DB như tên DB, tên bảng,…
    - Ta có Clause 1 union Clause 2, union sẽ kết hợp kết quả của Clause 1 và Clause 2 và trả về kết quả duy nhất,
    
    → Nếu Clause 1 trả về null ⇒ Truy vấn trên vẫn trả về kết quả Clause 2
    
    - Ví dụ như select name from tables where id = ‘ ’ union select name from tables2
        - Trong câu truy vấn này vế đầu mệnh đề ra null ⇒ Cả câu truy vấn trả về kết quả của câu truy vấn 2 select name from tables2
- **Boolean-based:** Là kỹ thuật tấn công SQL Injection dựa vào việc gửi các truy vấn tới DB bắt buộc ứng dụng trả về các kết quả khác nhau phụ thuộc vào câu truy vấn là True hay False
    - Ví dụ đầu tiên là kiểu Boolean SQL Injection, 1=1 luôn đúng ⇒ trả về tất cả các bản ghi từ bảng table
    - Kiểu tấn công này thường chậm (đặc biệt với cơ sở dữ liệu có kích thước lớn) do người tấn công cần phải liệt kê từng dữ liệu, hoặc mò từng kí tự
- **Time-based:** Hacker gửi một truy vấn SQL tới DB làm cho DB đợi vài giây trước khi có thể hoạt động ⇒ Hacker có thể xem từ thời gian mà DB cần phản hồi suy đoán kết quả truy vấn là True hay False
    - SELECT * FROM products WHERE id=1-IF(MID(VERSION(),1,1) = '5', SLEEP(15), 0)
        - Nếu đúng hoặc hơn 15 giây server mới trả response ⇒ có thể kết luận DB sử dụng là mySQL version 5.x (MySQL hỗ trợ hàm IF())
    - Kiểu tấn công này tốn thời gian k kém gì Boolean-based


## **Cách khai thác lỗ hổng SQL Injection:**

- ***Phát hiện:***
    - Thêm vào câu truy vấn các dấu như dấu nháy kép, dấu nháy đơn, dấu chấm phẩy và các ký tự comment.
    - Nếu thông báo lỗi từ sql xuất hiện → trang web dính SQL Injection
- **Xác định số lượng cột trong mệnh đề Select:**
    - Sử dụng từ khóa Union, vì trong Union đòi hỏi trong mỗi mệnh đề select số lượng các trường đều phải bằng số lượng các trường được select trong mệnh đề select ban đầu
        - Union select 1,2,3,…—
        - Vì Union đòi hỏi kiểu dữ liệu các trường 2 bên phải giống nhau → sử dụng null thay vì 1,2,3,…
            - Union Select null,null,… —
    - Hoặc sử dụng order by. từ khóa order by được dùng để sắp xếp thứ tự cho các bản ghi thu được trong mệnh đề select. Sau order by có thể là tên một cột để xác định kết quả thu về được sắp xếp theo giá trị cột đó. Nếu giá trị sau order lớn hơn số cột được select thì chúng ta thấy thông báo lỗi
    
    
- **Xác định thông tin:**
    - Để biết được tên bảng, tên cột ta sử dụng đối tượng information_schema. Đối tượng này cung cấp các thông tin về tables, columns,… của cơ sở dữ liệu
    - **Cách liệt kê tất cả các table và column trong DB:**
        - ***Liệt kê tất cả các Table:***
            
            SELECT * FROM INFORMATION_SCHEMA.TABLES
            
            Kết quả trả về bao gồm cả danh sách các View nữa. Nếu chỉ muốn lấy danh sách Table thôi thì filter theo TABLE_TYPE
            
            - Select * from INFORMATION_SCHEMA.TABLES where TABLE_TYPE ='BASE TABLE’
        - ***Liệt kê tất cả các Column:***
            
            SELECT * FROM INFORMATION_SCHEMA.COLUMNS
            
- **Lấy bản ghi:**
    - SELECT column1,… FROM Column_name
    - Hoặc có thể dùng group_concat để nối nhiều giá trị với nhau như
    - SELECT GROUP_CONCAT(column1, column2,…) FROM Column_name

## **Cách ngăn chặn SQL Injection:**

- **Sử dụng parameter thay vì cộng chuỗi:** Nếu dữ liệu truyền vào không hợp lệ thì SQL Engine tự động báo lỗi, không cần dùng code check
- **Không hiển thị exception, message lỗi:** Hacker dựa vào message lỗi để tìm ra cấu trúc database → Khi có lỗi chỉ hiện tbao lỗi chứ không hiển thị đầy đủ thông tin về lỗi tránh bị lợi dụng
- **Lọc dữ liệu từ người dùng:** Sử dụng filter lọc các ký tự đặc biệt như (; “ ‘) hay các từ khóa như SELECT, UNION
    - Lọc các ký tự đặc biệt như (; “ ‘) ta có thể bypass nó bằng cách
        - Encoded các ký tự đặc biệt này sang dạng hexa, double encoding
        - We Could Always Try To bypass Addslashes ..with %bf and %af :D, So When We use %bf%27 <%27 là ‘> as Our Input, addslashes() function adds a Slash(%5C) before our Quote(%27) and it becomes %bf%5C%27 and %bf%5C = a Chinese Multibyte Character ? and ThereFore %bf%5C%27 Equals To ?’
    - Lọc các từ khóa. Tuy nhiên, ta có thể bypass các filter này bằng nhiều cách khác nhau. Cụ thể xem trong https://book.hacktricks.xyz/pentesting-web/sql-injection
    
- **Phân quyền trong DB:** Nếu chỉ truy cập dữ liệu từ một số bảng, tạo một account trong DB và gán quyền truy cập cho account đó chứ không dùng account root hay sa ⇒ hacker có inject được sql cũng k đọc được dữ liệu từ các bảng chính, sửa hoặc xóa dữ liệu
