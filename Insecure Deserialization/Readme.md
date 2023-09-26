# Insecure Deserlialization

## **Serialization là gì?**

- Serialization là quá trình chuyển đổi các cấu trúc dữ liệu phức tạp ví dụ như các đối tượng (Objects) thành các luồng bytes (stream of bytes), các luồng bytes này sẽ được gửi đến máy chủ và máy chủ đó sẽ chuyển đổi lại nó thành đối tượng (Deserialization). Serialization chuyển đổi objects thành các bytes stream để:
    - Viết dữ liệu phức tạp vào bộ nhớ tiến trình, một file hoặc một database
    - Gửi các dữ liệu phức tạp như, gửi qua mạng, giữa 2 thành phần trong một ứng dụng hoặc trong một lời gọi API
    - Ngược lại với Serialization là Deserialization
- Ví dụ về Serialization và Deserialization:
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/293bca11-80f2-400a-b9d0-6e5f9aa4e135/Untitled.jpeg)
    

- Chú ý rằng: tất cả các thuộc tính ban đầu của đối tượng sẽ được lưu trữ trong data steam được serialize kể cả các trường private → Để ngăn các trường đó bị serialize thì chúng phải được đánh dấu là “transient” trong khi khai báo

## Rủi ro của deserialization

- Rủi ro của deserialization xuất hiện khi các dữ liệu có thể được điều khiển bởi người dùng được deserialize bởi website → Nó cho phép attacker có thể thực hiện serialize các đối tượng để truyền các dữ liệu độc hại vào trong code của ứng dụng
- Nó thậm chí còn có thể thay thể một object được serialize bằng một object thuộc một lớp thực thể khác. Điều đáng nói là các đối tượng của bất kỳ class nào mà có ở trên website đều sẽ được deserialize và khởi tạo → vì lý do đó nên insecure deserialization đôi khi còn được gọi là lỗ hổng “object injection”
- Một đối tượng của một lớp không hợp lệ có thể gây ra một exception. Tuy nhiên, tác hại của nó đã được thực hiện. Rất nhiều cuộc tấn công dựa trên deserialization được hoàn tất trước khi quá trình deserialization được hoàn thành ngay cả khi các chức năng của chính website đó không được tương tác trực tiếp với object độc hại.

## Các lỗ hổng insecure deserialization vulnerabilities phát sinh như nào

- Hầu hết các phát sinh tiêu biểu của insecure deserialization là do sự thiếu nhận thức về sự deseralize user-controllable data có thể nguy hiểm như nào. Tốt hơn hết, Các user input không nên được deserizalize
- Tuy nhiên, đôi khi bản thân website nghĩ nó an toàn vì họ đã triển khai một vài hình thức kiểm tra bổ sung trên các dữ liệu được deserialize. Cách này có thể không hiệu quả vì chúng hầu hết không thể triển khai validation và sanitization để giải quyết mọi tình huống. Các cách kiểm tra này cũng có thể có các lỗ hổng cơ bản như khi dựa trên kiểm tra data khi data ấy đã được deserialize → như đã nói ở trên quá muộn để ngăn chặn cuộc tấn công
- Các lỗ hổng cũng có thể xảy ra vì các đổi tượng được deserialize thường được cho là đáng tin cậy. Đặc biệt khi sử dụng các ngôn ngữ với định dạng serialization là nhị phân → Các lập trình viên có thể nghĩ users không thể độc hoặc thực thi dữ liệu. Tuy nhiên, nó vừa có thể thực hiện một cuộc tấn công khai thác binary serialized objects vừa có thể khai thác string-based formats
- Các cuộc tấn công Deserialization-based cũng có thể được tạo ra dựa vào số dependencies tồn tại trong các websites. Các website tiêu biểu có thể triển khai nhiều các thư viện khác nhau, mỗi chúng đều có các dependencies riêng → Điều này tạo ra một lượng lớn các class và các method mà khó có thể quản lý một cách an toàn → Attacker có thể tạo một instances của bất cứ class nào trong đó → Rất khó để đoán được methods nào có thể liên quan tới các data độc hại

## Impact của insecure deserizalization

- Có thể dẫn tới một số lỗ hổng như sử dụng lại mã tồn tại trong ứng dụng một cách không an toàn, đặc biệt là RCE, Privilege escalation, truy cập file tùy ý hoặc là dẫn tới DOS

## Cách khai thác các lỗ hổng insecure deserialization

### Xác định lỗ hổng

- **Định dạng PHP serialization**
    - PHP sử dụng hầu hết các định dạng chuỗi mà con người có thể đọc được, với chữ cái biểu diễn cho kiểu dữ liệu và các số đại diện cho độ dài mỗi mục
        - Ví dụ như:
            
            ```
            $user->name = "carlos";
            $user->isLoggedIn = true;
            
            Sau khi được serialization sẽ là:
            O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
            ```
            
    - Giải thích thêm về các format trong định dạng serialization
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/dfc40f47-649d-4492-ba43-90c738cef269/Untitled.webp)
    
    - Các methods có sẵn trong PHP để phục vụ cho việc serialization là serialize() và unserialize() → Nếu có quyền truy cập vào source code ⇒ Có thể tìm 2 method trên tại bất cứ đâu
