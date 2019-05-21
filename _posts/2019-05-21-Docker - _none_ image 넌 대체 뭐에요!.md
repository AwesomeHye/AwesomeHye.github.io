# Docker - &lt;none&gt; image 넌 대체 뭐에요!


도커에서 컨테이너 생성하는데 자꾸 에러가 나서  
Dockerfile을 고치고  
이미지 다시 생성하고  
컨테이너 생성해보고  
반복 삽질 중인 평화로운 나날이었다.  
그런데 어느 순간  
`sudo docker images`  
를 쳐보니..  

![lots of nones](/assets/20190521/lots_of_nones.png)

누가 생성했는지도 모르는 &lt;none&gt;:&lt;none&gt;  이미지들이 증식되어있었다. 

## 결론부터 말하자면
`sudo docker rmi $(sudo docker images -f "dangling=true" -q) --force`

필요없는 이미지이니 지워주면 된다.

## &lt;none&gt; image가 뭔데요?
**&lt;none&gt;:&lt;none&gt;  이미지는 어떤 코드에서든 더 이상 참조되지 않는 이미지이다.**  
docker build를 하거나 이미지를 pull했을 때 생겨난다.

현재 도커 이미지의 상황을 보면
![before build](/assets/20190521/before_build.png)

IMAGE ID가 7184b27319b4 이고 이름이 test:0.1 인 이미지가 있다.

이 이미지의 Dockerfile을 일부 수정하고,

`sudo docker build --tag test:0.1 ./`
이미지를 재생성 해주면

![after build](/assets/20190521/after_build.png)

다음과 같이 원래 test:0.1의 ID였던 7184b27319b4 가 &lt;none&gt;의 이름으로 변경된 것을 볼 수가 있다.  

**즉,  빌드 이전에는 ID가 7184b27319b4인 이미지를 우리가 test라는 이름으로 참조할 수 있었지만, test이미지가 재생성되면서 test라는 이름이 다른 이미지로 옮겨갔고, 7184b27319b4 이미지는 아무도 참조하지 않는 이름없는 &lt;none&gt; 이미지가 되어버린것이다.**  


이와 같은 &lt;none&gt;:&lt;none&gt; 이미지를  
**dangling image 라 하며,**  
> dangling: 매달려있는. 마치 누가 이미지를 절벽에서 밀었는데 가까스로 나뭇가지를 잡고 매달려있는 모습이 형상된다.  

슬프지만 쓸 때 없이 디스크 공간을 잡아먹기 때문에 수시로 prune해주는 것이 좋다.  
(prune: 일명 가지치기)

JAVA 같은 경우는 아무도 참조하지 않는 할당된 메모리인 경우 garbage collector가 자동으로 정리해주지만 docker는 가비지 콜렉터가 없기 때문에 dangling image들을 수동으로 지워주어야 한다.  

지우는 명령어는
`sudo docker rmi $(sudo docker images -f "dangling=true" -q) --force`
이다.
- *sudo* : root 계정이 아닐 경우에 붙여주는데, $( 다음에 오는 docker images 앞에도 반드시 붙여주어야 한다.
(나 같은 경우는 계속 에러나길래 알고보니까 뒤에 sudo 안 붙여줘서 였다.)  
- *rmi*: 이미지 삭제
- *-f* : filter. dangling=true로 dangling 이미지 목록만 보여준다.
- *-q*:  IMAGE ID 목록만 보여준다.


## 참조 링크
[what-are-docker-none-none-images]: http://www.projectatomic.io/blog/2015/07/what-are-docker-none-none-images/ {:target="_blank"}






