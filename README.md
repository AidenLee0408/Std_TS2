# 타입스크립트 스터디노트 2
 노마드코더님의 [Typescript로 블록체인 만들기 강의](https://nomadcoders.co/typescript-for-beginners/lobby) 중 블록체인 코딩 과정을 스터디 노트로 작성한 것입니다.



## Block 구조 생성

```

class Block {
    static calculateBlockHash = (
        index:number, 
        previousHash:string, 
        timestamp:number, 
        data:string
        ): string => CryptoJS.SHA256(index + previousHash + timestamp + data).toString();

    static validateStructure = (aBlock: Block): boolean => 
      typeof aBlock.index === "number" && 
      typeof aBlock.hash === "string" && 
      typeof aBlock.previousHash === "string" &&
      typeof aBlock.timestamp === "number" &&
      typeof aBlock.data === "string";

    public index: number;
    public hash: string;
    public previousHash: string;
    public data: string;
    public timestamp: number;

    constructor(
      index: number,
      hash: string,
      previousHash: string,
      data: string,
      timestamp: number
    ) {
      this.index = index;
      this.hash = hash;
      this.previousHash = previousHash;
      this.data = data;
      this.timestamp = timestamp;
    }
  }
```

static 메소드 2개와 public 변수 5개와 Block의 구조를 입력을 해줍니다.

정적 타입인 calculateBlockHash 메소드는 내가 직접 블록을 생성하지 않아도 사용가능한 메소드를 만들 수 있게 해주고 
정적 타입인 validateStructure 메소드는 생성된 Block이 유효한지 판단 해줍니다.

public 변수뒤에 number, string 과 같은 타입을 명시해줌으로써 에러를 방지하고 constructor 안에는 public 변수들의 내용들을 그대로 가져다가 입력해줌으로써 Block 구조를 완성해준다.



## 첫 번째 Block 생성

```
const genesisBlock: Block = new Block(0, "2020202020202", "", "hello", 123456);
```

index 자리에는 0을 입력하고 hash, previous hash, data, timestamp 자리에는 임의값을 입력함으로써 첫번째 블록을 생성시켜준다. 



## 블록체인 생성

```
let blockchain: Block[] = [genesisBlock];
```

다음과 같이 값을 입력하여 Block만 블록체인에 추가할 수 있도록 하였다.

```
const getBlockchain = (): Block[] => blockchain;
const getLatestBlock = (): Block => blockchain[blockchain.length -1];
const getNewTimeStamp = (): number => Math.round(new Date().getTime() / 1000);
```

getBlockchain : 현재의 블록체인(길이)을 얻는다.

getLatestBlock : 블록체인에 가장 최근에 생성된 Block을 얻는다.

getNewTimeStamp : 새로운 현재의 타임스탬프를 얻는다.



## 새로운 Block 생성

```
const createNewBlock = (data: string): Block => {
    const previousBlock: Block = getLatestBlock();
    const newIndex: number = previousBlock.index + 1;
    const newTimestamp: number = getNewTimeStamp();
    const newHash: string = Block.calculateBlockHash(
        newIndex, 
        previousBlock.hash,
        newTimestamp,
        data
    );
    const newBlock : Block = new Block(
        newIndex,
        newHash,
        previousBlock.hash,
        data,
        newTimestamp
    );
    addBlock(newBlock);
    return newBlock;
  };
```

createNewBlock의 데이터 타입이 string 인지 확인을 한 뒤 Block 타입을 명시해주고 새 블록에 구조를 만들어준다.

이후 newHash를 생성하기위해 필요한 인자들을 받아오고 newBlock에도 필요한 인자를 받아 온 뒤 블록을 추가하고 newBlock을  리턴시키게 되면 새로운 블록을 생성하게 된다.



## Block에서 해쉬 얻기

```
  const getHashforBlock = (aBlock: Block) :string => Block.calculateBlockHash(
  aBlock.index, 
  aBlock.previousHash, 
  aBlock.timestamp, 
  aBlock.data);
```



## Block 구조 검증하기

```
const isBlockVaild = (
    candidateBlock: Block, 
    previousBlock: Block
    ) : boolean => {
    if(!Block.validateStructure(candidateBlock)){
      return false;
    } else if(previousBlock.index + 1 !== candidateBlock.index){
      return false;
    } else if(previousBlock.hash !== candidateBlock.previousHash){
      return false;
    } else if(getHashforBlock(candidateBlock) !== candidateBlock.hash) {
      return false;
    } else {
      return true;
    }
  };
```

블록이 유효하다면 구조를 검증한다. 

유효하지 않다면 false를 리턴,

이전 블록의  index + 1이  candidateBlock.index와 일치하지 않는다면 false를 리턴,

이전 블록의 hash가 andidateBlock.previousHash와 일치하지 않는다면 false를 리턴,

hash를 계산했는데 다른 hash를 가지고 있다면 false를 리턴,

이 모든 구조를 통과했다면 true를 리턴하여 push한다.



## 블록체인에 블록 추가하기

```
const addBlock = (candidateBlock: Block) : void => {
    if(isBlockVaild(candidateBlock, getLatestBlock())){
      blockchain.push(candidateBlock);
    }
  }
```

candidateBlock과 최근에 만들어진 getLatestBlock을 비교하여 참이라면 생성된 Block을 push하여 블록을 블록체인에 추가하게 된다.



## 마무리

```
  createNewBlock("second block");
  createNewBlock("third block");
  createNewBlock("fourth block");

  console.log(blockchain);
```

createNewBlock으로 블록들을 생성해주고 블록체인이 제대로 작동하는지 확인하기 위하여 console.log를 입력해 블록체인을 실행시켜준다.



## 실행 결과

```
[
  Block {
    index: 0,
    hash: '2020202020202',
    previousHash: '',
    data: 'Hello',
    timestamp: 123456
  },
  Block {
    index: 1,
    hash: '329f0b9e585b0b3f8a58d8f93ad53bd777c964d65e1457708a38800f7d595a63',
    previousHash: '2020202020202',
    data: 'second block',
    timestamp: 1644212037
  },
  Block {
    index: 2,
    hash: 'dc82615acaf706d809143af747d1b80c4a7aea2bd736990be302f0ce6c6bdfad',
    previousHash: '329f0b9e585b0b3f8a58d8f93ad53bd777c964d65e1457708a38800f7d595a63',   
    data: 'third block',
    timestamp: 1644212037
  },
  Block {
    index: 3,
    hash: 'b738f202a4a880c550e48be46a8699953407d5006aed6f41c03dc5d6c1962e6e',
    previousHash: 'dc82615acaf706d809143af747d1b80c4a7aea2bd736990be302f0ce6c6bdfad',   
    data: 'fourth block',
    timestamp: 1644212037
  }
]
```

터미널에 npm start를 입력하여 실행을 시켜보면 1~3까지 새로운 블록들이 정상적으로 체이닝 된 것을 알 수 있다. 3번의 previous hash를 보면 2번의 hash가 있는 것을 볼 수 있고 2번의 previous hash를 보면 1번의 hash가 있는것을 알 수 있다. 이런 hash를 이용하여 블록들이 연결된 것을 블록체인이라고 한다. (어디까지나 블록체인의 일부이다.)

이와같이 TypeScript로 블록체인을 만들게되면 이 코드가 어떻게 흘러가는지, 블럭의 속성과 타입, function의 리턴 타입 등이 보이기 때문에 가독성이 좋아져서 혼란스럽지 않게 파악이 가능하다는 장점이 큰 것 같다.



> 개인적인 소감

​	평소에 웹3와 메타버스에 대한 관심이 많았다. 이와 관련된 서비스를 만들고 싶었기 때문에 크립토, 블록체인 기술, NFT 등 웹 3와 메타버스의 핵심 기술에 대하여 많은 영상들을 보고 관련 소식들을 팔로잉 했었다. 그럼에도 불구하고 블록체인을 직접적으로 만들어 볼 엄두를 내지 못했었다. 프론트엔드에 대한 역량조차 아직 부족하다고 생각하여 나의 타스크를 좀 더 발전시키고 집중하고 싶었기 때문이다. 

​	이 강의는 그런 나의 갈증을 해소해 줄 수 있어서 더할 나위 없이 좋았다. TypeScript를 공부하고 있던 나는 간단한 블록체인을 TypeScript로 만들어 볼 수 있었기 때문에 정말 많은 것을 배울 수 있었다. 만약 자신이 TypeScript를 다룰 줄 아는데 크립토와 관련된 프로그래밍 공부를 하고 싶다면 블록체인에 간단히 배울 수 있으니 강의를 한 번 들어보는 걸 추천한다. (단, TypeScript를 전혀 다룰 줄 모른다면 다른 강의를 통해 먼저 학습을 한 뒤 듣기를 권하고 싶다.)



![TS수료증](https://user-images.githubusercontent.com/89402202/153022667-0f75384d-9552-45b8-a930-db4eef5f9df5.png)







