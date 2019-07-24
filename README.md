## What is NFS?

**NFS : Network File System**

> 클라이언트 컴퓨터의 사용자가 네트워크 상의 파일을 직접 연결된 스토리지에 접근하는 방식과 비슷한 방식으로 접근하도록 도와 준다.<br>

> 어떤 한 시스템(클라이언트)에서 다른 시스템(서버)의 자원을 자신의 자원처럼 사용이 가능하도록 하는 것을 말한다.<br>

> NFS는 여러 명이 같이 사용되는 대용량 프로그램이나 데이터들을 하나의 호스트에 넣어 두고 각 클라이언트들은 이 호스트의 데이터가 들어있는 디렉토리를 마운트하여 사용한다.<br>

#### <장점>

- 디스크 공간 면에서도 많은 절약을 가져올 수 있으며 각 각의 클라이언트마다 동일한 프로그램을 설치할 필요성도 없어지므로 관리면에서도 매우 편리해 진다.<br>


*****

#### 방화벽 문제 때문에 test 환경으로 하려면, 다른 컴퓨터가 아닌 같은 컴퓨터의 가상 환경에서 해야 함. 

#### 예를 들어, 같은 NAT에서 다른 컴퓨터로 한다고 해도, 각 컴퓨터의 운영체제의 방화벽 등의 문제 때문에 연결이 되지 않음(clnt_create rpc port mapper failure - unable to receive errno 113 (no route to host) --> 이런 에러 발생)

#### AWS server를 활용하면, 방화벽 문제만 해결하면 잘 Mount 됨.



*****

## BUILD a SERVER 

### Step 1
~~~
apt-get update
~~~

### Step 2
~~~
apt install nfs-kernel-server
~~~

### Step 3
~~~
mkdir -p /mnt/sharedfolder
~~~
>  공유할 폴더 만들기(관리자 권한이어야 만들 수 있음)

### Step 4
~~~
chown nobody:nogroup /mnt/sharedfolder
~~~

### Step 5
~~~
chmod 777 /mnt/sharedfolder
~~~

### Step 6
~~~
vi /etc/exports
~~~

> 파일 안에 들어가서 공유할 폴더, 접근 허용할 IP주소(client), 접근하는 클라이언트에게 줄 권한 적고 나오기
> ex) **/mnt/sharedfolder *(ro)**
>> - 모든 곳에서 접근하는 것에게 ro 권한 허용
>> - ro(read-only), rw(read-write) 등의 옵션 사용 가능

<img width="500" alt="스크린샷 2019-07-24 오후 4 34 16" src="https://user-images.githubusercontent.com/37536415/61774107-23470c00-ae31-11e9-95c2-9e2e9d829f42.png">

### Step 7
~~~
exportfs -a
~~~
<img width="500" alt="스크린샷 2019-07-24 오후 4 36 14" src="https://user-images.githubusercontent.com/37536415/61774130-2c37dd80-ae31-11e9-91ac-bcd05386d8a8.png">

**/etc/exports 에 바꾼 내용 update **

### Step 8
~~~
systemctl restart nfs-kernel-server
~~~

### Step 9
~~~
ufw allow from 192.168.0.48(client의 ip 주소) to any port nfs
~~~
> client로 사용할 os에서 ifconfig 하여 확인된 주소

### Step 10
~~~
ufw enable
~~~
> 방화벽 활성 상태로 만들기 ufw enable(enable을 하지 않으면, status를 확인하면 상태가 비활성임) - 아래 사진 참고

<img width="500" alt="스크린샷 2019-07-24 오후 4 39 12" src="https://user-images.githubusercontent.com/37536415/61774330-9d779080-ae31-11e9-8144-61f88173a0cc.png">

### Step 11
~~~
ufw status
~~~
<img width="500" alt="스크린샷 2019-07-24 오후 4 39 19" src="https://user-images.githubusercontent.com/37536415/61774331-9d779080-ae31-11e9-9bcf-63de117636ba.png">

**허용한 IP 주소에 대해서 활성화 상태인 것을 확인할 수 있음**

### Step 12
~~~
showmount -e
~~~
> server 측에서 mount가 잘 되고 있는 지 확인하기 위해서 사용
>> 공유하고자 하는 폴더 (/etc/exports) 에서 지정해주었던 /mnt/sharedfolder에 대해 *(global)한 ip 주소들이 접근 가능하다는 것을 확인할 수 있음

<img width="500" alt="스크린샷 2019-07-24 오후 4 41 26" src="https://user-images.githubusercontent.com/37536415/61774488-ec252a80-ae31-11e9-8e1d-aa1d97009bd4.png">

### Step 13
~~~
ifconfig
~~~
> 서버 측의 ip 주소를 확인하기 위해(클라이언트에서 접속할 때, 필요함 기억해두기)

### Step 14
~~~
systemctl start nfs-server
~~~

### Step 15
~~~
systemctl enable nfs-server
~~~

### Step 16
~~~
systemctl status nfs-server
~~~
<img width="500" alt="스크린샷 2019-07-24 오후 4 46 31" src="https://user-images.githubusercontent.com/37536415/61774799-9bfa9800-ae32-11e9-83d6-34ff9f0f5645.png">


#### 서버는 준비 완료

*****

## BUILD a CLIENT

### Step 1
~~~
mkdir /nfs/
~~~
> 서버의 폴더를 공유할 수 있는 폴더 만들기

### Step 2
~~~
mount -t nfs 192.168.0.49:/mnt/sharedfolder /nfs/
~~~
- 192.168.0.49 : 아까 확인했던 서버의 ip 주소
- /mnt/sharedfolder : 서버의 공유할 폴더(서버에서 공유할 폴더 생성했었음. 그 파일 경로)
- /nfs/ : 클라이언트의 폴더(step 1에서 )

### Step 3
~~~
df -h
~~~
> 연결 확인
<img width="500" alt="스크린샷 2019-07-24 오후 4 53 24" src="https://user-images.githubusercontent.com/37536415/61775338-b97c3180-ae33-11e9-8cbc-c968766da94d.png">

### Step 4
~~~
exit
~~~
> 현재 터미널 종료 후 다시 터미널 켜기(종료하지 않으면, 공유된 폴더의 목록을 확인하지 못할 수 있음)

### Step 5
~~~
cd /nfs/
~~~
> 새로운 터미널 창에서 mount된 폴더로 이동

### Step 6
~~~
ls
~~~
- 만약, 서버에 아무 파일이 없다면 아무 것도 뜨지 않을 것
- 파일이 있다면, 그 파일을 확인할 수 있어야 함
- 현 상태에서 서버 쪽에서 파일을 만들면, 클라이언트 쪽에서도 ls 명령어로 바로 파일을 확인할 수 있음

<br>
<br>
<br>

##### NFS의 기본 port : 2049
##### 위의 방법으로 되지 않을 때, 방화벽 문제가 있을 수 있음(서버 쪽에서 ufw 작업을 하기는 하지만... 방화벽 문제가 있으면, 서버의 firewall도 고쳐보기)
~~~
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd    --permanent    --add-port=2049/tcp
firewall-cmd    --permanent    --add-port=2049/udp
firewall-cmd --reload
~~~
