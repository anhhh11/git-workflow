# Cấu trúc gọi lệnh cơ bản Linux:

```
	command_name --param1 val1 --param2 val2 valOfNoNameParam1 valOfNoNameParm2
	# hoặc cách viết tắt
	command_name -p1 val1 -p2 val2 valOfNoNameParam1 valOfNoNameParam2
	# Quy ra code:
	# command_name(param1=val1,param2=val2,valOfNoNameParam1,valOfNoNameParm2)
	# Thường không quy định thứ tự với param có tên
	# Chi tiết về tham số(số lượng, thứ tự, ý nghĩa) thường dùng -h hoặc --help để xem
	# VD: ls --help; git --help
```


# Cấu trúc gọi lệnh git:

Giống cấu trúc gọi lệnh cơ bản linux nhưng thêm git ở đầu
```
	git command_name --param1 val1 --param2 val2 valOfNoNameParam1 valOfNoNameParm2
	# hoặc:
	git command_name -p1 val1 -p2 val2 valOfNoNameParam1 valOfNoNameParam2
```

Command_name của git đảm nhiệm 1 chức cụ thể duy nhất. Ví dụ:
- branch: thêm/sửa/xóa nhánh.
- commit
- log: xem, tìm kiếm lịch sử commit.
....


# Flow làm việc với git:

0. Tạo user git để share giữa các user 
* Nếu mỗi user với các tài khoản khác nhau push lên chung một repo -> sẽ gây ra lỗi, 
  do mặc định linux file nào được user nào push lên thì owner:group sẽ là của user đó thay vì là root:project1 như khởi tạo ban đầu 
  -> lỗi trong quá trình push, pull tạo branch,... -> dùng 1 user git share giữa các user *
- Tạo cặp khóa public, private

```
	# Trên máy local
	ssh-keygen -t rsa (từ cygwin hoặc git mingw), tạo cặp khóa
	# Gửi public key lên server
```

```
	su
	adduser git
	cd /projects/project1
	chown -R git:git ./
	mkdir .ssh
	# append toàn bộ ssh public key của user1, user2, user3 vào ~/.ssh/authorized_keys của user `git`
	mkdir -p /home/git/.ssh
	# append public key của các user
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

1. tạo remote repo
cd /projects/project1
git init --bare # tạo repo rỗng, không commit tạo branch được, chỉ push được.

2. khởi tạo các branch
```
# trên một client của userX bất kì
git clone remoteServer:/projects/project1
cd project1
git checkout -b dev # branch dev chứa bản mới nhất
# Khởi tạo template cho dự án như: copy-paste template CakePHP.....
# Tạo tệp .gitignore loại bỏ không track hay commit nhưng tệp của VD CakePHP:
# /lib
# /app/tmp
# /app/config/database.php
git add . # Thêm vào index
git commit -am "Initilize project" # Commit 
git checkout -b beta # tạo branch beta chứa bản dev ổn định
git checkout -b production # tạo branch production chứa bản beta đã accept
git push origin dev -u 
# Push lên server nguồn origin(nguồn mặc định khi clone về)
# Thêm -u để những lần push sau, chỉ cần `git push` sẽ tương đương với `git push origin dev -u`
# origin ở đây là nguồn remote, xem các nguồn ở `git remote -v`
# để đổi nguồn: `git remote set-url origin remoteServerOther:/projects/project1`
git push origin beta -u
git push origin production -u
```

3. Mỗi user1, user2, user3 sẽ clone về
```
git clone remoteServer:/projects/project1
git fetch --all # lấy tất cả các nhánh về
git checkout dev # sang nhánh dev
```

4. Mỗi user1, user2, user3 sẽ clone về được phân các chức năng, mỗi chức năng sẽ được dev ở các branch khác nhau.VD:
- Chức năng 1, user1 nhánh feature_f1
- Chức năng 2, user2 nhánh feature_f2
- Chức năng 2, user3 nhánh feature_f3
Các nhánh này được tạo thành từ nhánh dev ban đầu. Ví dụ với user1:
```
git checkout dev # về nhánh dev
git checkout -b feature_f1
# thao tác trên nhánh này, code...code..
git add . # thêm tất cả file mới vào inđex
git commit -am "...thêm controller..." # từ index xác nhận commit
git pull origin dev # pull về dev mới nhất
# làm phẳng history
git rebase dev # 
# * LƯU Ý: 
#  - KHÔNG ĐƯỢC từ `dev` thực hiện `git rebase feature_fX` SẼ BỊ CONFLICT (tham khảo ở đây: https://www.atlassian.com/git/tutorials/merging-vs-rebasing/conceptual-overview)
#  - Chỉ rebase trên branch mà không có ai khác đang sử dụng, ví dụ:
#	  git checkout someBranch
#     git rebase dev  
#   thì someBranch phải chưa được public, hoặc hiện tại không ai đang thao tác trên branch đó - hay checkout từ branch đó sang 1 branch khác
# *
# `xử lý conflict nếu có` && `git add` && `git rebase --continue`
git checkout dev
git merge feature_f2 # merge feature_f2 vào dev
git push origin dev # đẩy dev lên remote
```

5. Sẽ có userX nào đó đảm nhiệm review code và merge từ feature -> beta
```
git pull origin dev
git checkout dev
git merge beta
```

6. merge từ beta -> production
```
git pull origin beta
git checkout production
git merge beta
git tag -am "...Some message..." v1.01 # đánh tag phiên bản
```

7. Với các lỗi phát sinh ở feature tương ứng:
- userX code chúc năng đó sẽ checkout sang branch - ví dụ lỗi ở feature_f1 - feature_f1, từ branch này(hoặc branch phù hợp(VD: dev)
nếu lỗi này liên quan đến nhánh khác VD feature_f2..) sang branch `fix_bug_XYZ_feature_f1`
```
	git checkout feature_f1 # hoặc `git checkout dev`
	git checkout -b fix_bug_XYZ_feature_f1
	# fix bug, add, commit
	git checkout dev
	git merge fix_bug_XYZ_feature_f1
	git push origin dev
```

8. Nếu có conflict khi merge code, các userX có code trùng nhau sẽ bàn lại ngồi sửa. Rồi commit lên dev....

# Các lệnh thường dùng

1. Commit các tệp đã modify với kèm với message ngắn(1 dòng)
```
git commit -am "<message>" 
```

2. Commit các tệp mới và các tệp đã modify với kèm với message ngắn(1 dòng)
```
git add <tep1> <tep2> # hoặc: `git add .` để thêm tất
git commit -am "<message>" 
```

3. Lấy tất cả nhánh từ remote
```
git fetch --all
```

4. Tạo nhánh mới
```
git branch -b feature_fX
```

5. Checkout sang nhánh cũ(hoặc nhánh vừa mới fetch về)
```
git checkout branch_name
```

6. Merge 2 nhánh
```
git checkout feature_fX
git merge dev
```

7. Xem log các commit
```
git log --all
```

8. Xem danh sách các nhanh trên máy
```
git branch -v
``

9. Xem danh sách tất cả nhánh(trên máy+remote)
```
git branch --all
```

10. Làm phẳng history (thay vì phân nhánh khi merge thì rebase cho history dạng thẳng(linear))
```
git checkout feature_fX
git rebase dev
git checkout dev
git merge feature_fX
```