- **Định dạng Java serialization**
    - Java và 1 số ngôn ngữ khác thường sử dụng deserialization dạng nhị phân → Tìm bất cứ class nào triển khai interface java.io.Serializable → Nếu có quyền truy cập vào source code, tìm tất cả methods sử dụng readObject() - method này được sử dụng để đọc object từ InputStream

### Khai thác

- Để khai thác được lỗ hổng này trên nền tảng PHP ta cần 2 điều kiện sau:
    - Đối tượng cần tấn công phải có lớp sử dụng Magic Method
    - Tìm được POP chain, hay chính là có thể tùy chỉnh được các đoạn code trong quá trình hàm unserialize() được gọi
- **Thực hiện serialized objects**
    - Sửa đổi các thuộc tính Object
        - Ví dụ ta có một object được serialized như sau:
            
            `O:4:"User":2:{s:8:"username";s:6:"carlos";s:7:"isAdmin";b:0;}`
            
            - Và ta có cách deserialization bên phía server như sau:
                
                ```
                $user = unserialize($_COOKIE);
                if ($user->isAdmin === true) {
                // allow access to admin interface
                }
                ```
                
        - Có thể thấy rằng attacker có thể thay đổi giá trị boolean của isAdmin thành 1. re-encode object và ghi đè chúng lên phiên cookie hiện tại với các giá trị đã được sửa đổi → lỗ hổng này có thể khởi tạo một object dựa trên data từ cookie, bao gồm cả các trường được sửa đổi → dẫn tới privilege escalation
    - Chỉnh sửa các kiểu dữ liệu
        - Giả sử như phía server xử lý object được deserlia như sauL
            
            ```
            $login = unserialize($_COOKIE)
            if ($login['password'] == $password) {
            // log in successfully
            }
            ```
            
            - Ta có thể thấy trong đoạn PHP kia sử dụng == thay vì === ⇒ Ta có thể khai thác lỗ hổng Type juggling trong PHP cụ thể như sau:
                - Khi sử dụng == → PHP sẽ convert 2 vế thành cùng 1 kiểu dữ liệu và so sánh chúng với nhau hay có nghĩa 5 == “5” (True) hoặc 0 == “Bất cứ String nào không bắt đầu bằng số” (True)
- **Sử dụng chức năng ứng dụng**
    - Ngoài check các giá trị thuộc tính thì các chức năng của một website cũng có thể thực hiện các hành động nguy hiểm trên dữ liệu từ một object deserialized.
    - Giả sử như trên website tồn tại một chức năng cho phép người dùng xóa ảnh bằng cách truy cập vào địa chỉ của image như sau: `$user->image_location`
        - Nếu $user được tạo từ một serialized object → attacker có thể chỉnh sửa giá trị image_location thành một đường dẫn file tùy ý ⇒ Có thể xóa file tùy ý
- **Các Magic methods**
    - Các method này là các method tự động được gọi khi một event hoặc một scenario xảy ra mà không cần phải gọi trực tiếp. Ví dụ như phương thức khởi tạo,…
    - Các method này thường được chỉ ra bởi tiền tố hoặc bao quanh method là 2 dấu gạch dưới “__” (double_underscores) Ví dụ như **“__construct()”** hoặc **“__init__”.**  Thông thường, các method đặc biệt này được dùng để khởi tạo các thuộc tính của một instance → Tuy nhiên, các method đặc biệt này có thể được tùy chỉnh bởi developer để thực thi bất kỳ đoạn code nào mà họ muốn
    - Các magic method thường gặp là:
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/f326cb05-75aa-4f8a-bc52-67f4554bd783/Untitled.webp)
        
    - Các method này được sử dụng rộng rãi và bản thân nó không chứa lỗ hổng. Nhưng nó trở nên nguy hiểm nếu đoạn code mà được thực thi xử lý các dữ liệu có thể được điều khiển bởi attacker → khi một object được deserialize, nó có thể được khai thác bởi attacker để tự động gọi tới các phương thức trên dữ liệu được deserialize khi gặp điều kiện tương ứng
    - Ví dụ ta có đoạn code như sau:
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/71aa5605-3257-43c8-ae1b-949040b21e3a/Untitled.webp)
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/30f58f6e-4fb8-4387-accd-5b981aeae167/Untitled.webp)
        
        ⇒ Phân tích ta thấy lỗ hổng tồn tại ở dòng 19 nơi mà người dùng có thể control được data, giả sử 2 dòng comment cuối cùng là payload, ta cùng phân tích như sau:
        
        - Đầu tiên, nó sẽ gọi class Database đồng thời hàm _**_destruct()** được gọi tự động. Bời vì thuộc tính handle được chèn thành đối tượng **TempFile** và gọi tới hàm **shutdown()** của lớp File
        - Trong hàm **shutdown()** lại gọi tới hàm **close()** được kế thừa ở lớp TempFile → chương trình sẽ xóa file mà attacker chỉ định thông qua thuộc tính filename → Xóa file **.htaccess**
