- **JWT là gì?**
  - JWT là một định dạng chuẩn, để truyền an toàn các thông tin giữa các bên dưới dạng đối tượng JSON
  - JWT gồm 3 phần: header, payload và signature
    - **Header - gồm 2 phần**
        - Loại token - mặc định là JWT
        - Thuật toán chính dùng để mã hóa - HS256 hoặc RS256
    - **Payload:**
        - Chứa dữ liệu mà ta muốn lưu lại trong JWT
    - **Signature:**
        - Vì header và payload được lưu trữ ở dạng plaintext  → sig được sử dụng để ngăn chặn data bị chỉnh sửa
        - Được mã hóa bởi header, payload và một chuỗi bí mật
        - Đầu tiên, chúng ta sẽ **Encode** *(chuyển đổi)* 2 cái **Header** và **Playload** ở trên theo kiểu **[Base64URL Encoder](https://kjur.github.io/jsjws/tool_b64uenc.html)**, và nối 2 chuỗi nhận được lại (cách nhau bởi dấu chấm **“.”**) rồi gán nó vào một biến là **data**.
            
            →Tiếp theo sẽ **Hash (băm)** cái **data** đó bằng **“alg”**
            
            → Sau khi băm xong ở trên thì thực hiện **Encode** tiếp một lần nữa cái dữ liệu băm đó dưới dạng **Base64URL Encode**, và chúng ta sẽ thu được chữ ký “**Signature”**
            
- **********Tính năng của JWT:**********
    - **JWT thường dùng cho xác thực:** Khi mà user login, mỗi request đều chứa chuỗi token JWT → cho phép người dùng truy cập đường dẫn, dịch vụ và tài nguyên ứng với token đó. SSG (Single Sign On) cũng là một chức năng có sử dụng JWT một cách rộng rãi vì JWT có kích thước đủ nhỏ để đính kèm trong request
    - **Trao đổi thông tin:** Nhờ vào signature → xác thực nguồn gốc thông tin và xác thực tính toàn vẹn: vì sig được tính toán dựa vào nội dung của header và payload
- **JWT hoạt động thế nào:**
    - **Trong việc xác thực**, khi user login thành công, Server trả về 1 chuỗi JWT về Browser, và Token JWT được lưu lại trong Browser người dùng(thường là localStorage hoặc HTTP Cookies) thay vì việc tạo 1 session trên Server và trả về cookie
    - Khi user muốn truy cập → Browser sẽ gửi token JWT này trong Header Authorization

    ![image](https://github.com/thanhlam-attt/Web_Vulnerabilities/assets/79523444/eff4bd30-bd91-4ec9-be07-a2f216d864af)


- ***Vậy thì, JWT có được lưu trên server không hay chỉ mỗi user’s browser không, nếu không → server validate JWT user’s browser như thế nào?***
    - Phần lớn JWT được sử dụng để xác thực và ủy quyền cho người dùng. Thông thường, khi người dùng đăng nhập thành công, server sẽ tạo ra một JWT và gửi nó lại cho trình duyệt của người dùng. Trình duyệt sẽ lưu trữ JWT trong cookie hoặc local storage. Mỗi khi người dùng gửi yêu cầu tới server, JWT sẽ được gửi lại cùng yêu cầu để server có thể xác thực người dùng. Vì vậy, JWT thường được lưu trữ trên trình duyệt của người dùng.
    - Tuy nhiên, trong một số trường hợp, server cũng có thể lưu trữ JWT để đảm bảo tính toàn vẹn của dữ liệu. Ví dụ, trong các ứng dụng có tính năng "Remember me", JWT có thể được lưu trữ trên server để đảm bảo rằng người dùng vẫn được xác thực ngay cả khi họ đóng trình duyệt.
    - Server validate bằng cách decrypt JWT nếu ra đoạn hash ⇒ Đúng user
- **JWT và Session Cookies trong việc Authentication:**
    - ***JWT, Cookies được lưu trên Browser người dùng còn Session được lưu cả trên user Browser và bên phía server***
    - ***Cơ chế xác thực đăng nhập bằng session và cookies:***
        
        ![image](https://github.com/thanhlam-attt/Web_Vulnerabilities/assets/79523444/d0c00b47-1c65-40c0-aedb-a0a05e5b3f8b)

        
        - Sau khi đăng nhập, server sẽ tạo ra session cho user và lưu vào đâu đó như file, memory, database,… Sau đó một sessionID được lưu vào trong cookies của trình duyệt, khi user truy cập vào website thì session ID đó sẽ được browser lấy ra và gửi kèm theo trong request. Nhờ vậy mà server biết user này đã đăng nhập hay chưa. Sau khi user log-out thì session sẽ bị xóa đi (hoặc có thể không - như facebook hay insta chả hạn)
    - ***Cơ chế xác thực đăng nhập bằng token:***
        - Khi user đăng nhập thì server sẽ tạo ra một đoạn token được mã hóa và gửi lại nó cho client. Khi nhận được token thì client sẽ lưu trữ vào bộ nhớ (thường là local storage). Sau đó mỗi khi client request lên server thì sẽ gửi kèm theo token. Từ token này server sẽ biết được user này là ai (bằng cơ chế giải mã)
    - ***Vậy nên dùng cái nào:***
        - JWT là phương pháp sẽ trở thành 1 tiêu chuẩn để thay thế cho Session và Cookies.
        - Vì token được lưu phía client trong khi session cookies được lưu cả bên phía server → là vấn đề lớn khi mà một số lượng lớn người dùng sử dụng hệ thống cùng 1 lúc
        - Nhưng cũng có nhược điểm là JWT có kích thước lớn hơn nhiều so với session ID vì JWT chứa nhiều thông tin người dùng hơn. Ngoài ra thông tin người dùng trong JWT có thể bị decode
- **Những rủi ro khi sử dụng JWT xác thực:**
    - **JWT không bảo vệ dữ liệu của chúng ta** mà chỉ xác thực tính toàn vẹn và xác thực nguồn gốc thông tin, vì: các bước xử lý Header, Payload và Signature dữ liệu chỉ được Encoded và Hash chứ không phải Encrypted → Có thể dễ dàng decoded và xem thông tin, đánh cắp thông tin nhạy cảm
        - **Sự khác biệt giữa Encoded và Encrypted:**
            - **Encode:** là quá trình chuyển đổi từ định dạng này sang định dạng khác dựa vào vào một phương pháp, một bảng mã được công bố công khai → mục đích là để tăng sử dụng dữ liệu trong hệ thống khác nhau
            - **Encrypt:** là quá trình chuyển đổi dữ liệu được sử dụng các thuật toán mã hóa, chúng chuyển đổi sang dạng chỉ có thể giải mã được khi có khóa phù hợp → mục đích để bảo mật thông tin
                
                → Encode không cần thuật toán còn Encrypt thì có, Encrypt được sử dụng để bảo mật thông tin còn Encode đơn giản chỉ là chuyển đổi sang định dạng khác
                
    - **Rủi ro về việc chia sẻ thông tin quá mức trong phần payload:** có một số trường hợp lưu trữ cả secret key trong payload
    - **Rủi ro về khóa** → Nếu secret key bị lộ hacker có thể giả mạo thông tin người dùng
    - **Rủi ro về thời gian sống:**
        - JWT có một thời gian sống được thiết lập (expiration time)
            - Nếu thời gian sống quá lớn → hacker có thể sử dụng jwt cho mục đích xấu
            - Nếu thời gian sống quá ít → người dùng phải đăng nhập lại liên tục khiến giảm trải nghiệm của người dùng
- **Tấn công JWT Authentication:**
    - **Tìm kiếm thông tin nhạy cảm trong JWT:**
        - Những thông tin nhạy cảm có thể được lưu trữ trong payload nhưng chúng chỉ được encode bằng mã base64 → rất dễ decode
    - **Có thể thay đổi phần alg thành none** → không cần áp dụng chữ ký số → tự do tùy chỉnh thông tin payload mà không cần tới khóa
    - **Khóa HS256** quá yếu có thể dẫn tới dễ dàng bị **brute force** bằng các tools như jwt-tools, hash-cat hoặc john the ripper
        - Chú ý là **chỉ brute force được HS256** chứ không brute force được RS256 với 2 lý do sau:
            - Brute force không thể được thực hiện trên khóa RS256 vì **độ dài của khóa RSA là quá lớn**
            - **Khóa riêng được giữ bí mật không thể bị rò rì** → không thể brute force trên khóa
    - Nếu **thay đổi phần alg từ mã hóa bất đối xứng** như RS256 **sang mã hóa đối xứng** như HS256
        - HS256 sử dụng secret key để ký và xác thực mỗi thông điệp còn RS256 sử dụng private key để ký thông điệp và sử dụng public key để xác thực chúng
        
        → khi thay đổi giá trị alg từ RS256 sang HS256 thì public key của mã hóa bất đối xứng sẽ là secret key → lộ khóa
        
    
        ![image](https://github.com/thanhlam-attt/Web_Vulnerabilities/assets/79523444/f675e026-829c-4c04-8e58-093dc98fba5a)

    
    - Ở đoạn code trên mô tả việc xác thực chữ ký JWT sử dụng public_key, khi chuyển từ RS sang HS thì public_key chuyển thành secret key và private key cũng phải chuyển thành secret key. Các bước tấn công như sau:
      1. Thay đổi kiểu thuật toán từ RS256 sang HS256
      2. Thực hiện thay đổi trong phần payload
      3. Ký token bằng khóa công khai
      4. Trả lại JWT cho ứng dụng 
    
    
    
    - ****JWT header parameter injections:****
        - **Trong JWT header chỉ có tham số header “alg” là bắt buộc nhưng ngoài alg còn chứa một vài tham số header khác như:**
            - **JWK (JSON Web Key):** là một cấu trúc dữ liệu JSON đại diện cho một khóa mật mã
            - **JKU (JSON Web Key set URL):** tham số jku trong header JWT được sử dụng để chỉ ra bộ khóa web JSON URL. Tham số này chỉ định nơi trích xuất khóa web JSON, chủ yếu là khóa công khai được sử dụng để xác thực chữ ký. ***Tấn công JKU thực hiện theo các bước sau:***
                - Kẻ tấn công thay đổi giá trị tham số jku thành khóa công khai được lưu trữ của chính nó
                - Nếu cơ chế lọc URL không được triển khai → máy chủ ứng dụng sẽ tìm nạp khóa từ URL được đề cập trong header của token
                - Vì kẻ tấn công đã ký token bằng khóa riêng tư mà nó sở hữu → máy chủ sẽ nhận ra JWT là một mã hợp lệ khi xác minh bằng khóa công khai
            - **KID (Key ID):** Thường các máy chủ ủy quyền sử dụng nhiều khóa bí mật để ký token. Phần tử KID sẽ hoạt động như một mã định danh chỉ định khóa nào sẽ được sử dụng trong xác minh chữ ký token
                - Vì kẻ tấn công có quyền kiểm soát tham số kid ⇒ có thể gửi payload chèn lệnh đến máy chủ và thực hiện các hành động độc hại như: command injection, path traversal hay SQL Injection ⇒ dẫn tới RCE
                
                ![image](https://github.com/thanhlam-attt/Web_Vulnerabilities/assets/79523444/430dc9ff-3f25-4179-9634-6e4af8e9a319)

                ![image](https://github.com/thanhlam-attt/Web_Vulnerabilities/assets/79523444/f931e494-cdf2-478f-8195-a496cbd71923)

                ![image](https://github.com/thanhlam-attt/Web_Vulnerabilities/assets/79523444/ad580fe5-b8c3-41c1-86ae-07dddf790c40)
                
                
                - Một số ứng dụng lưu trữ khóa trong cơ sở dữ liệu, nếu một khóa được tham chiếu trong tham số KID → có thể dễ bị chèn SQL → thực hiện được SQL Injection
- **Cách ngăn chặn các rủi ro này:**
    - **Sử dụng HTTPS** đảm bảo thông tin trong JWT được mã hóa trên đường truyền, tránh bị giả mạo hoặc lấy trộm JWT
    - **Sử dụng mã hóa:**  Bảo vệ nội dung JWT, tránh bị đánh cắp và giả mạo thông tin trong JWT
    - **Sử dụng các thuật toán mã hóa an toàn - có key an toàn:** Tránh bị brute force khóa và ngăn chặn hacker giải mã để đọc thông tin trong JWT
    - **Cập nhật thông tin trong JWT:** Đảm bảo rằng nội dung trong JWT không lộ những thông tin nhạy cảm hay bị thay đổi, ngăn chặn bị lừa đảo
    - **Thiết lập thời gian sống hợp lý:** ngăn chặn kẻ tấn công sử dụng JWT cũ để truy cập vào hệ thống.
- **Các công cụ decode, crack key JWT:**
    - https://jwt.io/
    - jwt_tools
        - git clone https://github.com/ticarpi/jwt_tool
        - python3 jwt_tool.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJyb2xlIjoiZ3Vlc3QifQ.4kBPNf7Y6BrtP-Y3A-vQXPY9jAh_d0E6L4IUjL65CvmEjgdTZyr2ag-TM-glH6EYKGgO3dBYbhblaPQsbeClcw -C -d /usr/share/wordlists/rockyou.txt
    - Hashcat:
        - `hashcat -m 16500 -a 0 jwt_token.txt /usr/share/wordlists/rockyou.txt`
    - John The Ripper:
        - `john jwt_token.txt -w=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256`

- Attack JWT: https://portswigger.net/web-security/jwt
- Bypass JWT: https://www.thehacker.recipes/web/inputs/jwt
- Tài liệu 1: https://blog.intigriti.com/2021/07/27/hacker-tools-jwt_tool/?cn-reloaded=1
- Tài liệu 2: https://security.stackexchange.com/questions/262106/crack-jwt-hs256-with-hashcat
- Tài liệu 3: https://tuhocnetworksecurity.business.blog/2021/01/28/kali-linux-can-ban-bai-8-hash-cracking-voi-hashcat-john-the-ripper-va-crackstation/
- Tài liệu 4: https://viblo.asia/p/tim-hieu-ve-json-web-token-jwt-7rVRqp73v4bP
- Tài liệu 5: https://viblo.asia/p/jwt-json-web-tokens-va-session-cookies-trong-viec-authentication-XL6lA9QDlek
- Tài liệu 6: https://redfoxsec.com/blog/jwt-authentication-bypass/
