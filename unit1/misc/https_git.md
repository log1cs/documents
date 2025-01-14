# In case you need Git via HTTP

### English
First of all, go back to your home directory.

Open `.gitconfig` with any text editor. I'll use `nano` in this guide.

```
nano .gitconfig
```

Then add this as the first section in your text file:

```
[http]
	cookiefile = %USERPROFILE%\\.gitcookies
	postBuffer = 1737418240
	version = HTTP/1.1
```

The file should look like this:
```
[http]
        cookiefile = %USERPROFILE%\\.gitcookies
        postBuffer = 1737418240
        version = HTTP/1.1
[user]
        name = <your username>
        email = <your email>
[color]
        ui = auto
```

Save it and you're good to go!

### Tiếng Việt

Đầu tiên thì bạn cần về lại thư mục home của mình bằng cách sử dụng câu lệnh

```
cd
```

Sau đó thì mở `.gitconfig` bằng trình text editor bạn muốn. Ở đây tôi sẽ dùng `nano`.

```
nano .gitconfig
```

Sau đó thì add dòng dưới vào section đầu tiên:

```
[http]
        cookiefile = %USERPROFILE%\\.gitcookies
        postBuffer = 1737418240
        version = HTTP/1.1
```

File sau khi đã edit sẽ có cấu trúc như dưới:
```
[http]
        cookiefile = %USERPROFILE%\\.gitcookies
        postBuffer = 1737418240
        version = HTTP/1.1
[user]
        name = <username của bạn>
        email = <email của bạn>
[color]
        ui = auto
```

Lưu lại file sau khi đã hoàn tất thay đổi.