- **Inject thuộc tính các đối tượng**
    - Các method có sẵn được định nghĩa bởi chính class của nó. Do đó, nếu attacker có thể thao túng xem class của các object nào được đưa vào trong serialized data, chúng có thể ảnh hưởng đến code nào được thực thi sau đó ngay cả trong quá trình deserialization
    - Các phương thức deserialization không kiểm tra chúng đang deserialize cái gì → có thể truyền vào trong các đối tượng của bất cứ serialize class mà có sẵn trên website và object sẽ được deserialize → Có thể cho phép attacker tạo instance của class bất kỳ
- **Gadget chains**
    - Các class chứa các deserialization magic methods có thể được sử dụng để khởi tạo nhiều cuộc tấn công phức tạp liên quan đến lời gọi chuỗi dài các method liên quan (Ví dụ như ví dụ xóa file tùy ý ở trên)→ Các method này được gọi là **Gadget**
    - Kỹ thuật POP lợi dụng các đoạn mã nguồn khác có sẵn trong chương trình (thường là các đối tượng có sẵn trong mã nguồn hay thư viện đi kèm) được gọi là **gadget** và kết hợp chúng lại với nhau thành một payload hoàn chỉnh **(gadget chains)** để khai thác.
    - Cách tạo một gadgets-chain đi theo một số bước cơ bản như sau:
        - Xác định lỗ hổng trong hệ thống → tìm các keyword như serialize() và unserialize() xem entrypoint nào sử dụng hàm này và xác định xem tại entrypoint đó có control được không
        - Tìm gadgets-chain trong các thư viện, có một công cụ đã tổng hợp một số gadgets-chain hợp lệ trên một số nền tảng, thư viện và hỗ trợ tạo ra PoC hoàn chỉnh, giúp giảm khá nhiều thời gian tự xây dựng ⇒ https://github.com/ambionics/phpggc/
        - Viết serialize() gadgets-chain đã hoàn chỉnh
- **Tự tạo exploit**
    - **Như đã nói ở trên thì cách để tạo ra một expoit riêng của mình gồm 3 bước:**
        - Xác định lỗ hổng trong hệ thống
        - Tìm gadgets-chain trong các thư viện
        - Viết serialize() gadgets-chain đã hoàn chỉnh ⇒ Hay còn gọi là viết payload
- **PHAR deserialization**
    - Phar là gì?
        - Phar file trong PHP tương tự như jar file trong java là một packet format cho phép ta gói nhiều các tập code, các thư viện, hình ảnh,… vào một tệp
        - Cấu trúc của một phar file gồm có:
            - **Stub:** là một file PHP hoặc ít nhất chứa đoạn code sau `<?php __HALT_COMPILER();`
            - **A manifest:** miêu tả khái quát nội dung sẽ có trong file
            - **Nội dung chính của file**
            - **[Signature]:** để kiểm tra tính toàn vẹn của file
    - Từ đó nên để khai thác lỗ hổng PHAR deserialization ta cần 3 điều kiện sau:
        - Tìm được POP chain trong source code cần khai thác
        - Tìm được phar file vào đối tượng cần khai thác
        - Tìm được entry point, đó là những chỗ mà các filesystem function gọi tới các phar file do người dùng kiểm soát. Một số filesystem function có thể trigger lỗ hổng này là:
            
            ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/d4501ab6-fd80-4670-945a-53719057366d/Untitled.png)
            
    - Ví dụ về một trường hợp khai thác lỗ hổng PHAR deserialization trên zend framework như sau:
        - Đầu tiên là tìm entrypoint mà filesystem function được gọi:
            
            ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/58a632e4-f863-4f35-ae6f-3bce4ff4d2b6/Untitled.png)
            
            ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/35076210-91aa-4789-815f-f916b6d83da6/Untitled.png)
            
            - Ta có thể thấy tại entry point đó, file_exists() được gọi và có tham số truyền vào là $url được gửi lên bởi người dùng
        - Tiếp đến là tìm POP chain → điều này chỉ có thể thực hiện bằng 2 cách 1 là review source 2 là sử dụng tool PHPGCC
        - Tạo một file PHAR với tên **“test.phar”** với POP chain được generate bên trên:
            
            ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/34eca784-6f10-4262-90ca-8aae8215b3fc/Untitled.png)
            
            → Đẩy file này vào chỗ cần khai thác và chạy nó là xong.
            
            ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e7b71266-98be-4fc8-8ccd-140d1349865e/1c322366-a38f-4eaa-9846-e3b528fad192/Untitled.png)
            

## Tham khảo

- https://portswigger.net/web-security/deserialization#what-is-serialization
- https://medium.com/@vnptsec/insecure-deserialization-8b6594cef727
- https://sec.vnpt.vn/2019/08/ky-thuat-khai-thac-lo-hong-phar-deserialization/
- https://www.sonarsource.com/blog/new-php-exploitation-technique/
