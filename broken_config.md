0. Chuẩn bị
- Tạo 3 user: user1, user2, user3, đổi password, tạo home, trao quyền.
- Tạo nhóm mới cho project: project1
```
	su
	useradd user1; # tạo user
	passwd user1; # đổi mk
	usermod --move-home --home /home/user1 user1 # tạo home folder
	cd /home/user1 
	chown user1:user1 ./ # gán quyền sở hữu home cho user1
	chmod u=rwx,g=rwx,o= ./ # đổi mod, không cho phép ai ngoài user1, và group user1 được theo tác(bao gồm read,write,execute) trên home
	# .... tương tự cho user2,user3, có thể dùng lệnh adduser để tự động hóa việc tạo home, cấp quyền :)
	addgroup project1
	usermod --append user1 --groups project1 # *user được thêm mới cần logout, login lại thì quyền group mới có hiệu lực*
	# .... tương tự cho user2,user3
	grep ^project1 /etc/group # danh sách user trong group
	mkdir -p /projects/project1 # tạo thư mục chứa project1
	cd /projects/project1
	chown root:project1 ./ # đổi quyền sở hữu cho root và group project1
	chmod g=rwx,o= ./
```
- Tạo cặp khóa public, private
```
	# Trên máy local
	ssh-keygen -t rsa (từ cygwin hoặc git mingw), tạo cặp khóa
	# Gửi khóa lên server qua lệnh
	ssh-copy-id user1@123.123.123.123 
	# Hoặc login vào tài khoản user1
	# copy paste nội dung ~/.ssh/id_rsa.pub trên máy local vào cuối tệp ~/.ssh/authorized_keys trên home của user1
	# config file ~/.ssh/config trên local với nội dung như sau
		Host remoteServer
		Hostname 123.123.123.123
		User user1
		IdentityFile ~/.ssh/id_rsa
	# Với Hostname chứa địa chỉ IP hay URL của server
	# IdentityFile thư mục chứa public, private key
	# Host tên viết tắt thay cho dùng trực tiếp Hostname (sẽ đề cập ở sau)
	ssh remoteServer # login vào được server với account là user1
	whoami # user1 -> OK
```

0.5. Tạo user git để share giữa các user 
* Nếu mỗi user với các tài khoản khác nhau push lên chung một repo -> sẽ gây ra lỗi, 
  do mặc định linux file nào được user nào push lên thì owner:group sẽ là của user đó thay vì là root:project1 như khởi tạo ban đầu 
  -> lỗi trong quá trình push, pull tạo branch,... *
```
	su
	adduser git
	cd /projects/project1
	chown -R git:project1 ./
	mkdir .ssh
	# append toàn bộ ssh public key của user1, user2, user3 vào ~/.ssh/authorized_keys của user `git`
	mkdir -p /home/git/.ssh
	touch /home/git/.ssh/authorized_keys # tạo tệp rỗng
	cat /home/user1/.ssh/authorized_keys >> /home/git/.ssh/authorized_keys # tương tự user2,user3
	chmod 700 ~/.ssh
	chmod 600 ~/.ssh/authorized_keys
```
Trên máy client sửa User name từ user1 -> git
```
	Host remoteServer
	Hostname 123.123.123.123
	User git
	IdentityFile ~/.ssh/id_rsa
```