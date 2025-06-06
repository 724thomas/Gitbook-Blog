# Product Create Workflow

## Brainstorming

A: 제품 등록할때, 이미지도 같이 등록을 한다.

B: 제품 등록이 너무 오래걸릴거 같은데?

A: 그럼 제품 등록과 이미지 등록을 분리한다.  api를 분리하고 비동기로 처리?

B: 이미지 등록이 실패할 수도 있을거 같고, 이미지 등록이 늦어질수도 있지 않을까?

A: 그러면 이미지 등록을 먼저 한다면? 만약 이미지 등록에서 실패를 하더라도, 유저가 다시 업로드 할 수 있다. 이미지 등록이 늦어지는 문제는, 이미지 등록이되고 나서 제품이 등록되게 하는 방법이 있을거 같다.



## Workflow

1. 클라이언트에서 이미지를 한장씩 올린다. \
   서버에서는 s3와, productImage 테이블에 저장한다. \
   응답으로 저장된 image-id와, image-url이 반환됨.
2. 클라이언트쪽에서 받은 응답들을 저장하고 있는다.\
   이미지 순서를 바꾸고, 대표 썸네일 이미지를 선택한다.\
   제품 등록을 하면서, 이미지id : \[대표 썸네일 여부, 이미지 순서]를 보낸다.
3. 서버에서는 제품을 저장한다.\
   받은 이미지id를 순회하면서 제품id와 연결, 썸네일 여부, 이미지 순서를 업데이트 한다.

2-2. 만약, 제품이 등록이 취소되면, productImage 테이블에 저장된 레코드는 product와 연결이 되어있지 않다. \
그렇기 때문에, batch를 돌렸을때, product와 연결되어있지 않은 레코드들을 가져오고, 그 레코드들을 s3에서 삭제하고, product image 테이블에서도 삭제한다



## Feedback

1. batch를 통해 이미지 삭제를 하지 않아도 됨.
2. 자주 접근되지 않는 이미지는 자동으로 지워지게 할 수 있다.

그럼 어떻게 변하지?

1. 클라이언트에서 이미지를 한장씩 올린다. \
   서버에서는 s3에 이미지를 저장하고, image-id와, key가 반환됨.
2. 클라이언트쪽에서 받은 응답들을 저장하고 있는다.\
   이미지 순서를 바꾸고, 대표 썸네일 이미지를 선택한다.\
   제품 등록을 하면서, 이미지key : \[대표 썸네일 여부, 이미지 순서]를 보낸다.
3. 서버에서는 제품과 이미지 테이블에 데이터를 저장한다. (받은 이미지id를 순회하면서 제품id와 연결, 썸네일 여부, 이미지 순서를 업데이트를 할 필요가 없어진다.)

2-2. 만약 제품 등록이 취소되더라도, S3에 있는 이미지를 따로 지우지 않는다.

결국 s3이미지는 삭제되지 않고, 일정기간 사용이 안되면 cold storage로 옮겨지게 된다.
