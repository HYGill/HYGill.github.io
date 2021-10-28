## Spring Batch



###### [배치 기본요소]

- **Reader** : 데이터 저장소에서 특정 데이터를 읽음

- **Processor** : 데이터를 가공하는 비즈니스 로직 처리

- **Writer** : Processor에서 수정된 데이터를 다시 서장소에 기록

- **Job** : Batch 처리의 기본 단위로 Job 객체는 여러 개의 step 인스턴스를 갖을 수 있음.

- **Step** : 실질적인 배치 처리를 정의하고 제어하는 데 필요한 모든 정보가 들어있는 객체

- **Chuck** : Stept(ItemReader -> Processor -> ItemWriter) 수행 시 데이터 처리, 커밋 단위 설정

- **Tasklet** : ItemReader, ItemWriter 구조가 아닌 경우에 사용하는 구조로 하나의 메서드에서 처리



------

###### [개발에서 마주친 문제]

1. **Step들이 데이터를 공유해야했음**

   Spring batch에는 **JobExecutionContext**, **StepExecutionContext** 이 두개의 ExecutionContext가 존재하는데 데이터를 참조가 Job level, Step level에서 사용가능하게 된다.

   나는 Step들이 같은 데이터를 참조했어야하기때문에 **JobExecutionContext**를 사용하는 것으로 첫번째 가닥을 잡았다.

   개발 도중 '나는 그냥 max 값 하나 int로 받아오는건데 이렇게까지 복잡하게 해야하나..?' 생각이 들어 그냥 싱글톤 component 만들기로 변경~

   ```
   @Component
   public class DataSaveBean<T> {
   	private Map<String, T> dataMap;
   	
   	public DataSaveBean() {
   		// ConcurrentHashMap : Multi-Thread 환경에서 사용
   		//  여러 쓰레드가 동시에 읽을 수 있지만, 쓰기 작업에는 특정 세그먼트 or 버킷에 대한 Lock을 사용한다
   		this.dataMap = new ConcurrentHashMap<>();
   	}
   	
   	public void putData(String key, T data) {
   		if(dataMap == null) {
   			return;
   		}
   		dataMap.put(key, data);
   	}
   	
   	public T getData(String key) {
   		if(dataMap == null){
   			return null;
   		}
   		return dataMap.get(key);
   	}
   }
   ```

   이렇게 class 만들어놓고

   ```
   @Bean
   public ItemWriter<Integer> writer() {
   	return new ItemWriter<Integer>(){
   		@Override
   		public void write(List<? extends Integer> items) throws Exception {
   		// writer가 list밖에 안되어서 이렇게 했는데 리스트로 받으려면 for문으로 하기!
   		dataMap.putData("data", items.get(0));
   		}
   	}
   }
   ```

   writer에서 put 하면 멤버변수로 상단에 선언해놓은 곳에 저장되기 때문에 밑에서 사용가능~

​		배치는 순서가 정해져있고 배치가 끝나면 다 리셋되기 때문에 가능한 방법이 아닌가 싶다. 그래서 든 생각은 그러면 JobExecutionContext 이건 언제 쓰는 걸까?



2. **write할때 Chunk 단위가 1000이었는데 10000건이 들어오면 max값을 10번 받아야하는거 아닌지?**

   우선 chunk의 유효성이 어디서부터 시작되는지 몰라서 나온 의문

   ![batch](https://user-images.githubusercontent.com/47243329/139216748-5d18d88f-0acb-4ce1-b53f-9a9f40939635.png)[출처 : https://t1.daumcdn.net/cfile/tistory/999A91385BCE63BC07]

   그것은 바로바로 Read에서부터~ 눈물줄줄..쨋던 그렇다..max값이 다음 chunk단위가 시작될때 초기화된다!
   
   그래서 process에서 item에 max값을 set해줄 때 공유하려고 만든 dataMap에도 같이 set해줘서 다음번 chunk가 시작될때에도 이어서 값을 받을 수 있도록 처리하였다.



3. **Max값 받아올때 어차피 1 row니까 chunk와 pageSize를 1로 하였더니 무한콜링**

   pageSize는 데이터 읽어올 때고 chunk는 write할 때 사용하는 단위라고 생각해서 어차피 둘다 1이니까 1했는데 무한루프에 빠진듯한 느낌~

   값을 이것저것 바꿔보니 chunk 1은 맞고 pageSize 1이 잘못되었었다.

   우선 값이 1개니까 page가 필요없긴해서 빼긴했는데 왜인지 궁금해서 연구해봄

   

   - **page 사용이유** : 우선 page와 chunk의 수는 맞추는 것이 일반적. transaction 처리를 chunk단위로 하기 때문에 실패하면 pageSize 대로 다시 가져와서 처리하게 된다.
   - 하다보니 **cursorItemReader** 형식이 있어서 바꿔봤더니 똑같이 작동되어서 바꿨다. 이거는 connection을 한번만 하고 data를 스트리밍형식으로 iterator로 가져온다. 대용량 데이터를 가져올때 용이하다고 했음. 근데 한번 가져오는거에 꽂혀서 나도 이걸로 바꿨다. Max는 한번가져오니까 ㅎㅎ

   - 문제가 무엇일까 MybatisPagingItemReader.class에 디버깅을 걸어보니 현재 page를 가져오는 getPage()가 계속 진행된다. 

     pageSize를 2로하면 1개만 넣어지고 밑에 아무것도 없으니까 마지막이라는 것을 인식하는데 1로하고 쿼리를 던지면 계속 값은 있고 하니까 계속 요청하는 것이 아닐지? 

